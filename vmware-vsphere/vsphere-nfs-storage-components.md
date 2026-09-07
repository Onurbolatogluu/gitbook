---
icon: square-ring
---

# vSphere NFS storage components

Önceki bölümde iSCSI'yi ele aldık. Şimdi ikinci datastore tipine geçiyoruz: **NFS**.

Aradaki farkı tek cümleyle söylemek mümkün: **iSCSI size bir disk verir, NFS size bir klasör verir.**

iSCSI'de storage sistemi ham bir disk (LUN) sunar, ESXi onu VMFS ile biçimlendirir. NFS'te ise karşı taraf zaten hazır bir klasör paylaşır; ESXi bunu doğrudan datastore olarak bağlar, biçimlendirme yapmaz.

Bu makalede NFS'in hangi parçalardan oluştuğunu, host'un bu klasöre nasıl bağlandığını ve production ortamında nelere dikkat edilmesi gerektiğini anlatacağız.

### Zincirin Parçaları

NFS kurulumunda dört parça vardır:

```
[NFS sunucusu] → [Ağ] → [VMkernel portu] → [ESXi host]
```

Her birini sırayla açalım.

### 1. NFS Sunucusu

Klasörü paylaşan taraftır. İki şekilde olabilir:

**Hazır NAS cihazı:** Satın alıp ağa bağlarsınız. NetApp, Synology, QNAP gibi cihazlar bu iş için tasarlanmıştır. Yönetim arayüzünden klasör oluşturur, paylaşıma açarsınız.

**Sunucu işletim sistemi:** Elinizde diskleri olan bir sunucu varsa, üzerine NFS servisi kurup klasör paylaşabilirsiniz. Linux'ta bu iş `nfs-kernel-server` paketiyle yapılır; Windows Server'da da NFS rolü mevcuttur.

İkisi de aynı işi görür. Fark, hazır cihazın yönetiminin kolay olması ve genelde daha iyi performans ile dayanıklılık sunmasıdır. Lab ortamında bir Linux sunucusu fazlasıyla yeterlidir.

Sunucu tarafında yapmanız gereken şey basittir: bir klasör oluşturmak ve onu ESXi host'larına açmak. Linux'ta bu tanım `/etc/exports` dosyasında tutulur:

```
/mnt/vmstore    192.168.10.0/24(rw,sync,no_root_squash)
```

Bu satır şunu söyler: `/mnt/vmstore` klasörünü, 192.168.10.x ağındaki makinelere okuma-yazma yetkisiyle aç.

Buradaki `no_root_squash` önemlidir. ESXi paylaşıma root kullanıcısıyla bağlanır. Bu ayar olmazsa sunucu root erişimini kısıtlar ve host klasöre yazamaz. NFS kurulumlarında en sık karşılaşılan hata budur.

### 2. Ağ

NFS sunucusunun bir network kartı vardır ve normal Ethernet ağına bağlanır. Özel bir altyapı gerekmez; iSCSI'deki gibi standart switch ve kablo kullanılır.

Trafik TCP/IP üzerinden akar. Bu, NFS'i kurulum açısından kolay kılan şeydir. Ama aynı zamanda daha önce konuştuğumuz riski de beraberinde getirir: **storage trafiği ağı paylaşırsa, ağdaki yoğunluk disk performansını etkiler.**

Bu yüzden ağ tarafında aynı kurallar geçerlidir:

* NFS trafiğini **ayrı VLAN'a** koyun
* Mümkünse **ayrı NIC** verin
* 1GbE'de kesinlikle ayırın; 10GbE'de paylaşabilirsiniz (vDS varsa NIOC ile pay ayırarak)
* Yedeklilik için host'tan sunucuya en az iki yol bulunsun

### 3. VMkernel Portu

Host'un NFS sunucusuna ulaşmak için kullandığı arayüzdür. Bu noktada ağ serisinde öğrendiklerimiz devreye girer.

ESXi, NFS paylaşımına VMkernel portu üzerinden bağlanır. Host tarafında yapmanız gerekenler:

1. Bir vSwitch üzerinde VMkernel portu oluşturun
2. Ona statik bir IP verin — NFS sunucusuyla aynı ağda olmalı
3. Uplink'inin, NFS sunucusuna ulaşan switch'e bağlı olduğundan emin olun

Burada iSCSI'den önemli bir fark vardır: **NFS'te port binding yoktur.** iSCSI'de her NIC için ayrı vmk oluşturup binding yapıyorduk. NFS v3 tek bağlantı üzerinden çalışır, o yüzden böyle bir yapılandırma bulunmaz.

Peki NFS'te yedeklilik nasıl sağlanır? **NIC teaming ile.** VMkernel portunun bulunduğu vSwitch'e iki uplink verirsiniz; biri düşerse diğeri devralır. Yani iSCSI'nin tersine, burada yedeklilik ağ katmanından gelir.

Bir ek not: NFS v4.1 çoklu yol (multipathing) destekler. Ancak bazı vSphere özellikleriyle uyumluluk farkları olduğu için çoğu ortam hâlâ v3 kullanır.

### 4. ESXi Host: Datastore Olarak Bağlama

Son adımda host'a paylaşımı tanıtırsınız. Üç bilgi gerekir:

* **Sunucunun IP adresi:** Örneğin `192.168.10.50`
* **Klasör yolu:** Örneğin `/mnt/vmstore`
* **Datastore adı:** Host'ta görünecek isim, örneğin `NFS-Datastore-01`

Bu bilgileri girdiğinizde ESXi paylaşıma bağlanır ve klasör datastore olarak listede belirir. VMFS'te olduğu gibi biçimlendirme adımı yoktur; klasör zaten hazırdır.

#### Kritik kural: Aynı isim, aynı yol

Cluster ortamında bu datastore'u tüm host'lara eklerseniz, üç bilgi de **birebir aynı** olmalıdır:

* Aynı IP (biri hostname, diğeri IP kullanırsa host'lar bunu farklı datastore sanar)
* Aynı klasör yolu (büyük-küçük harf dahil)
* Aynı datastore adı

Küçük bir fark bile vMotion ve HA'yı bozar; host'lar aynı depolamayı iki ayrı datastore olarak görür. Bu, NFS kurulumlarında en sık yapılan hatalardan biridir.

### NFS'in Artıları ve Eksileri

**Artıları:**

* Kurulumu basittir. LUN oluşturma, IQN tanımlama, port binding gibi adımlar yoktur.
* Kapasite genişletmek kolaydır; sunucu tarafında klasörü büyütmeniz yeterlidir.
* Thin provisioning doğal olarak çalışır; sadece kullanılan alan yer kaplar.
* Dosya seviyesinde çalıştığı için sunucu tarafından yedeklemek kolaydır.

**Eksileri:**

* **Boot from SAN yapılamaz.** Sunucu bir klasörden açılamaz.
* **RDM kullanılamaz.** Ortada ham bir disk yoktur ki VM'e verilebilsin.
* NFS v3 tek yol üzerinden çalışır; çoklu yol için v4.1 gerekir.
* Yoğun I/O gerektiren iş yüklerinde blok tabanlı çözümlerin gerisinde kalabilir.

### Ne Zaman NFS Seçilir?

Karar genelde şu noktalarda netleşir:

**NFS'e uygun durumlar:** Genel amaçlı sunucu iş yükleri, ISO ve şablon depolama, yedekleme alanı, elinizde zaten iyi bir NAS varsa, ekipte blok storage tecrübesi yoksa.

**iSCSI veya FC'ye uygun durumlar:** RDM gerektiren uygulamalar (paylaşımlı disk isteyen Windows cluster'ları), storage'dan boot etme ihtiyacı, gecikmeye çok duyarlı veritabanları.

Pratikte birçok ortam ikisini birlikte kullanır: VM diskleri iSCSI'de, ISO ve şablonlar NFS'te. Bu, her teknolojiyi güçlü olduğu yerde kullanan makul bir yaklaşımdır.

### Kurulum Sonrası Doğrulama

Bağlantı kurulduktan sonra iki şeyi kontrol edin:

```bash
# Bağlı NFS datastore'ları listele
esxcli storage nfs list

# NFS sunucusuna erişimi test et
vmkping -I vmk1 192.168.10.50
```

`vmkping` komutundaki `-I` parametresi, testin doğru arayüzden yapıldığını garanti eder. Sunucuya ulaşılamıyorsa sorun genelde üç yerdedir: VMkernel portunun IP'si yanlış ağda, `/etc/exports` tanımı host'un IP'sini kapsamıyor, ya da arada bir firewall NFS portunu (2049) engelliyor.

Yedekliliği de gerçekten test edin: bir kabloyu çekip sanal makinelerin çalışmaya devam ettiğini görün.

### Sonuç

NFS, vSphere'de en kolay kurulan depolama tipidir. Özetle:

* **iSCSI disk verir, NFS klasör verir.** ESXi klasörü olduğu gibi datastore olarak bağlar, biçimlendirme yapmaz.
* Zincir dört parçadan oluşur: **NFS sunucusu → ağ → VMkernel portu → host.**
* Sunucu tarafında klasörü paylaşırken `no_root_squash` ayarını atlamayın; ESXi root ile bağlanır.
* **NFS'te port binding yoktur**; yedeklilik NIC teaming ile sağlanır.
* Cluster'da tüm host'lara **aynı IP, aynı yol, aynı isimle** ekleyin; yoksa vMotion ve HA çalışmaz.
