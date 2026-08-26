---
icon: rectangle-tall
---

# Create A New Distributed Port Group

Önceki bölümde datacenter seviyesinde bir Distributed Switch oluşturmuş ve onu "boş bir kabuk" olarak bırakmıştık. Bu kabuğu işlevsel kılan bileşenlerin başında **distributed port group** gelir: sanal makinelerin vDS'e bağlandığı, port yapılandırmasının ve politikaların tanımlandığı merkezi nesnedir.

Bu makalede vDS yönetim arayüzünü tanıyacak, yeni bir distributed port group'un nasıl oluşturulduğunu adım adım ele alacak ve sihirbazın en çok kafa karıştıran iki kavramını — **port binding** ve **port allocation** — production perspektifiyle netleştireceğiz.

### vDS Yönetim Arayüzüne Genel Bakış

Oluşturduğunuz Distributed Switch'i envanterde seçtiğinizde, yönetimin toplandığı birkaç görünüm karşınıza çıkar. Bunlardan en çok çalışacağınız iki tanesi şunlardır:

#### Configure → Properties

Switch'in temel özelliklerini düzenlediğiniz yerdir: adı, açıklaması, **uplink sayısı** ve gelişmiş ayarlar (MTU, discovery protokolü, yönetici iletişim bilgisi). Oluşturma sırasında belirlediğiniz uplink sayısını sonradan buradan artırıp azaltabilirsiniz — örneğin varsayılan 4'ten ortamınıza uygun bir değere çekmek mümkündür.

Bir uyarı: uplink sayısını **azaltmadan önce** o slotlara eşlenmiş fiziksel adaptör bulunmadığından emin olun; kullanımdaki bir uplink slotunu kaldırmak, ilgili host'ların bağlantısını etkiler.

#### Configure → Topology

vDS'in bütün resmini gösteren görünümdür. Standard Switch topolojisiyle aynı mantıkla okunur, ancak ölçeği farklıdır:

* **Sol taraf (virtual):** Distributed port group'lar ve içlerindeki VM/VMkernel sayıları
* **Sağ taraf (physical):** Uplink slotları ve bunlara eşlenmiş fiziksel adaptörler

Yeni oluşturulmuş bir vDS'te tipik görüntü şudur: bir distributed port group (sıfır VM ile) ve dört uplink slotu — hepsi **sıfır fiziksel adaptörle**. Bu, henüz hiçbir host'un vDS'e eklenmediğini ve dolayısıyla hiçbir vmnic'in bu slotlara eşlenmediğini gösterir. Varsayılan dört slotun anlamı da burada netleşir: her host'un bu switch'e dört adede kadar fiziksel adaptör bağlayabileceği bir çerçeve tanımlanmıştır; yedekli bir yapı için host başına en az iki slotun gerçekten doldurulması gerekir.

#### Networking envanterindeki iki kategori

Datacenter'ın Networking görünümünde vDS ile birlikte iki nesne tipi belirir ve ikisini ayırt etmek önemlidir:

* **Distributed Port Group:** Sanal makinelerin ve VMkernel portlarının bağlandığı, sizin oluşturduğunuz port group'lar. Switch'in _iç_ tarafıdır.
* **Uplink Port Group:** vDS ile birlikte otomatik oluşan, fiziksel adaptörlerin bağlandığı özel port group. Switch'in _dış_ tarafıdır ve host'lar eklenene kadar sıfır port içerir.

Standard Switch'teki "port group ↔ uplink" ayrımının merkezi karşılığı budur.

### Adım Adım: Distributed Port Group Oluşturma

vDS'e sağ tıklayıp **Distributed Port Group → New Distributed Port Group** yolunu izleyin.

#### 1. Ad ve Konum

Port group'a amacını yansıtan bir isim verin (`DPG-Production-VLAN10`, `DPG-DMZ` gibi). Konum, ait olacağı vDS'tir. İsimlendirme disiplini burada da geçerlidir: distributed port group'lar tüm host'larda ortak olduğu için, isim doğrudan operasyonel bir referanstır.

#### 2. Port Binding

Sihirbazın en teknik kararıdır ve bir VM'in port group'taki portu **ne zaman** tahsis edeceğini belirler:

* **Static binding (varsayılan ve önerilen):** Port, VM port group'a bağlandığı anda tahsis edilir ve VM kapalıyken bile ona ayrılmış kalır. Bu, port bazlı istatistiklerin ve politikaların VM ile birlikte kalıcı olmasını sağlar. Neredeyse tüm production senaryolarında doğru seçim budur.
* **Ephemeral (no binding):** Port, VM açıldığında oluşturulur, kapandığında yok olur. vCenter olmadan da host üzerinden yönetilebildiği için asıl değeri **kurtarma senaryosudur**: vCenter çevrimdışıyken bir VM'i (özellikle vCenter'ın kendisini) ağa bağlamanın yoludur. Buna karşılık kalıcı port geçmişi tutmaz ve ölçeklenebilirliği düşüktür — günlük kullanım için tercih edilmez, acil durum port group'u olarak bulundurulur.
* **Dynamic binding:** Eski sürümlerden kalma, artık kullanımdan kaldırılmış bir seçenektir; yeni tasarımlarda kullanılmamalıdır.

Pratik öneri: production port group'larını **static binding** ile oluşturun ve ortamda tek bir **ephemeral** port group'u kurtarma amaçlı hazır bulundurun. Bu ikinci nokta, önceki bölümde değindiğimiz "vCenter bağımlılığını yönetme" prensibinin somut uygulamasıdır.

#### 3. Port Allocation ve Port Sayısı

Port tahsis davranışını belirler:

* **Elastic (varsayılan):** Sekiz portla başlar ve ihtiyaç arttıkça otomatik olarak port ekler, azaldıkça geri alır. Kaç VM bağlanacağını önceden bilmediğiniz durumlarda — yani çoğu durumda — doğru seçimdir.
* **Fixed:** Port sayısını sabitler (varsayılan 128, değiştirilebilir). Bir port group'a bağlanabilecek VM sayısını bilinçli olarak sınırlamak istediğiniz özel senaryolarda anlamlıdır; sınır dolduğunda yeni VM bağlanamaz.

Elastic ile başlamak, "port yetmedi" sorununu kökten ortadan kaldırdığı için genel öneridir.

#### 4. VLAN Tipi

Standard Switch'te tek bir VLAN ID alanı vardı; vDS daha zengin seçenekler sunar:

* **None:** VLAN etiketleme yok
* **VLAN:** Tek bir VLAN ID (Standard Switch'teki VST karşılığı, en yaygın kullanım)
* **VLAN Trunking:** Birden fazla VLAN'ın etiketli olarak port group üzerinden geçmesine izin verir — sanal firewall/router appliance'ları gibi kendi etiketlemesini yapan iş yükleri için
* **Private VLAN:** vDS'e özgü ileri bir izolasyon mekanizması; aynı subnet içindeki VM'leri birbirinden yalıtmak için kullanılır (Standard Switch'te karşılığı yoktur)

Tipik VM ağları için **VLAN** tipini seçip ilgili ID'yi girmeniz yeterlidir. VST kullanıyorsanız fiziksel switch portlarının trunk olarak yapılandırılması gerektiğini hatırlayın.

#### 5. Customize Default Policies (İsteğe Bağlı)

Bu seçeneği işaretlerseniz sihirbaz, port group'un tüm politikalarını oluşturma anında tanımlamanıza izin verir: **security** (Promiscuous Mode, MAC Address Changes, Forged Transmits), **traffic shaping**, **teaming and failover**, **monitoring** ve **miscellaneous** (port blocking).

Bu adım, vDS'in merkezi yönetim avantajının en somut göründüğü yerdir: burada tanımladığınız politika, port group'a bağlı **tüm host'lardaki tüm VM'ler** için geçerli olur. Standard Switch'te aynı ayarı her host'ta ayrı ayrı yapmanız gerekiyordu.

Pratik yaklaşım: ilk oluşturmada varsayılanlarla ilerleyip politikaları sonradan düzenlemek de mümkündür; ancak güvenlik ayarlarını (üçü de **Reject**) baştan doğru kurmak, "sonra bakarım" ile unutulan istisnaların önüne geçer.

**Finish** ile port group oluşturulur ve envanterde diğerlerinin yanında yerini alır.

### Distributed Port Group Ne İşe Yarar?

Tanımı netleştirmek gerekirse: bir distributed port group, bir Distributed Switch'e bağlı olan ve **her üye portun yapılandırma seçeneklerini belirleyen** port topluluğudur. Bir bağlantının vDS üzerinden ağa nasıl yapılacağını tanımlar.

Buradaki kritik fark ölçektir. Standard Switch'te port group, tek bir host'un içindeki bir tanımdı. Distributed port group ise vDS'e katılan **tüm host'larda aynı anda** geçerlidir:

* Bir VM hangi host'ta çalışırsa çalışsın, aynı port group'a bağlıysa aynı VLAN'ı, aynı güvenlik politikalarını ve aynı teaming davranışını kullanır.
* Port group üzerinde yapılan bir değişiklik — VLAN değişimi, güvenlik politikası güncellemesi — tüm host'lara ve tüm bağlı VM'lere anında yansır.
* vMotion sırasında VM'in ağ tanımı kendisiyle birlikte gelir; hedef host'ta "aynı isimde port group var mı?" sorusu ortadan kalkar.

Oluşturduktan sonra port group'un kendi görünümünden bağlı host sayısını, VM sayısını ve port istatistiklerini izleyebilir; **Edit** ile port sayısını, politikaları ve VLAN yapılandırmasını sonradan değiştirebilirsiniz. Yeni oluşturulmuş bir port group'ta bu sayıların sıfır olması normaldir — henüz host eklenmemiş ve VM taşınmamıştır.

### Sonuç

Distributed port group, vDS mimarisinde sanal makinelerin ağa bağlandığı ve ağ politikalarının merkezden tanımlandığı temel yapı taşıdır. Özetle:

* Oluşturma sihirbazındaki dört karar önemlidir: **isim** (tüm host'larda ortak referans), **port binding** (production için static; kurtarma için bir adet ephemeral), **port allocation** (varsayılan elastic), **VLAN tipi** (tipik olarak VLAN; trunking ve private VLAN ileri senaryolar için).
* Politikaları oluşturma anında veya sonradan tanımlayabilirsiniz; hangisini seçerseniz seçin, güvenlik üçlüsünü **Reject** olarak kurmak temel duruş olmalıdır.
* Topolojide uplink slotlarının boş görünmesi, henüz host eklenmediğinin göstergesidir; port group'lar tanımlı olsa da switch trafik taşımaya hazır değildir.

Sıradaki adım bu tabloyu tamamlamak: host'ları vDS'e ekleyip fiziksel adaptörleri uplink slotlarına eşlemek ve ardından teaming/failover politikalarıyla trafiğin bu uplink'ler arasında nasıl dağılacağını belirlemek.
