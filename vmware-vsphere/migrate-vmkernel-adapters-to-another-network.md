---
icon: manhole
---

# Migrate VMkernel Adapters to Another Network

Önceki bölümde sanal makinelerin ağ bağlantılarını Distributed Switch'e taşımıştık. Ancak bir host üzerindeki trafik yalnızca VM trafiğinden ibaret değildir: management, vMotion, vSAN, Fault Tolerance gibi hypervisor servisleri **VMkernel portları** üzerinden akar ve bu portlar hâlâ Standard Switch'lerde durmaktadır.

Bu makalede geçişin son halkasını tamamlıyoruz: VMkernel adaptörlerinin vDS'e nasıl taşınacağını, işlemin neden VM migrasyonundan daha dikkat gerektirdiğini ve tamamlanmış bir vDS topolojisinin nasıl okunacağını ele alıyoruz.

### Neden VMkernel Portlarını da Taşımalı?

Standard Switch mimarisinde her host'un VMkernel portları o host'a özgüydü; on host'ta vMotion ağını değiştirmek on ayrı düzenleme demekti. VMkernel portlarını vDS'e taşıdığınızda bu tablo değişir:

* Tüm host'ların vMotion, vSAN veya management trafiği **tek bir distributed port group** üzerinden yönetilir.
* VLAN, MTU, teaming ve failover politikaları bir kez tanımlanır, tüm host'lara uygulanır.
* Servis ağlarının host'lar arası tutarlılığı — vMotion ve vSAN'ın sağlıklı çalışması için kritik olan şey — mimarinin garantisi haline gelir.

Bu, vDS'e geçişin yalnızca VM trafiği için değil, altyapı trafiği için de anlamlı olmasının nedenidir.

### VM Migrasyonundan Farkı: Neden Daha Dikkatli Olmalı?

VMkernel taşıma işlemi teknik olarak VM migrasyonuna benzer, ancak risk profili farklıdır ve bunu baştan bilmek gerekir:

* Bir VM'in ağı yanlış taşınırsa, o VM ağa çıkamaz — sorun izole ve geri alınabilirdir.
* **Management vmk'sı** yanlış taşınırsa, host'un vCenter ile bağlantısı kopar. vCenter host'u yönetemez hale gelir ve kurtarma yalnızca DCUI/konsol üzerinden mümkün olur.
* **vSAN veya iSCSI vmk'sı** yanlış taşınırsa storage erişimi etkilenir; bu, üzerinde çalışan tüm VM'leri etkileyen çok daha geniş bir sorundur.

Bu nedenle pratik yaklaşım şudur: **önce riski düşük servis vmk'larını taşıyın** (vMotion, provisioning, replication), yapıyı doğrulayın, management ve storage vmk'larını en sona bırakın. Management vmk'sını taşırken konsol erişiminizin (iLO/iDRAC/IPMI) hazır olduğu bir bakım penceresi seçin.

### Adım Adım: VMkernel Adaptörü Taşıma

vDS'e sağ tıklayıp **Add and Manage Hosts** ile başlayın ve bu kez **Manage host networking** kısmını seçin — bu seçenek, vDS'e zaten eklenmiş host'ların yapılandırmasını düzenlemek içindir.

#### 1. Host seçimi

**Attached hosts** ile vDS'e ekli host'lardan işlem yapacaklarınızı seçin. Tek bir host üzerinde çalışabileceğiniz gibi birden fazlasını da seçebilirsiniz. Kademeli geçiş prensibi gereği, ilk taşımada tek host ile başlamak doğru yaklaşımdır.

#### 2. Görev seçimi

Sihirbaz üç kalemi listeler: **Manage physical adapters**, **Manage VMkernel adapters** ve **Migrate virtual machine networking**. Bu işlemde yalnızca **Manage VMkernel adapters** ile ilgileneceğiz; fiziksel adaptörler önceki adımlarda eşlenmiş, VM'ler önceki bölümde taşınmıştı.

Diğer kalemleri işaretsiz bırakmak, sihirbazın ilgili adımlarını atlamasını sağlar ve işlemi tek konuya odaklar. Bu, kademeli geçişin sihirbaz üzerindeki karşılığıdır: her seferinde tek bir değişiklik sınıfı.

#### 3. VMkernel adaptörlerini eşleme

Bu adımda seçili host'ların tüm vmk portları listelenir. Her satırda şu bilgiler yer alır: vmk adı, bağlı olduğu **kaynak port group** ve **hedef port group** (varsayılan olarak "Do not migrate").

Tipik bir tablo şöyle görünür:

* `vmk0` → kaynak: Management Network (vSwitch0) → hedef: Do not migrate
* `vmk1` → kaynak: PG-Management-2 (yeni switch) → hedef: Do not migrate
* `vmk2` → kaynak: PG-vSAN (yeni switch) → hedef: Do not migrate

Taşımak istediğiniz vmk'yı seçip **Assign port group** ile hedef distributed port group'u belirlersiniz. Seçmediğiniz vmk'lar "Do not migrate" durumunda kalır ve mevcut Standard Switch'lerinde çalışmaya devam eder — bu, tam olarak istediğiniz kademeli davranıştır.

Örneğin yalnızca vSAN vmk'sını taşımak istiyorsanız o satıra hedef port group atar, diğerlerine dokunmazsınız.

**Hedef port group seçiminde dikkat**

Buradaki kritik nokta: **her VMkernel servisi için ayrı bir distributed port group kullanın.** VM'lerin bağlı olduğu port group'a vmk atamak teknik olarak mümkündür, ancak izolasyon prensibini bozar — host'un yönetim veya storage trafiği, VM'lerle aynı Layer 2 segmentine iner.

Doğru yapı şöyledir:

* `DPG-VM-Production` → sanal makineler
* `DPG-vMotion` → vMotion vmk'ları (kendi VLAN'ında)
* `DPG-vSAN` → vSAN vmk'ları (kendi VLAN'ında)
* `DPG-Management` → management vmk'ları

İsimlendirmeyi amaca göre yapmak — hangi port group'un hangi trafiği taşıdığını isminden anlamak — çok host'lu ortamlarda operasyonel bir zorunluluktur.

#### 4. Etki analizi

Sihirbaz **Analyze impact** adımında değişikliğin mevcut servisleri etkileyip etkilemeyeceğini değerlendirir. Uyarı çıkarsa devam etmeden önce gözden geçirin; özellikle iSCSI port binding veya vSAN gibi servislerde bu analiz gerçek bir koruma sağlar.

#### 5. Tamamlama

Özet ekranında güncellenecek host'ları ve taşınacak vmk'ları doğrulayıp **Finish** ile işlemi tamamlarsınız. Taşıma anlıktır; vmk kendi IP'siyle birlikte yeni port group'a geçer.

### Sonucu Doğrulama

Taşıma sonrası vDS'in **Configure → Topology** görünümü tabloyu bütün olarak gösterir. Tamamlanmış bir geçişte tipik görünüm şöyledir:

**Sol taraf (virtual):**

* VMkernel için ayrılmış distributed port group: içinde taşınan vmk, kendi IP'siyle
* VM port group'ları: içlerinde önceki adımda taşınan sanal makineler

**Sağ taraf (physical):**

* Uplink slotları ve her birinin altında eşlenmiş fiziksel adaptörler

Uplink tarafını okurken bir noktaya dikkat edin: **bir uplink slotu altında birden fazla adaptör görünmesi normaldir** — çünkü her host kendi vmnic'ini aynı slota eşlemiştir. Örneğin `Uplink 1` altında Host-A/`vmnic2` ve Host-B/`vmnic1` birlikte görünür. Bu, slotların soyut yer tutucular olmasının doğal sonucudur: aynı slot, her host'ta o host'un ilgili fiziksel adaptörünü temsil eder.

Doğrulamayı topolojiyle sınırlı bırakmayın; taşınan servisin gerçekten çalıştığını test edin:

```bash
# vmk'nın hangi IP ve port group ile çalıştığını doğrula
esxcli network ip interface list

# İlgili arayüz üzerinden bağlantı testi
vmkping -I vmk2 <hedef-ip>

# Jumbo Frames kullanılıyorsa MTU testi
vmkping -I vmk2 -s 8972 -d <hedef-ip>
```

vMotion taşındıysa iki host arasında bir test migration'ı çalıştırın; vSAN taşındıysa cluster sağlık durumunu kontrol edin. Topolojide doğru görünen bir yapılandırma, fiziksel switch tarafındaki bir VLAN eksikliği nedeniyle pekâlâ çalışmıyor olabilir.

### Geçiş Tamamlandığında Elinizde Ne Var?

Bu adımla birlikte üç bileşenin tamamı vDS'e taşınmış olur:

1. **Fiziksel adaptörler** → uplink slotlarına eşlendi
2. **Sanal makineler** → distributed port group'lara taşındı
3. **VMkernel portları** → servis bazlı distributed port group'lara taşındı

Artık ortamınızdaki ağ yapılandırmasının tamamı tek merkezden yönetiliyor. Bir güvenlik politikasını, VLAN'ı, teaming davranışını veya traffic shaping ayarını değiştirmek istediğinizde, ilgili distributed port group üzerinde **tek bir düzenleme** yapmanız yeterlidir; değişiklik o port group'a bağlı tüm host'lardaki tüm VM ve VMkernel portlarına anında uygulanır.

Yirmi host'lu bir ortamda Standard Switch ile yirmi ayrı düzenleme ve ardından yirmi ayrı doğrulama gerektiren iş, tek bir işleme iner. Serinin başından beri izlediğimiz hikâyenin vardığı nokta budur.

#### Geçiş sonrası temizlik

Yapı doğrulandıktan sonra — ancak doğrulama tamamlanmadan önce değil — Standard Switch tarafında kalan artıkları değerlendirin:

* Boşalan Standard Switch'leri ve port group'ları kaldırmak envanteri temiz tutar.
* Buna karşılık **host başına bir Standard Switch'i management için bırakmak**, yaygın ve savunulabilir bir tasarım tercihidir: vCenter erişilemez durumdayken host'a bağlanmanın vDS'e bağımlı olmayan bir yolu kalır. Yalnızca-vDS mimarisi tercih edilecekse, kurtarma amaçlı **ephemeral binding'li** bir port group hazır bulundurulmalıdır.
* Kaynak Standard Switch yapılandırmasını (port group adları, VLAN'lar, uplink eşlemeleri) belgelemeden silmeyin; geri dönüş gerekirse bu kayıt işinizi görür.

### Sonuç

VMkernel migrasyonu, Standard Switch'ten Distributed Switch'e geçişi tamamlayan adımdır. Özetle:

* vmk taşıma, VM taşımadan **daha yüksek riskli** bir işlemdir: management vmk'sı host'un vCenter bağlantısını, storage vmk'sı ise tüm VM'lerin disk erişimini etkileyebilir.
* Doğru sıra riski düşükten yükseğe doğrudur: önce vMotion/provisioning, en son management ve storage. Konsol erişimi hazır tutulmalıdır.
* Sihirbazda yalnızca taşınacak vmk'lara hedef port group atanır; diğerleri "Do not migrate" ile yerinde bırakılır — bu, kademeli geçişin doğal aracıdır.
* Her servis için **ayrı distributed port group** kullanın; vmk'ları VM port group'larıyla aynı çatı altında toplamayın.
* Doğrulamayı topolojiyle bitirmeyin: `vmkping` ile arayüz bazlı test yapın, taşınan servisi (vMotion, vSAN) gerçekten çalıştırarak kontrol edin.

Geçiş tamamlandı; ağın tamamı artık tek merkezden yönetiliyor. Serinin devamında bu merkezi yapının asıl değerini ortaya çıkaran ileri yeteneklere geçeceğiz: teaming ve failover politikalarının ince ayarı, Network I/O Control ile bant genişliği yönetimi ve port mirroring gibi izleme araçları.
