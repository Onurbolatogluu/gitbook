---
icon: hand-holding-droplet
---

# vSphere iSCSI SAN components

Bir önceki bölümde vSphere'in desteklediği depolama teknolojilerine baktık ve iSCSI'nin neden en yaygın tercih olduğunu gördük: mevcut Ethernet ağını kullanır, Fibre Channel'a göre çok daha ucuzdur ve modern 10/25GbE ağlarda çoğu iş yükü için yeterli performansı verir. Fibre Channel'ın maliyeti yüzünden orta ölçekli birçok ortamda tek gerçekçi paylaşımlı depolama seçeneği iSCSI'dir.

Bu makalede bir iSCSI SAN'ın hangi parçalardan oluştuğunu anlatacağız. Amaç, yapılandırmaya geçmeden önce her parçanın ne işe yaradığını netleştirmek.

### Zincirin Parçaları

iSCSI'de veri şu yolu izler:

```
[Fiziksel diskler] → [LUN'lar] → [Storage processor'lar]
        → [Ethernet ağı] → [iSCSI initiator] → [ESXi host]
```

Her parçayı sırayla açalım.

### 1. Fiziksel Diskler ve LUN

iSCSI storage sistemi, içinde çok sayıda disk barındıran bir kabindir. Bu diskler ESXi'ye tek tek verilmez. Önce gruplanır ve mantıksal parçalara bölünür.

**LUN (Logical Unit Number)**, storage sisteminin dışarıya sunduğu bu mantıksal disktir. Basit bir örnek:

* Kabinde 500 GB'lık 10 disk var
* İki disk birleştirilip **LUN 1** olarak sunuluyor (1 TB)
* İki disk daha **LUN 2** oluyor (1 TB)
* Üç disk **LUN 3** oluyor (1,5 TB)

ESXi bu LUN'ları birer disk olarak görür ve üzerlerine VMFS datastore kurar. Host, arkada kaç disk olduğunu ve nasıl gruplandığını bilmez; sadece kendisine verilen mantıksal diski görür.

**Bu gruplamayı genelde siz yapmazsınız.** Kurumsal storage sistemleri satın alındığında LUN'lar hazır gelir. Sizin işiniz, hazır LUN'ları host'lara sunmak ve ESXi tarafını yapılandırmaktır. Yine de bu katmanı bilmek, kapasite planlarken ve performans sorunu ararken işinize yarar.

#### LUN boyutlandırma

Birkaç basit prensip:

* **Büyük LUN mu, küçük LUN mu?** Büyük LUN'lar yönetimi kolaylaştırır. Küçük LUN'lar iş yüklerini birbirinden ayırır, bir sorun çıkarsa etkisi dar kalır. Eskiden LUN başına VM sayısını sınırlayan kilitleme sorunları vardı; modern vSphere'de bu büyük ölçüde çözüldü (VAAI ATS).
* **Performans katmanı:** Farklı disk tiplerinden (SSD, SAS, NL-SAS) oluşan LUN'ları ayrı datastore olarak sunun. Böylece hangi VM'in hızlı diske, hangisinin yavaş diske gideceğine karar verebilirsiniz.
* **Büyüme payı:** LUN'ları sonradan büyütmek mümkün ama planlı bir iştir. Baştan biraz pay bırakmak sonraki işi azaltır.

### 2. Storage Processor (SP)

Storage sisteminin beynidir. Ağdan gelen istekleri karşılar, LUN'lara yönlendirir, cevabı geri gönderir.

Kurumsal iSCSI sistemlerinin neredeyse hepsi **en az iki storage processor** ile gelir. Büyük sistemlerde dört veya daha fazla olabilir. Sebebi iki tane:

* **Yedeklilik:** Bir SP arızalanırsa veya güncelleme için yeniden başlarsa, diğeri işi devralır. Tek SP'li bir sistem, tüm sanal ortamınız için tek arıza noktasıdır.
* **Performans:** Yük iki denetleyici arasında bölünür.

**Önemli bir detay:** İki SP'nin olması tek başına yetmez. Her SP **farklı bir fiziksel switch'e** bağlanmalıdır. Aksi halde switch arızasında iki yol da kesilir. Host tarafındaki NIC'ler için de aynı kural geçerlidir.

Depolama yolunda yedeklilik, ağ yedekliliğinden daha kritiktir. Ağ kesilirse VM'lere erişilemez; storage kesilirse VM'ler diskini kaybeder.

Bir de SP'lerin çalışma şeklini bilmek gerekir. Bazı sistemler **active/active** çalışır (her SP her LUN'a hizmet verir), bazıları **active/passive** (her LUN'un bir sahibi vardır). Bu fark, ESXi tarafında seçeceğiniz yol politikasını etkiler.

### 3. Ağ

iSCSI'nin en belirgin özelliği, özel bir altyapı istememesidir. Storage processor'lar normal Ethernet kablosuyla ağa bağlanır, trafik TCP/IP üzerinden akar. Fibre Channel'daki gibi ayrı kart, ayrı switch, ayrı kablo gerekmez.

Ama bu kolaylık, en sık yapılan hatanın da kaynağıdır: **storage trafiğinin normal ağ trafiğiyle aynı yolu paylaşması.**

Depolama trafiği gecikmeye çok duyarlıdır. Bir yedekleme işi ya da yoğun bir vMotion aynı hattı doldurduğunda, storage trafiği sıkışır. Sonuç, sanal makinelerde disk gecikmesi olarak görünür — donmalar, uygulama zaman aşımları.

**Ağ tarafında yapılması gerekenler:**

* **Ayırın:** iSCSI trafiği kendi VLAN'ında olsun, tercihen kendi NIC'leri üzerinde. Kritik ortamlarda ayrı switch çifti kullanılır.
* **Yedekleyin:** Host'tan storage'a en az iki bağımsız yol olsun. Bu yollar farklı NIC, farklı switch ve farklı SP üzerinden geçmeli.
* **Jumbo Frames (MTU 9000):** Verimi artırır. Ama zincirin tamamında aynı olmak zorundadır — VMkernel portu, vSwitch, fiziksel switch ve storage. Bir halkada 1500 kalırsa, bulması zor performans sorunları çıkar.
* **Switch ayarları:** iSCSI portlarında PortFast/Edge açık olsun, flow control doğru ayarlansın.

### 4. iSCSI Initiator

Storage tarafına **target**, host tarafına **initiator** denir. Initiator, ESXi'nin storage'ı bulmasını ve LUN'lara bağlanmasını sağlayan parçadır.

İki tipi var:

#### Software initiator

ESXi'nin içinde çalışan bir yazılımdır. Host'un normal network kartlarını kullanır, ek donanım istemez.

* **Artısı:** Bedava, her host'ta hazır, kurulumu kolay.
* **Eksisi:** iSCSI işlemleri host'un işlemcisini kullanır. Modern işlemcilerde bu yük hissedilmez.

#### Hardware initiator (HBA)

iSCSI işini kendi üzerinde yapan özel bir karttır.

* **Artısı:** İşlemciyi yormaz. Storage'dan boot etme (boot from SAN) imkânı verir.
* **Eksisi:** Para verip almak gerekir, yönetimi biraz daha karmaşıktır.

İki alt tipi vardır: **bağımsız (independent)** HBA her şeyi kendi yapar, kendi IP'si vardır. **Bağımlı (dependent)** HBA ise iSCSI işini üstlenir ama IP yapılandırması için ESXi'nin VMkernel portuna ihtiyaç duyar.

#### Hangisini seçmeli?

Cevap basit: **özel bir gerekçeniz yoksa software initiator kullanın.** Ortamların büyük çoğunluğu böyle çalışır. HBA sadece iki durumda gerekir: işlemci yükü ölçülebilir şekilde sorun oluyorsa, ya da storage'dan boot etmeniz gerekiyorsa.

Bir yanlış anlaşılmayı da netleştirelim: HBA ağ yükünü azaltmaz. Paketler yine aynı Ethernet ağından geçer. HBA'nın rahatlattığı yer işlemcidir. Ağı rahatlatmak istiyorsanız çözüm ayrı NIC ve ayrı VLAN'dır — HBA değil.

### 5. Storage Nasıl Bulunur?

Host, ağdaki storage'ı kendiliğinden bulmaz. Adresini siz verirsiniz. İki yöntem var:

* **Dynamic Discovery (SendTargets):** ESXi'ye storage processor'ın IP adresini girersiniz. Host o adrese bağlanıp "hangi hedefleri sunuyorsun?" diye sorar, storage kendi listesini döner. Yaygın kullanılan yöntem budur.
* **Static Discovery:** Hedefleri tek tek elle yazarsınız. Nadiren gerekir.

Adresi girdikten sonra host bir tarama (rescan) yapar ve erişebildiği LUN'ları listeler.

#### IQN ve LUN masking

Her initiator ve target'ın benzersiz bir adı vardır: **IQN (iSCSI Qualified Name)**. Şuna benzer:

```
iqn.1998-01.com.vmware:host01-1a2b3c4d
```

Storage tarafında "hangi LUN hangi host'a görünsün" tanımı bu IQN'lere göre yapılır. Buna **LUN masking** denir. Fibre Channel'daki zoning'in iSCSI karşılığıdır.

Bu tanım önemlidir: yanlış yapılandırılırsa bir host, başka bir cluster'ın LUN'unu görebilir ve veri bozulmasına yol açabilir.

#### Güvenlik: CHAP

iSCSI trafiği varsayılan olarak şifrelenmez. Erişim kontrolü için **CHAP** kullanılır — basit bir kullanıcı adı/parola doğrulaması gibi düşünebilirsiniz.

* **One-way CHAP:** Storage, host'u doğrular.
* **Mutual CHAP:** İkisi birbirini doğrular.

Ayrı bir storage VLAN'ında çalışıyorsanız risk düşüktür. Yine de CHAP'i açmak, yanlış yapılandırılmış bir host'un başkasının LUN'una bağlanmasını engelleyen basit bir korumadır.

### 6. Çoklu Yol (Multipathing)

Host ile storage arasında birden fazla yol varsa, ESXi bunları yönetir. Üç politika vardır:

* **Fixed:** Belirlediğiniz yol kullanılır. O yol düşerse alternatife geçer.
* **Most Recently Used (MRU):** Son çalışan yol kullanılmaya devam eder. Genelde active/passive sistemlerde varsayılandır.
* **Round Robin:** Yollar arasında sırayla dağıtım yapar. Active/active sistemlerde hem yedeklilik hem hız sağlar. Modern ortamlarda önerilen budur.

**Kritik nokta:** Software initiator'da çoklu yol kurmanın yolu **iSCSI port binding**'dir. Her fiziksel NIC için ayrı bir VMkernel portu oluşturulur ve bunlar initiator'a bağlanır.

Bu yapılmazsa, iki NIC'iniz olsa bile gerçek çoklu yol elde edemezsiniz. Kablolar doğru takılı olsa da ESXi tek yol üzerinden çalışır ve bir kablo çektiğinizde datastore düşer. iSCSI kurulumlarında en sık atlanan adım budur.

Ayrıca storage üreticinizin önerilerine bakın. Çoğu üretici kendi sistemi için özel bir yol politikası ve IOPS değeri tavsiye eder.

### Kurulum Sırası

Parçaları bir araya getirdiğimizde tipik akış şöyledir:

1. Storage ağa bağlanır; SP'ler farklı switch'lere dağıtılır
2. LUN'lar oluşturulur (veya hazır gelir) ve host IQN'lerine sunulur
3. ESXi'de storage için ayrı VMkernel portları ve ayrı VLAN yapılandırılır
4. Software iSCSI adapter açılır, port binding yapılır
5. Dynamic discovery ile storage IP'si girilir, gerekiyorsa CHAP ayarlanır
6. Rescan yapılır; görünen LUN'lar üzerine VMFS datastore kurulur
7. Yol politikası (genelde Round Robin) ayarlanır

Son adımdan sonra bir şey daha yapın: **bir kabloyu çekin ve VM'lerin çalışmaya devam ettiğini görün.** Test edilmemiş yedeklilik, yedeklilik sayılmaz.

Doğrulama için:

```bash
# LUN başına yol sayısını ve politikayı gör
esxcli storage nmp device list

# Storage'a erişimi test et
vmkping -I vmk2 192.168.20.10
```

### Sonuç

iSCSI SAN karmaşık görünür ama birkaç net parçadan oluşur. Özetle:

* **LUN**, disklerin gruplanmasıyla oluşan mantıksal diskdir. ESXi arkadaki yapıyı görmez, sadece kendisine verilen LUN'u bilir.
* **Storage processor** storage'ın ağa açılan kapısıdır. En az iki tane olmalı ve **farklı switch'lere** bağlanmalıdır.
* **Ağ** normal Ethernet'tir. iSCSI'yi ucuz kılan bu, ama trafiği **ayırmak ve yedeklemek** şarttır.
* **Initiator** host tarafındaki bağlantı noktasıdır. Özel bir gerekçe yoksa **software** yeterlidir; HBA ağ yükünü değil işlemci yükünü azaltır.
* **Storage kendiliğinden bulunmaz.** IP'sini siz girersiniz. Erişim kontrolü **IQN tabanlı LUN masking** ve **CHAP** ile yapılır.
* **Çoklu yol**, software initiator'da **port binding** ile kurulur. Bu adım atlanırsa yedeklilik kâğıt üzerinde kalır.

Sıradaki bölümde bunları uygulamaya dökeceğiz: ESXi'de software iSCSI adapter'ı açmak, VMkernel portlarını ve port binding'i yapılandırmak, storage'ı tanıtmak ve LUN üzerine VMFS datastore kurmak.
