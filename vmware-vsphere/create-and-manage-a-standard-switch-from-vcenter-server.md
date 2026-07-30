---
icon: jedi-order
---

# Create and Manage a Standard switch From vCenter Server

Serinin önceki bölümlerinde tüm sanal ağ yapılandırmasını doğrudan host'a bağlanarak, ESXi Host Client üzerinden gerçekleştirdik. Ancak birden fazla host'un bulunduğu her ortamda yönetimin asıl merkezi **vCenter Server**'dır — ve host üzerinde yaptığımız her işlemin vCenter arayüzünde birebir karşılığı vardır: switch oluşturmak, port group ve VMkernel eklemek, uplink atamak, politikaları düzenlemek.

Bu makalede aynı işlemlerin vCenter tarafındaki akışını ele alacak; **Add Networking** sihirbazının üç bağlantı tipini, sihirbazın hangi adımda neyi zorunlu tuttuğunu ve çok host'lu ortamlarda Standard Switch yönetiminin yapısal sınırını inceleyeceğiz.

### Host Ağ Yapılandırmasına vCenter'dan Bakmak

vCenter envanterinde ilgili host'u seçip **Configure → Networking** bölümüne geldiğinizde, ağ yapılandırmasının tamamı dört görünümde karşınıza çıkar:

* **Virtual Switches:** Host'taki tüm Standard Switch'ler ve topolojileri — hangi port group hangi switch'te, hangi vmk hangi IP ile, hangi uplink bağlı. Önceki bölümlerde kurduğumuz yapı burada bütün olarak görünür: `vSwitch0` (VM Network + Management), yeni switch (VM port group + VMkernel portları) ve uplink'siz private switch.
* **VMkernel Adapters:** Tüm vmk portları tek listede — her birinin IP'si, bağlı port group'u ve etkin servisleri (`vmk0` management, `vmk1` management + vMotion, `vmk2` vSAN gibi).
* **Physical Adapters:** Host'taki tüm vmnic'ler, hangi switch'e atandıkları, hız ve MAC bilgileri.
* **TCP/IP Configuration:** Stack'ler, DNS ve gateway yapılandırması.

Mevcut nesnelerin düzenlenmesi de buradan yapılır: bir port group'u veya VMkernel portunu seçip **Edit** dediğinizde isim, VLAN ID, security, traffic shaping ve teaming ayarlarının tümüne — Host Client'taki ile aynı içerikle — erişirsiniz.

#### "Uplink redundancy lost" uyarısı ve alarm disiplini

Tek uplink'li switch'i olan bir host, vCenter'da **"Network uplink redundancy lost"** alarmı üretir. Arayüz bu alarmı **Reset to Green** ile kapatmanıza izin verir — ancak burada önemli bir operasyon prensibi vardır: **alarmı sıfırlamak sorunu çözmez, yalnızca görünmez yapar.** Reset to Green'in meşru kullanımı, kök nedeni giderdikten sonra durumun yeniden değerlendirilmesini tetiklemektir; lab ortamında bilinçli olarak tek uplink'le çalışıyorsanız kapatabilirsiniz, production'da ise doğru refleks alarma değil eksik uplink'e müdahale etmektir. Alarmları düzeltmeden sıfırlama alışkanlığı, gerçek bir kesintinin habercisi olan sinyallerin de sessizce kapatılmasıyla sonuçlanır.

### Add Networking Sihirbazı: Üç Bağlantı Tipi

vCenter'da ağ ekleme işlemlerinin tamamı tek bir giriş noktasından başlar: host'a sağ tıklayıp (veya Configure → Networking üzerinden) **Add Networking**. Sihirbazın ilk adımı size üç seçenek sunar ve doğru seçimi yapmak, önceki bölümlerde netleştirdiğimiz kavram ayrımının doğrudan uygulamasıdır:

#### 1. VMkernel Network Adapter

Host servisleri (vMotion, iSCSI, NFS, Fault Tolerance, vSAN, management, replication) için vmk portu oluşturur. Akış, Host Client'taki ile aynıdır: switch seçimi → port group adı → servis seçimi → IP yapılandırması.

Sihirbaz burada size mevcut switch'lerin tümünü sunar — **uplink'siz private switch dahil**. Teknik olarak private switch üzerinde de vmk oluşturabilirsiniz; ancak bu anlamsızdır: fiziksel adaptörü olmayan bir switch'teki servis trafiği host'tan hiçbir yere çıkamaz. vMotion, vSAN veya storage trafiği taşıyacak her vmk, mutlaka uplink'li bir switch üzerinde yaşamalıdır. Sihirbaz sizi bu hatadan korumaz; seçimin mantığı size aittir.

Örnek bir akış — yeni switch üzerine vSAN için vmk eklemek:

1. Connection type: **VMkernel Network Adapter** → Next
2. Select target device: mevcut switch'lerden uplink'li olanı seçin
3. Port properties: isim (`vmk-vSAN` gibi), VLAN ID, TCP/IP stack (default) ve servis olarak yalnızca **vSAN** işaretli
4. IPv4 settings: **static IP** (adresleme planınıza uygun, çakışmayan bir adres) ve subnet
5. Review → Finish

İşlem sonrası switch topolojisinde yeni vmk, IP'siyle birlikte görünür. Serinin önceki bölümündeki prensip burada da geçerlidir: **her servise kendi vmk portu** — bu örnekte vSAN, management veya vMotion ile aynı porta bindirilmemiştir. (vSAN özelinde bir ek not: bu yapılandırma cluster'daki **her host'ta** yapılmalı ve vSAN vmk'ları tercihen aynı subnet'te olmalıdır; tek host'ta vSAN vmk'ı tanımlamak yalnızca hazırlıktır.)

#### 2. Physical Network Adapter

Mevcut bir switch'e uplink atamak için kullanılır. Bu akışın diğerlerinden kritik bir farkı vardır: **fiziksel adaptör seçimi zorunlu adımdır.** Host'ta boşta vmnic yoksa sihirbaz ilerlemenize izin vermez — "there are no free network adapters" durumunda işlem ancak iptal edilebilir. Bir private/local switch'i sonradan dış dünyaya açmak (uplink kazandırmak) istediğinizde kullanacağınız akış budur; ön koşulu, serbest bir fiziksel adaptörün varlığıdır.

#### 3. Virtual Machine Port Group for a Standard Switch

Sanal makineler için port group oluşturur. Akışta önce hedef switch'i seçersiniz — mevcutlardan biri veya **New standard switch** ile yenisi. Yeni switch seçeneğinde sihirbaz uplink atama adımını da içerir; ancak bu adım, physical adapter akışının aksine **atlanabilir**: adaptör eklemeden Next dediğinizde _"There are no active physical network adapters for this switch"_ uyarısı gelir ve OK ile devam edebilirsiniz. Sonuç, önceki bölümde kurduğumuz uplink'siz izole switch'in vCenter üzerinden oluşturulmuş halidir.

Bu asimetri bilinçlidir ve sihirbazın mantığını özetler: **port group uplink'siz de anlamlıdır** (private network senaryosu), **uplink ataması ise adaptörsüz anlamsızdır** — biri uyarıyla geçilebilir, diğeri kesin gerekliliktir.

### Management Servisi ve Arayüz Erişimi Üzerine Bir İncelik

Yeni oluşturduğunuz bir vmk'nın IP'sine tarayıcıdan gittiğinizde, host'un giriş ekranıyla karşılaşır ve oturum açabilirsiniz — üstelik o vmk'da Management servisi işaretli olmasa bile. Bu davranış kafa karıştırıcı olabilir, çünkü iki farklı şeyi birbirinden ayırmak gerekir:

* **Arayüze erişilebilirlik:** Host'un web arayüzü, host'a ait vmk IP'leri üzerinden genel olarak erişilebilirdir; bir IP'ye tarayıcıyla ulaşabilmeniz, o vmk'nın "management portu" olduğu anlamına gelmez.
* **Management servis etiketi:** vmk üzerindeki Management işareti, host'un **yönetim düzlemi trafiğinin** (vCenter iletişimi, HA heartbeat) hangi arayüzü kullanacağını belirleyen tanımdır.

Pratik sonuç şudur: bir vmk'yı gerçek bir yedek yönetim yolu yapmak istiyorsanız — önceki bölümde kurduğumuz ikinci management portu senaryosu — Management servisini **açıkça işaretlemeniz** gerekir; tarayıcıdan erişebiliyor olmak yeterli gösterge değildir. Servis vmk'ları (yalnızca vSAN veya yalnızca vMotion işaretli olanlar) ise amaçları dışında yönetim yolu olarak planlanmamalıdır.

### Çok Host'lu Ortamlarda Yapısal Sınır: vCenter Bir Pencere, Merkez Değil

vCenter tüm host'ların ağ yapılandırmasını tek arayüzde gösterir — ancak Standard Switch söz konusu olduğunda bunun ne anlama gelip gelmediğini doğru okumak gerekir:

* vCenter, Standard Switch yapılandırmasını **merkezileştirmez**; yalnızca her host'un kendi yerel yapılandırmasına açılan ortak bir pencere sunar.
* İkinci bir host'a aynı yapıyı kurmak istediğinizde (aynı switch'ler, aynı port group adları, aynı VLAN'lar), tüm adımları o host için **baştan tekrarlarsınız**. Üçüncü, dördüncü, onuncu host için de aynısı geçerlidir.
* Bu tekrar, yalnızca zaman maliyeti değil **tutarlılık riskidir**: port group adında tek bir harf farkı vMotion uyumluluğunu bozar ve bu tür sapmalar elle yönetimde kaçınılmaz olarak birikir.

Bu sınırla başa çıkmanın üç kademesi vardır:

1. **PowerCLI ile otomasyon:** Ağ yapılandırmasını script olarak tanımlayıp her yeni host'a aynı script'i uygulamak, elle tekrarın hatasını ortadan kaldırır ve Standard Switch ile çalışmaya devam eden ortamlar için en erişilebilir çözümdür.
2. **Host Profiles:** Referans bir host'un yapılandırmasını profil olarak çıkarıp diğer host'lara uygulamak ve sapmaları (compliance) denetlemek — Enterprise Plus lisansı gerektirir.
3. **vSphere Distributed Switch:** Sorunu kökten çözen mimari adımdır: yapılandırma vCenter'da **bir kez** tanımlanır, tüm host'lara otomatik dağıtılır ve tutarlılık tasarım gereği garanti edilir. Host sayısı arttıkça Standard Switch'ten vDS'e geçişin asıl gerekçesi tam olarak budur.

### Sonuç

vCenter, Standard Switch yönetiminde yeni bir yetenek eklemez — Host Client'ta yaptığınız her şeyi, tüm host'ları tek envanterde görerek yapmanızı sağlar. Özetle:

* **Add Networking** sihirbazının üç bağlantı tipi, serinin kavram haritasıyla birebir örtüşür: vmk portu (host servisleri), physical adapter (uplink atama) ve VM port group. Uplink ataması adaptörsüz ilerlemez; port group ise bilinçli olarak uplink'siz oluşturulabilir.
* Servis vmk'larını her zaman uplink'li switch'lere yerleştirin; arayüze erişilebilen her IP'nin yönetim yolu olmadığını, gerçek yedek yönetim yolunun Management servisinin açıkça işaretlenmesini gerektirdiğini unutmayın.
* Alarmları düzeltmeden sıfırlamayın; Reset to Green teşhisin sonucu olmalı, yerine geçmemelidir.
* vCenter'ın sunduğu tek arayüz, Standard Switch'in host-bazlı doğasını değiştirmez. Host sayısı büyüdükçe tutarlılığı elle değil; PowerCLI, Host Profiles veya nihai çözüm olarak vDS ile güvence altına alın.

Bu bölümle birlikte Standard Switch'in hem host hem vCenter tarafındaki tüm yönetim yüzeyini kapsamış olduk. Serinin doğal devamı, yukarıda sınırlarını çizdiğimiz sorunun mimari cevabı: yapılandırmanın merkezden tanımlandığı, politikaların tüm cluster'a tek hamlede dağıtıldığı **vSphere Distributed Switch** dünyası.
