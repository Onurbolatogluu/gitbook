---
icon: chart-network
---

# Overview of vNetwork Distributed Switch concepts

Serinin bu noktasına kadar Standard Switch'i (vSS) her yönüyle ele aldık: uplink yönetimi, port group'lar, VMkernel portları, izole ağlar ve hem host hem vCenter üzerinden yönetimi. Bir önceki bölümde bu yolculuğun vardığı sınırı da net biçimde tanımladık: Standard Switch **host-bazlı** bir nesnedir ve her host'ta ayrı ayrı yapılandırılır. Şimdi bu sınırın mimari cevabıyla tanışıyoruz: **vSphere Distributed Switch (vDS)**.

Bu makale, vDS'in ne olduğunu ve Standard Switch'ten temel farkının nereden kaynaklandığını kavramsal düzeyde ortaya koyar. Yapılandırma adımlarına geçmeden önce, "neden vDS?" sorusunun cevabını sağlam bir zemine oturtmak amacındayız.

### Temel Fark: Switch Hangi Katmanda Yaşıyor?

İki switch tipi arasındaki tüm farklılıklar tek bir yapısal ayrımdan doğar: **switch nesnesinin envanterde hangi katmanda tanımlandığı.**

#### Standard Switch: Host Katmanında

Bir Standard Switch, oluşturulduğu **host'un içinde** var olur ve yalnızca o host tarafından bilinir. Bir cluster'da on host varsa ve hepsinde aynı ağ yapısını istiyorsanız, on ayrı switch'i on kez, tek tek yapılandırmanız gerekir. Her host kendi switch'inin, kendi port group'larının ve kendi politikalarının sahibidir; bu nesneler host'lar arasında hiçbir şekilde ilişkili değildir. Aynı isimde iki port group oluştursanız bile bunlar iki ayrı, bağımsız tanımdır.

#### Distributed Switch: vCenter Katmanında

Bir vDS ise **vCenter Server katmanında** tanımlanır ve buradan cluster'daki host'lara dağıtılır. Switch'in "beyni" — tüm port group tanımları, VLAN'lar, güvenlik ve teaming politikaları — vCenter'da tek bir yerde tutulur; her host yalnızca bu merkezi tanımın kendi üzerindeki uzantısını çalıştırır.

Bunun doğrudan sonucu, vDS'in en çok bilinen ön koşuludur: **vCenter olmadan vDS oluşturulamaz.** Standard Switch host'un kendi başına yaşayabildiği bir yapıyken, vDS'in var olması için onu tanımlayan ve dağıtan bir merkez zorunludur. vCenter devre dışı kaldığında mevcut vDS trafiği akmaya devam eder (bu kritik bir tasarım detayıdır; buna aşağıda döneceğiz), ancak yeni yapılandırma yapılamaz.

### vDS'in Çözdüğü Asıl Problem: Tekrar ve Tutarsızlık

Standard Switch ile çalışırken bir güvenlik politikasını değiştirmek istediğinizde — örneğin tüm switch'lerde Forged Transmits'i Reject yapmak — her host'a gider, her switch'i açar, değişikliği tek tek uygularsınız. On host, dört switch demek kırk ayrı düzenleme demektir. Bu yaklaşımın iki maliyeti vardır:

* **Zaman:** İşlem host ve switch sayısıyla doğru orantılı olarak büyür.
* **Tutarsızlık riski:** Kırk düzenlemenin birinde yapılan hata veya atlanan bir adım, o host'u diğerlerinden farklı bir yapılandırmada bırakır. Bu tür sapmalar (configuration drift) elle yönetimde kaçınılmazdır ve genellikle ancak bir vMotion başarısız olduğunda ya da bir failover beklendiği gibi çalışmadığında fark edilir.

vDS bu problemi kökten çözer: **politikayı bir kez tanımlarsınız, tüm host'lar otomatik olarak uygular.** Bir distributed port group üzerinde VLAN'ı değiştirdiğinizde, o port group'a bağlı tüm host'lardaki tüm VM'ler anında yeni yapılandırmayı devralır. Tutarlılık, elle korunması gereken bir hedef olmaktan çıkıp mimarinin tasarım garantisi haline gelir.

### Kavramsal Yapı: vDS Nasıl Kurgulanır?

vDS, Standard Switch'teki kavramların merkezi karşılıklarıyla çalışır. Terminoloji farkını baştan oturtmak, ileriki yapılandırmayı kolaylaştırır:

* **vSphere Distributed Switch:** vCenter'da tanımlı merkezi switch nesnesi. Bir cluster'a birden fazla vDS eklenebilir (örneğin biri VM trafiği, biri storage/vMotion için).
* **Distributed Port Group:** Standard Switch'teki port group'un merkezi karşılığıdır. Bir kez tanımlanır ve vDS'e bağlı tüm host'larda geçerli olur. VM'ler ve VMkernel portları bu port group'lara bağlanır.
* **Uplink / Uplink Port Group:** vDS, fiziksel adaptörleri doğrudan değil, soyut **uplink** tanımları üzerinden yönetir (`Uplink 1`, `Uplink 2` gibi). Her host, kendi fiziksel vmnic'lerini bu soyut uplink'lere eşler. Böylece "Uplink 1 her host'ta o host'un ilk fiziksel adaptörüdür" gibi tutarlı bir soyutlama kurulur; politikalar fiziksel port isimlerine değil, bu soyut uplink'lere göre tanımlanır.
* **Proxy Switch (Hidden vSS):** Her host üzerinde, vDS'in o host'taki yerel uzantısı olarak çalışan gizli bir bileşen bulunur. Merkezi tanımı yerelde uygulayan ve vCenter çevrimdışıyken bile trafiğin akmaya devam etmesini sağlayan katman budur.

Bu yapıda "control plane" (yapılandırmanın tanımlandığı yer) vCenter'da, "data plane" (trafiğin fiilen aktığı yer) ise her host'un kendisindedir. Bu ayrım, vDS'in hem merkezi yönetim hem de host bağımsızlığı sağlamasının teknik temelidir.

### Yönetim Kolaylığının Ötesinde: vDS'e Özgü Yetenekler

vDS'i yalnızca "merkezi yönetilen Standard Switch" olarak görmek eksik bir tablodur. Merkezi kontrol düzlemi, Standard Switch'te mümkün olmayan bir dizi ileri yeteneğin de kapısını açar:

* **Network I/O Control (NIOC):** Trafik türlerine (vMotion, VM, storage) bant genişliği payı ve öncelik tanımlayarak, konsolide uplink'lerde bir trafiğin diğerlerini boğmasını engeller.
* **Load-Based Teaming (LBT):** Uplink'lerin gerçek kullanımını izleyip portları dinamik olarak dengeleyen, yalnızca vDS'te bulunan load-balancing politikası.
* **LACP desteği:** Fiziksel switch ile dinamik link aggregation (Standard Switch yalnızca statik EtherChannel destekler).
* **LLDP:** CDP'ye ek olarak vendor-bağımsız komşu keşif protokolü.
* **Port mirroring ve NetFlow:** Trafik analizi ve izleme için kurumsal seviye araçlar.
* **Network vMotion:** Bir VM host'lar arasında taşındığında ağ port istatistiklerinin ve durumunun korunması.
* **Ephemeral / static port binding seçenekleri:** Port tahsis davranışı üzerinde ince kontrol.

Bu yetenekler, vDS'i büyük ve trafik-yoğun ortamlarda yalnızca kolaylık değil, çoğu zaman bir gereklilik haline getirir.

### Dikkat Edilmesi Gereken Noktalar

vDS güçlü bir mimaridir, ancak birkaç pratik gerçeği baştan bilmek gerekir:

* **Lisans:** vDS, **vSphere Enterprise Plus** (veya onu içeren paketler) gerektirir. Standard Switch tüm sürümlerde bulunurken vDS lisansa bağlıdır; ortamınızın lisans seviyesi bu kararı doğrudan etkiler.
* **vCenter bağımlılığı:** vDS'in kontrol düzlemi vCenter'dır. Veri düzlemi host'ta yaşadığı için vCenter çökse bile mevcut VM trafiği kesilmez; ancak yeni port group oluşturmak veya politika değiştirmek vCenter geri gelene kadar mümkün olmaz. Bu nedenle **vCenter'ın kendisinin yönetim ağı, tercihen bir Standard Switch üzerinde tutulur** — vCenter'ı ayağa kaldıran ağın, vCenter'a bağımlı olmaması sağlıklı tasarımdır. (Yalnızca-vDS ortamlarında bu risk, ephemeral binding'li bir kurtarma port group'u ile yönetilir.)
* **Geçiş dikkat ister:** Mevcut bir ortamı Standard Switch'ten vDS'e taşımak, management vmk'sının migrasyonu gibi hassas adımlar içerir; yanlış sırayla yapılırsa host'un vCenter bağlantısı kopabilir. Bu nedenle geçiş, planlı ve tercihen kademeli (uplink'leri teker teker taşıyarak) yapılır.

### Sonuç

vSphere Distributed Switch, Standard Switch'in host-bazlı doğasının yarattığı tekrar ve tutarsızlık problemine verilen mimari cevaptır. Özetle:

* Tek yapısal fark — switch'in **vCenter katmanında** tanımlanması — hem en büyük avantajı (tek noktadan tanımlanıp tüm host'lara dağıtılan tutarlı yapılandırma) hem de en temel ön koşulu (vCenter olmadan var olamaması) doğurur.
* vDS, kırk ayrı düzenlemeyi tek bir tanıma indirger; tutarlılığı elle korunacak bir hedef olmaktan çıkarıp tasarım garantisine dönüştürür.
* Merkezi kontrol düzlemi, NIOC, LBT, LACP, port mirroring gibi Standard Switch'te bulunmayan kurumsal yetenekleri de beraberinde getirir.
* Bedeli, Enterprise Plus lisansı ve vCenter'a olan yapısal bağımlılığın bilinçli yönetilmesidir.

Kavramsal zemini kurduğumuza göre, serinin devamında somut adımlara geçebiliriz: bir Distributed Switch'in vCenter üzerinde oluşturulması, host'ların ve fiziksel uplink'lerin ona eklenmesi, distributed port group'ların tanımlanması ve VM'ler ile VMkernel trafiğinin yeni switch'e taşınması.
