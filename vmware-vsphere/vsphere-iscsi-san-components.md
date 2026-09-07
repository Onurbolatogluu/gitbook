---
icon: hand-holding-droplet
---

# vSphere iSCSI SAN components

Bir önceki bölümde vSphere'in desteklediği depolama teknolojilerine genel bir bakış attık ve iSCSI'nin neden en yaygın tercih olduğunu ortaya koyduk: mevcut Ethernet altyapısını kullanır, maliyeti Fibre Channel'a göre belirgin şekilde düşüktür ve modern 10/25GbE ağlarla çoğu iş yükü için fazlasıyla yeterli performans sunar. Fibre Channel'ın yüksek maliyeti nedeniyle birçok orta ölçekli ortamda pratikte tek gerçekçi paylaşımlı depolama seçeneği iSCSI'dir.

Bu makalede bir iSCSI SAN'ın hangi bileşenlerden oluştuğunu, verinin fiziksel diskten sanal makineye kadar hangi katmanlardan geçtiğini ve ESXi tarafında bağlantının nasıl kurulduğunu ele alıyoruz. Amaç, yapılandırma adımlarına geçmeden önce zincirin her halkasının ne işe yaradığını netleştirmek.

### Genel Tablo: Zincirin Halkaları

Bir iSCSI SAN'da veri şu yolu izler:

```
[Fiziksel Diskler] → [RAID / LUN'lar] → [Storage Processor'lar]
        → [Ethernet Ağı / Switch] → [iSCSI Initiator] → [ESXi Host]
```

Her halkayı sırayla inceleyelim.

### 1. Fiziksel Diskler ve LUN Kavramı

iSCSI storage sistemi, içinde çok sayıda fiziksel disk barındıran bir kabindir. Bu diskler ESXi'ye tek tek sunulmaz; önce storage sistemi tarafından gruplanır ve mantıksal birimlere bölünür.

**LUN (Logical Unit Number)**, storage sisteminin dışarıya sunduğu bu mantıksal disk birimidir. Örnek bir yapılandırma:

* Kabinde 500 GB'lık 10 disk bulunuyor
* İki disk bir RAID grubunda birleştirilip **LUN 1** olarak sunuluyor (1 TB)
* İki disk daha **LUN 2** oluşturuyor (1 TB)
* Üç disk **LUN 3** olarak yapılandırılıyor (1,5 TB)

ESXi bu LUN'ları birer disk olarak görür ve üzerlerinde VMFS datastore oluşturur. Yani host, arkadaki fiziksel disk sayısından ve RAID yapısından habersizdir; yalnızca kendisine sunulan mantıksal birimi bilir.

**Pratikte bu yapılandırmayı genellikle siz yapmazsınız.** Kurumsal storage sistemleri satın alındığında, RAID grupları ve LUN'lar ya üretici ya da tedarikçi tarafından hazırlanmış olarak gelir. Sizin göreviniz, hazır LUN'ları host'lara sunmak ve ESXi tarafında yapılandırmaktır. Yine de bu katmanı anlamak, kapasite planlaması ve performans sorunlarını teşhis ederken kritik önem taşır.

#### LUN tasarımı üzerine notlar

Boyutlandırma kararında birkaç prensip vardır:

* **Çok sayıda küçük LUN mu, az sayıda büyük LUN mu?** Büyük LUN'lar yönetimi basitleştirir; küçük LUN'lar ise iş yüklerini birbirinden yalıtır ve bir sorunun etki alanını daraltır. Modern vSphere sürümlerinde VMFS kilitleme mekanizmaları geliştiği için (VAAI ATS), eskiden LUN başına VM sayısını sınırlayan kaygılar büyük ölçüde azalmıştır.
* **Performans katmanları:** Farklı disk tiplerinden (SSD, SAS, NL-SAS) oluşturulan LUN'ları ayrı datastore'lar olarak sunmak, iş yüklerini performans ihtiyacına göre yerleştirmenizi sağlar.
* **Genişleme payı:** LUN'ları sonradan büyütmek mümkündür ancak planlı bir işlemdir; baştan makul bir büyüme payı bırakmak operasyonel yükü azaltır.

### 2. Storage Processor'lar (SP)

Storage sisteminin "beyni" ve ağa açılan kapısıdır. Gelen iSCSI isteklerini karşılar, LUN'lara yönlendirir ve cevabı geri gönderir.

Kurumsal iSCSI sistemlerinin neredeyse tamamı **en az iki storage processor** ile gelir; daha büyük sistemlerde dört veya daha fazlası bulunabilir. Bunun iki gerekçesi vardır:

* **Yüksek erişilebilirlik:** Bir SP arızalandığında veya firmware güncellemesi için yeniden başlatıldığında, diğer SP hizmeti devralır. Tek SP'li bir sistem, tüm sanal ortamınız için bir single point of failure demektir.
* **Performans:** Yük iki denetleyici arasında paylaştırılarak toplam verim artırılır.

**Kritik tasarım notu:** İki SP'nin varlığı tek başına yeterli değildir. Her SP'nin **farklı bir fiziksel switch'e** bağlanması gerekir — aksi halde switch arızasında her iki yol da kesilir. Aynı şekilde host tarafındaki NIC'ler de farklı switch'lere dağıtılmalıdır. Depolama yolunda yedeklilik, ağ yedekliliğinden daha kritiktir: ağ kesintisinde VM'ler erişilemez hale gelir, storage kesintisinde ise diskleri kaybolur.

Ayrıca SP'lerin çalışma modunu bilmek gerekir: bazı array'ler **active/active** (her SP her LUN'a hizmet verir), bazıları **active/passive** (her LUN bir SP'ye "sahiptir") çalışır. Bu fark, ESXi tarafında seçeceğiniz path selection policy'yi doğrudan etkiler.

### 3. Ağ Katmanı

iSCSI'nin en belirgin özelliği, özel bir altyapı gerektirmemesidir: storage processor'lar standart Ethernet kablolarıyla ağa bağlanır ve trafik TCP/IP üzerinden akar. Fibre Channel'ın aksine ayrı HBA, ayrı switch ve ayrı kablolama gerekmez.

Ancak bu kolaylık, en sık yapılan hatanın da kaynağıdır: **iSCSI trafiğinin genel ağ trafiğiyle aynı yolu paylaşması.** Depolama trafiği gecikmeye son derece duyarlıdır; bir yedekleme işi ya da yoğun bir vMotion, aynı hattı paylaşan storage trafiğini etkilediğinde sanal makinelerde disk gecikmeleri (latency) olarak kendini gösterir.

**Ağ tarafında temel gereklilikler:**

* **İzolasyon:** iSCSI trafiği kendi VLAN'ında, tercihen kendi fiziksel adaptörleri üzerinde taşınmalıdır. Kritik ortamlarda ayrı bir fiziksel switch çifti kullanılır.
* **Yedeklilik:** Host'tan array'e giden en az iki bağımsız yol bulunmalı; bu yollar farklı NIC, farklı switch ve farklı SP üzerinden geçmelidir.
* **Jumbo Frames (MTU 9000):** Verimi artırır, ancak uçtan uca — VMkernel portu, vSwitch, fiziksel switch portları ve array — tutarlı olmak zorundadır. Zincirin bir halkasında 1500 kalırsa, sorunun teşhisi zor performans kayıpları yaşanır.
* **Flow control ve Spanning Tree:** iSCSI portlarında PortFast/Edge yapılandırması ve doğru flow control ayarları, gereksiz gecikmeleri önler.

### 4. iSCSI Initiator: Host Tarafındaki Bağlantı Noktası

Storage tarafına **target**, host tarafına **initiator** denir. Initiator, ESXi'nin iSCSI hedeflerini keşfetmesini ve LUN'lara bağlanmasını sağlayan bileşendir. İki tipi vardır:

#### Software iSCSI Initiator

ESXi'nin içinde çalışan yazılımsal bir bileşendir. Host'un standart fiziksel network adaptörlerini kullanır; ek donanım gerektirmez.

* **Avantajı:** Maliyetsizdir, her host'ta mevcuttur ve yapılandırması basittir.
* **Bedeli:** iSCSI protokol işlemleri host CPU'sunu kullanır. Modern işlemcilerde bu yük çoğu senaryoda ihmal edilebilir düzeydedir.

Pratikte ortamların büyük çoğunluğu software initiator kullanır ve bu, tavsiye edilen başlangıç noktasıdır.

#### Hardware iSCSI Initiator (HBA)

iSCSI işlemlerini kendi üzerinde gerçekleştiren özel bir adaptördür. İki alt tipi vardır:

* **Bağımsız (independent) HBA:** iSCSI oturumunu, TCP/IP ve kendi IP yapılandırmasını tamamen kendisi yönetir. ESXi'nin ağ katmanına ihtiyaç duymaz.
* **Bağımlı (dependent) HBA:** iSCSI offload yapan ancak IP yapılandırması için ESXi'nin VMkernel portlarına bağımlı olan adaptörlerdir.
* **Avantajı:** CPU yükünü devralır ve boot from SAN senaryolarını mümkün kılar.
* **Bedeli:** Ek donanım maliyeti ve yönetim karmaşıklığı.

#### Hangisini seçmeli?

Karar basittir: **ek donanım almak için özel bir gerekçeniz yoksa software initiator kullanın.** CPU offload ihtiyacı ölçülebilir şekilde ortaya çıkmadıkça ya da iSCSI üzerinden boot etme gereksinimi olmadıkça, hardware HBA'nın getirisi maliyetini karşılamaz.

### 5. Keşif (Discovery) Nasıl Çalışır?

Host, ağdaki iSCSI hedeflerini iki yöntemden biriyle bulur:

* **Dynamic Discovery (SendTargets):** ESXi'ye storage processor'ın IP adresini girersiniz; host o adrese bağlanıp "hangi hedefleri sunuyorsun?" diye sorar ve array kendisindeki hedefleri listeler. Yaygın kullanılan yöntem budur.
* **Static Discovery:** Hedefleri tek tek elle tanımlarsınız. Nadiren gerekir.

Buradaki önemli nokta şudur: keşif kendiliğinden olmaz, **hedefin adresini siz tanımlarsınız.** Yapılandırma sonrası host bir tarama (rescan) yapar ve erişebildiği LUN'ları listeler.

Her initiator ve target'ın **IQN (iSCSI Qualified Name)** adında benzersiz bir kimliği vardır — `iqn.1998-01.com.vmware:host01-1a2b3c4d` gibi. Array tarafında hangi LUN'un hangi host'a sunulacağı bu IQN'lere göre tanımlanır; bu işleme **LUN masking** denir ve Fibre Channel'daki zoning'in iSCSI karşılığıdır.

#### Güvenlik: CHAP

iSCSI trafiği varsayılan olarak şifrelenmez. Erişim denetimi için **CHAP (Challenge-Handshake Authentication Protocol)** kullanılır:

* **One-way CHAP:** Target, initiator'ı doğrular.
* **Mutual CHAP:** Karşılıklı doğrulama yapılır.

İzole bir storage VLAN'ında çalışıyorsanız risk düşüktür; yine de paylaşımlı altyapılarda CHAP'i etkinleştirmek, yanlış yapılandırılmış bir host'un başkasının LUN'una bağlanmasını önleyen basit ve etkili bir katmandır.

### 6. Çoklu Yol (Multipathing)

Host ile array arasında birden fazla fiziksel yol bulunduğunda, ESXi bu yolları **PSA (Pluggable Storage Architecture)** ile yönetir. Path selection policy seçenekleri:

* **Fixed:** Belirlenen tercih edilen yol kullanılır; o yol düşerse alternatife geçilir.
* **Most Recently Used (MRU):** Son çalışan yol kullanılmaya devam edilir; genellikle active/passive array'lerde varsayılandır.
* **Round Robin:** Yollar arasında sırayla dağıtım yapılır; active/active array'lerde hem yedeklilik hem verim sağlar ve modern ortamlarda önerilen politikadır.

Software initiator ile çoklu yol kurmanın yolu **iSCSI port binding**'dir: her fiziksel adaptör için ayrı bir VMkernel portu oluşturulur ve bunlar iSCSI initiator'a bağlanır. Bu yapılandırma olmadan birden fazla NIC'iniz olsa bile gerçek anlamda çoklu yol elde edemezsiniz — bu, iSCSI kurulumlarında en sık atlanan adımdır.

Ayrıca array üreticinizin **SATP/PSP** önerilerini kontrol edin; birçok üretici kendi array'i için özel bir path politikası ve IOPS değeri tavsiye eder.

### Pratikte Nasıl İlerlenir?

Bileşenleri bir araya getirdiğimizde tipik bir kurulum akışı şöyledir:

1. Storage sistemi ağa bağlanır; SP'ler farklı switch'lere dağıtılır.
2. Array üzerinde LUN'lar oluşturulur (veya hazır gelir) ve host IQN'lerine sunulur.
3. ESXi'de storage trafiği için ayrı VMkernel portları ve izole VLAN yapılandırılır.
4. Software iSCSI adapter etkinleştirilir, port binding yapılır.
5. Dynamic discovery ile array IP'si tanımlanır, gerekiyorsa CHAP ayarlanır.
6. Rescan yapılır; görünen LUN'lar üzerinde VMFS datastore oluşturulur.
7. Çoklu yol politikası (genellikle Round Robin) ayarlanır ve yol yedekliliği fiilen test edilir.

Son madde özellikle önemlidir: bir kabloyu çekip yolun gerçekten devraldığını doğrulamadan, yedekliliğin çalıştığını varsaymayın.

### Sonuç

iSCSI SAN, karmaşık görünen ancak birkaç net bileşenden oluşan bir yapıdır. Özetle:

* **LUN'lar**, fiziksel disklerin gruplanmasıyla oluşan mantıksal birimlerdir; ESXi arkadaki RAID yapısını görmez, yalnızca kendisine sunulan LUN'u bilir.
* **Storage processor'lar** array'in ağa açılan kapısıdır; en az iki tanesi bulunmalı ve **farklı fiziksel switch'lere** bağlanmalıdır.
* **Ağ katmanı** standart Ethernet'tir — bu iSCSI'yi ucuz kılan özelliktir, ancak trafiğin **izole edilmesi ve yedeklenmesi** şarttır.
* **Initiator** host tarafındaki bağlantı noktasıdır; özel bir gerekçe yoksa **software initiator** doğru tercihtir.
* **Discovery** elle tanımlanan hedef adresiyle başlar; erişim denetimi **IQN tabanlı LUN masking** ve **CHAP** ile sağlanır.
* **Çoklu yol**, software initiator'da **port binding** ile kurulur ve genellikle **Round Robin** politikasıyla kullanılır.

Bileşenleri ve aralarındaki ilişkiyi netleştirdiğimize göre, serinin devamında bunları uygulamaya dökebiliriz: ESXi üzerinde software iSCSI adapter'ın etkinleştirilmesi, VMkernel portlarının ve port binding'in yapılandırılması, hedeflerin tanıtılması ve sunulan LUN'lar üzerinde VMFS datastore oluşturulması.
