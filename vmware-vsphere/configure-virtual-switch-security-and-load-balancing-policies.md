---
icon: gear-complex
---

# Configure Virtual Switch Security and Load-Balancing Policies

VMware ESXi ortamında **standard switch**, host üzerindeki VM'lerin fiziksel ağa çıkışını sağlayan temel yapı taşıdır. Bir standard switch'i doğru yapılandırmak sadece "VM internete çıksın" meselesinden ibaret değildir; uplink redundancy'den NIC teaming policy'sine, security ayarlarından VMkernel/management network ilişkisine kadar bir dizi karar, hem ağın dayanıklılığını hem de host'un yönetilebilirliğini doğrudan etkiler.

Bu yazıda, bir standard switch'i production'a hazır hale getirirken dikkat edilmesi gereken temel noktaları — uplink redundancy, switch özellikleri (MTU, port sayısı, CDP), NIC teaming/load-balancing algoritmaları, security policy ayarları ve VMkernel yapılandırması — sırasıyla ele alıyorum.

***

### 1. Uplink Redundancy: Neden Bu Kadar Kritik?

ESXi host client veya vCenter üzerinde sıkça karşılaşılan bir uyarı vardır: _"This virtual switch has no uplink redundancy."_ Bu uyarı, switch'in yalnızca **tek bir uplink adaptörüne (fiziksel NIC)** bağlı olduğunu gösterir.

Bunun pratikteki anlamı basit: switch üzerindeki tüm trafik tek bir fiziksel yol üzerinden geçiyor demektir. Bu NIC üzerinde bir arıza, kablo kopması, switch portu sorunu ya da fiziksel switch tarafında bir kesinti yaşanırsa, host'un ağ bağlantısı tamamen kopar — yönetim erişimi dahil.

```
Redundancy YOK (tek uplink):
   [ vSwitch0 ] ---- vmnic0 ---- [ Fiziksel Switch ]
        |
   (vmnic0 arızalanırsa host izole kalır)

Redundancy VAR (NIC teaming):
   [ vSwitch0 ] ---- vmnic0 ---- [ Fiziksel Switch A ]
        |     \
        |      ---- vmnic1 ---- [ Fiziksel Switch B ]
        |
   (Active/Standby ya da Active/Active — biri düşerse diğeri devam eder)
```

Günümüz sunucularının çoğu zaten 2'den fazla fiziksel NIC ile geldiği için bu genelde donanımsal bir kısıt değil, bir **yapılandırma tercihi** meselesidir. Ancak özellikle HAProxy ve CDN tabanlı yapılar gibi kesintiye duyarlı sistemlerde, tek uplink'li bir vSwitch, üst katmanda ne kadar sağlam bir redundancy stratejisi kurulmuş olursa olsun, bunun fiziksel katmanda delinmesi anlamına gelir. Bu yüzden en az iki fiziksel NIC'i aynı vSwitch'e uplink olarak bağlamak, production ortamlar için pratikte zorunlu kabul edilmelidir.

***

### 2. Switch Özellikleri: MTU, Port Sayısı ve CDP

Bir standard switch yapılandırılırken göz atılması gereken üç temel özellik şunlardır:

| Özellik                | Varsayılan / Tipik Değer                                | Pratik Not                                                                                                                                                                                                                               |
| ---------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MTU**                | 1500                                                    | Jumbo frame (9000 MTU) genelde yalnızca vMotion veya iSCSI/NFS storage trafiği izole bir switch üzerinden taşınırken tercih edilir. Karışık trafik taşıyan bir switch'te MTU'yu varsayılandan değiştirmemek genelde doğru bir tercihtir. |
| **Ports**              | Switch oluşturulurken belirlenir (genelde 128 ve üzeri) | Gerçek sınır, host'un desteklediği toplam port sayısıdır; port sayısını gereğinden yüksek tutmak ekstra maliyet getirmez ama gereğinden düşük tutmak ileride VM eklerken tıkanıklığa yol açabilir.                                       |
| **Discovery Protocol** | CDP (Cisco Discovery Protocol) veya LLDP                | CDP/LLDP, fiziksel switch tarafının sanal switch'i "görebilmesini" sağlar; troubleshooting sırasında hangi vSwitch'in hangi fiziksel porta bağlı olduğunu tespit etmek için çok işe yarar. Cisco olmayan ortamlarda LLDP tercih edilir.  |

***

### 3. NIC Teaming ve Load-Balancing Policy

ESXi standard switch'te dört farklı load-balancing algoritması bulunur ve bu seçim, hem performansı hem de fiziksel switch tarafındaki gereksinimleri doğrudan etkiler:

1. **Route based on originating virtual port ID** (varsayılan) — Her VM'in sanal port ID'sine göre bir uplink'e sabitlenir. Basit ve ek fiziksel switch yapılandırması gerektirmez.
2. **Route based on source MAC hash** — VM'in MAC adresine göre uplink seçimi yapılır; port ID yöntemine benzer davranış sergiler.
3. **Route based on IP hash** — Kaynak ve hedef IP'nin hash'ine göre dağıtım yapar; fiziksel switch tarafında **EtherChannel/LACP (static)** yapılandırması _zorunludur_, aksi halde paket kaybı veya MAC flapping yaşanır.
4. **Explicit failover order** — Teaming yok; tanımlı sırayla tek bir aktif uplink kullanılır, diğerleri yalnızca failover için beklemededir.

```
Load-Balancing Karar Ağacı:
                    ┌─────────────────────────┐
                    │ Fiziksel switch LACP/    │
                    │ EtherChannel destekliyor │
                    │ mu ve yapılandırıldı mı? │
                    └───────────┬─────────────┘
                     Evet ───────┴──────── Hayır
                      │                       │
              IP Hash kullanılabilir   Port ID / MAC Hash
              (yüksek performans)      (varsayılan, güvenli)
```

Burada asıl kritik nokta şu: seçilen teaming policy ile fiziksel switch tarafındaki yapılandırma **tutarlı olmak zorundadır**. Örneğin fiziksel switch'te LACP yapılandırılmamışken IP Hash policy'sini seçmek, HAProxy ile yük dengelenen backend'lerde session affinity bozulduğunda görülen türden — trafiğin öngörülemez şekilde dağılması, aralıklı paket kaybı, teşhisi zor kesintiler — sorunlara yol açar.

***

### 4. Security Policy: Promiscuous Mode, MAC Address Changes, Forged Transmits

ESXi standard switch güvenlik politikası üç ayardan oluşur:

* **Promiscuous Mode** (Reject/Accept): Reddedildiğinde (varsayılan), bir VM'in vNIC'i kendisine ait olmayan trafiği dinleyemez. Kabul edilirse VM, switch üzerindeki tüm trafiği görebilir — genelde yalnızca IDS/IPS veya paket yakalama (packet capture) senaryolarında açılır.
* **MAC Address Changes** (Reject/Accept): Reddedildiğinde, guest OS içinden vNIC'in MAC adresi değiştirilirse trafik switch tarafından düşürülür. Bu, MAC spoofing saldırılarına karşı bir önlemdir.
* **Forged Transmits** (Reject/Accept): Reddedildiğinde, VM'den çıkan paketlerin kaynak MAC adresi, vNIC'e atanmış gerçek MAC ile eşleşmiyorsa paket düşürülür.

Bu üç ayarın varsayılan değeri (**Reject**) genelde production ortamlar için doğru tercihtir; **Accept**'e çevrilmesi genelde yalnızca nested virtualization (örneğin bir ESXi içinde başka bir hypervisor çalıştırmak) veya ağ izleme araçları gibi özel senaryolarda gerekir.

***

### 5. VMkernel Adaptörü ve Management Network

Her ESXi host'unda **en az bir VMkernel adaptörü, management servisi etkin şekilde bulunmak zorundadır.** Bu VMkernel port silinirse, host vCenter'dan veya doğrudan host client üzerinden yönetilemez hale gelir; bu yüzden management VMkernel'i silmek klasik "kendi ayağına sıkma" senaryolarından biridir.

Bir VMkernel port üzerinde etkinleştirilebilecek başlıca servisler şunlardır:

* **Management** (her zaman en az bir VMkernel'de zorunlu)
* **vMotion** (canlı VM taşıma trafiği)
* **Provisioning** (cold migration, cloning, snapshot işlemleri)
* **Fault Tolerance (FT) logging**
* **vSphere Replication** ve **NFC (Network File Copy)**

Bu servisleri tek bir VMkernel port üzerinde toplamak, best practice açısından önerilmez. Production ortamlarda yaygın yaklaşım, **her servis için ayrı bir VMkernel port ve mümkünse ayrı bir ağ segmenti/VLAN** kullanmaktır:

```
Örnek VMkernel Ayrımı (Best Practice):
  vmk0  -> Management        (10.0.10.x)
  vmk1  -> vMotion           (10.0.20.x, ayrı VLAN, jumbo frame önerilir)
  vmk2  -> Fault Tolerance   (10.0.30.x)
  vmk3  -> vSAN / Storage    (10.0.40.x, ayrı VLAN)
```

Bu ayrım, hem güvenlik (management trafiğinin izole edilmesi) hem de performans (vMotion gibi yoğun trafiğin diğer servisleri etkilememesi) açısından önemlidir.

***

### Sonuç: Pratik Özet

Bir standard switch'i production'a hazırlarken kontrol edilmesi gereken maddeler şu şekilde özetlenebilir:

* **En az iki uplink** kullanarak redundancy sağlamak, tek nokta arızasını (single point of failure) ortadan kaldırmak.
* **NIC teaming policy** seçimini fiziksel switch tarafındaki LACP/EtherChannel yapılandırmasıyla tutarlı tutmak.
* **Security policy**'de Promiscuous Mode, MAC Address Changes ve Forged Transmits ayarlarını, açık bir gereksinim olmadıkça varsayılan (Reject) değerde bırakmak.
* **Management VMkernel'ini** her zaman koruyarak, diğer servisleri (vMotion, FT, replication) ayrı VMkernel port'larına ve mümkünse ayrı VLAN'lara dağıtmak.

Bu dört madde, görece basit görünen bir standard switch yapılandırmasını, gerçek anlamda dayanıklı ve güvenli bir ağ mimarisine dönüştüren temel kararlardır.
