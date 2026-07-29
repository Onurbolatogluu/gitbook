---
icon: corn
---

# Add a VMkernel Port Group to a Standard Switch

Önceki adımda yeni switch'imize sanal makineler için bir port group tanımlamıştık. Ancak bir switch'in taşıyabileceği tek trafik türü VM trafiği değildir: vMotion, Fault Tolerance, provisioning ya da yedek bir management bağlantısı gibi **host'un kendi servisleri** de bu switch üzerinden akabilir. Bunun için gereken bileşen, **VMkernel portudur**.

Bu makalede yeni bir VMkernel portunun nasıl oluşturulacağını adım adım ele alacak; TCP/IP stack ve servis seçimlerinin ne anlama geldiğini açıklayacak, "her servise ayrı VMkernel" prensibinin gerekçesini ortaya koyacak ve pratik bir dayanıklılık senaryosu olarak **ikinci bir management portu** ile host erişiminin nasıl güvence altına alınacağını göstereceğiz.

### Ne Zaman VMkernel Portu Gerekir?

Switch'i yalnızca sanal makine trafiği için kullanacaksanız VMkernel portuna ihtiyacınız yoktur; port group yeterlidir. VMkernel portu, şu ihtiyaçlardan biri doğduğunda devreye girer:

* Host'a **vMotion** yeteneği kazandırmak
* **Fault Tolerance** logging trafiğini taşımak
* **Provisioning** (cold migration, cloning, snapshot) trafiğini ayırmak
* iSCSI/NFS gibi **IP tabanlı storage** bağlantısı kurmak
* **Replication** trafiğini yönlendirmek
* Mevcut management bağlantısına **yedek bir yönetim yolu** eklemek

Varsayılan kurulumda host'ta tek bir VMkernel portu bulunur: `vmk0`, Management Network port group'una bağlıdır ve host'a bağlanırken kullandığınız yönetim IP'sini taşır. Yeni servisler için ek portlar oluşturmak size kalmıştır.

### Adım Adım: Yeni VMkernel Portu Oluşturma

**Networking → VMkernel NICs → Add VMkernel NIC** yolunu izleyin. Sihirbazdaki her alan bir tasarım kararıdır:

#### 1. Port Group

VMkernel portu da bir port group üzerinde yaşar. İki seçeneğiniz vardır: mevcut bir port group'a bağlanmak ya da **New port group** ile yenisini oluşturmak. Doğru pratik, her VMkernel portuna kendi amacını anlatan yeni bir port group tanımlamaktır: `PG-vMotion`, `PG-Management-2` gibi. Sanal makinelerin bağlandığı port group'larla VMkernel port group'larını karıştırmamak, hem topolojinin okunabilirliğini hem de politika yönetimini basitleştirir.

#### 2. Virtual Switch

Portun hangi switch üzerinde — dolayısıyla hangi uplink'ler ve hangi fiziksel yol üzerinden — çalışacağını belirler. `vSwitch0` veya yeni oluşturduğunuz switch seçilebilir.

#### 3. VLAN ID

VMkernel trafiğini kendi VLAN'ında izole etmek için kullanılır. Lab ortamında 0 bırakılabilir; production'da ise özellikle vMotion için ayrı bir VLAN **güvenlik gereğidir** — vMotion trafiği, taşınan sanal makinenin bellek içeriğini şifrelenmemiş olarak taşıyabilir ve asla genel VM trafiğiyle aynı broadcast domain'de akmamalıdır.

#### 4. MTU

Varsayılan 1500 değerini koruyun; yalnızca iSCSI/vSAN gibi Jumbo Frames senaryolarında ve uçtan uca tutarlılığı garanti ederek 9000'e çıkarın.

#### 5. IP Version ve IP Settings

Ortamınızda IPv6 devre dışıysa yalnızca IPv4 seçeneğini görürsünüz; etkinse ikisi arasında (veya ikisini birlikte) seçim yapabilirsiniz.

IP ataması tarafında kural nettir: **her zaman static IP kullanın.** DHCP teknik olarak mümkündür ama yönetim düzleminde iki ciddi soruna yol açar: host'un hangi IP'ye sahip olduğunu bilemezsiniz ve DHCP altyapısındaki bir kesinti, vMotion veya management gibi kritik servislerin erişimini de düşürür. Yönetim düzlemi, başka hiçbir sisteme bağımlı olmamalıdır.

IP seçerken host'un mevcut yönetim IP'siyle ve ortamdaki diğer host'ların adresleriyle çakışmayan, adresleme planınıza uygun bir değer atayın; subnet mask'i ağınıza göre girin.

#### 6. TCP/IP Stack

Çoğu zaman **Default TCP/IP stack** doğru seçimdir; ancak açılır listedeki diğer seçeneklerin varlık nedenini bilmek gerekir:

* **Default stack:** Management dahil genel VMkernel trafiği için ortak stack'tir; host'un default gateway'i buradadır.
* **vMotion stack:** vMotion trafiğine **kendi gateway'ini** tanımlama imkânı verir. vMotion trafiğinin farklı bir subnet'e yönlendirilmesi (routed vMotion) gereken büyük ortamlarda kullanılır. Bir VMkernel portu bu stack'e alındığında, o host'ta vMotion yalnızca bu stack üzerindeki portlardan yapılabilir.
* **Provisioning stack:** Aynı mantığın cold migration/cloning trafiği için karşılığıdır.

Tek subnet üzerinde çalışan tipik ortamlarda default stack yeterlidir; ayrı stack'ler, yönlendirilmiş (routed) trafik ihtiyacı doğduğunda anlam kazanır.

#### 7. Services

Sihirbazın en kritik adımı budur: bu VMkernel portunun **hangi servisleri taşıyacağını** işaretlersiniz — vMotion, Provisioning, Fault Tolerance logging, Replication, NFC, Management ve diğerleri.

Teknik olarak tek bir porta birden fazla servis atamak mümkündür; örneğin hem vMotion hem Management işaretlenebilir ve port her iki görevi de üstlenir. Ancak profesyonel yapılandırma prensibi tam tersini söyler:

> **Her kritik servis için ayrı bir VMkernel portu kullanın.**

Gerekçesi izolasyondur ve iki boyutu vardır:

* **Arıza izolasyonu:** Bir VMkernel portunda yaşanan sorun (yanlış yapılandırma, IP çakışması, VLAN hatası) yalnızca o servisi etkiler; diğer servisler çalışmaya devam eder. Beş servisi tek porta yığdığınızda, tek bir hata beş servisi birden düşürür.
* **Trafik izolasyonu:** vMotion, migration sırasında hattı doyurabilen yüksek hacimli bir trafiktir. Management ile aynı portu ve yolu paylaşıyorsa, yoğun bir migration anında host yönetim erişiminiz yavaşlayabilir hatta zaman aşımına düşebilir. Ayrı portlar (ideal olarak ayrı VLAN'lar), servislerin birbirinin bant genişliğini yemesini engeller.

Lab ortamında kavramı görmek için tek porta birden fazla servis atanabilir; production'da ise servis-port eşlemesi birebir olmalıdır.

**Create** dedikten sonra switch topolojisinde yeni tabloyu görürsünüz: aynı switch üzerinde artık hem sanal makineleri taşıyan VM port group'u hem de yeni VMkernel portu (örneğin `vmk1`) kendi IP'siyle yan yana durur.

### Pratik Senaryo: İkinci Management Portu ile Yönetim Erişimini Yedeklemek

Yeni VMkernel portuna Management servisini de eklediğinizde, host artık **iki farklı IP üzerinden** yönetilebilir hale gelir — ve bu iki IP, farklı VMkernel portları üzerinden **farklı switch'lere ve farklı fiziksel adaptörlere** çıkar:

* **Birinci yol:** Yönetim IP'si → `vmk0` → `vSwitch0` → `vmnic0`
* **İkinci yol:** Yeni IP → `vmk1` → yeni switch → `vmnic1`

Tarayıcıdan yeni IP'ye gittiğinizde (self-signed sertifika uyarısını geçtikten sonra) aynı host'a, aynı kullanıcı adı ve parolayla, ancak tamamen bağımsız bir fiziksel yoldan bağlanırsınız.

Bu tasarımın değeri arıza anında ortaya çıkar: `vmnic0` arızalanırsa, ona bağlı kablo koparsa ya da o adaptörün bağlı olduğu fiziksel switch düşerse, birinci yönetim IP'si erişilmez olur — ancak ikinci IP, tamamen ayrı bir donanım zinciri kullandığı için çalışmaya devam eder. Host'unuzun yönetimini hiçbir senaryoda kaybetmezsiniz.

#### Önemli nüans: Hangi yedeklilik, hangi arızaya karşı?

Bu noktada serinin önceki makaleleriyle bağlantılı bir ayrımı netleştirmek gerekir, çünkü iki farklı dayanıklılık mekanizması kolayca karıştırılır:

* **NIC teaming (aynı switch'te iki uplink):** Tek bir management VMkernel portu, iki uplink'li bir switch üzerindeyse, adaptör/kablo arızasında failover ile **aynı IP** üzerinden çalışmaya devam eder. Kullanıcı açısından hiçbir şey değişmez; bu, birinci tercih edilmesi gereken mekanizmadır.
* **İkinci management VMkernel portu (farklı switch, farklı IP):** Teaming'in koruyamadığı arıza alanlarına karşı ek katman sağlar — VMkernel portunun kendisindeki yapılandırma hatası, ilgili subnet/VLAN'daki bir ağ sorunu ya da birinci switch'in tüm uplink'lerini etkileyen bir kesinti gibi. Bedeli, ikinci bir IP'nin yönetilmesi ve DNS/envanter kayıtlarının buna göre tutulmasıdır.

Doğru production tasarımı ikisini birlikte kullanır: management portları teaming'li switch'lerde yaşar, kritik ortamlarda ise farklı arıza alanlarından geçen ikinci bir yönetim yolu bulundurulur. vCenter'ın host'u tek bir adresle tanıdığını da unutmayın — ikinci management IP'si öncelikle **acil erişim (out-of-band benzeri) yoludur**; iLO/iDRAC/IPMI konsol erişimiyle birlikte, "host'a hiçbir koşulda erişimsiz kalmama" hedefinin parçasıdır.

### Doğrulama

Yapılandırmayı tamamladıktan sonra CLI üzerinden kontrol edin:

```bash
# VMkernel portlarını, IP'lerini ve bağlı port group'ları listele
esxcli network ip interface list
esxcli network ip interface ipv4 get

# Yeni vmk üzerinden bağlantı testi (arayüz belirterek)
vmkping -I vmk1 <hedef-ip>

# vMotion etkinse, vMotion ağında MTU dahil test
vmkping -I vmk1 -s 1472 -d <karşı-host-vmotion-ip>
```

`vmkping`'in `-I` parametresi, testin gerçekten yeni oluşturduğunuz arayüz üzerinden yapılmasını garanti eder — hangi trafiğin hangi yoldan aktığını doğrulamanın en doğrudan yöntemidir. `-d` (don't fragment) ve boyut parametresiyle yapılan test ise MTU uyumsuzluklarını daha ilk günden yakalar.

### Sonuç

VMkernel portu, sanal ağ yapısının "host tarafını" tamamlayan bileşendir ve oluşturma sihirbazındaki her alan bir tasarım kararına karşılık gelir. Özetle:

* VMkernel portu yalnızca host servisleri (vMotion, FT, storage, management, replication) gerektiğinde oluşturulur; her porta amacını anlatan kendi port group'u tanımlanmalıdır.
* **Static IP** yönetim düzleminin pazarlıksız kuralıdır; TCP/IP stack seçimi ise ancak routed vMotion/provisioning ihtiyacında default'tan sapmalıdır.
* **Her kritik servise ayrı VMkernel portu** prensibi, hem arıza hem trafik izolasyonu sağlar; çoklu servis tek portta yalnızca lab senaryosudur.
* İkinci bir management portu, teaming'in kapsayamadığı arıza alanlarına karşı bağımsız bir yönetim yolu açar — doğru tasarımda teaming'in alternatifi değil, tamamlayıcısıdır.

Bu adımla birlikte yeni switch'imiz tam teşekküllü hale geldi: uplink'i, sanal makineler için port group'u ve host servisleri için VMkernel portu tanımlı durumda. Buradan sonrası, bu portlara atanan servislerin — başta vMotion olmak üzere — tek tek yapılandırılıp devreye alınmasıdır.
