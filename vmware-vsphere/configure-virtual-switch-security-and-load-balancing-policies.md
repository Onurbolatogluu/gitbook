---
icon: gear-complex
---

# Configure Virtual Switch Security and Load-Balancing Policies

Sanallaştırma ortamlarında ağ altyapısının kalbi, fiziksel switch'lerin yazılımsal karşılığı olan **Virtual Switch (vSwitch)** yapılarıdır. VMware vSphere ortamında her ESXi host, varsayılan olarak bir **Standard Switch (vSS)** ile gelir ve bu switch hem sanal makinelerin dış dünyayla iletişimini hem de host'un yönetim trafiğini taşır. Ancak varsayılan kurulumla yetinmek, hem erişilebilirlik hem de güvenlik açısından ciddi riskler barındırır.

Bu makalede bir Standard Switch'in temel bileşenlerini inceleyecek, ardından production ortamlarında kritik önem taşıyan **Security Policy** ve **Load-Balancing / NIC Teaming** yapılandırmalarını en iyi pratiklerle birlikte ele alacağız.

### Standard Switch Mimarisine Genel Bakış

Bir Standard Switch, ESXi host üzerinde çalışan ve sanal makineleri fiziksel ağa bağlayan Layer 2 bir yapıdır. Fiziksel bir switch satın aldığınızda ürünün port sayısı, MTU değeri, discovery protokolleri gibi özelliklerini nasıl inceliyorsanız, vSwitch üzerinde de aynı bilgilere erişebilir ve bunları düzenleyebilirsiniz. Öne çıkan bileşenler şunlardır:

* **Uplink (Physical Adapter):** vSwitch'i fiziksel ağa bağlayan fiziksel NIC (vmnic). Bir vSwitch'in dış dünyaya çıkışı tamamen bu adaptörler üzerinden gerçekleşir.
* **Port Group:** Sanal makinelerin bağlandığı mantıksal port toplulukları. VLAN etiketleme, güvenlik ve teaming politikaları port group seviyesinde override edilebilir.
* **VMkernel Port:** Host'un kendi trafiği (management, vMotion, vSAN, Fault Tolerance, provisioning, replication vb.) için kullanılan özel port tipi.
* **MTU:** Varsayılan değer 1500 byte'tır ve çoğu senaryo için yeterlidir. iSCSI, NFS veya vSAN gibi storage trafiği taşıyan ortamlarda 9000 byte (Jumbo Frames) tercih edilebilir; ancak bu durumda uçtan uca tüm fiziksel ağ bileşenlerinin de aynı MTU değerini desteklediğinden emin olunmalıdır. Uçtan uca tutarlılık sağlanamıyorsa varsayılan değerde kalmak en güvenli yaklaşımdır.
* **Port Sayısı:** Standard Switch üzerinde 1500'ün üzerinde kullanılabilir port bulunur (örneğin 1513 gibi bir değer görürsünüz). Modern ESXi sürümlerinde portlar elastic olarak yönetildiği için pratikte port tükenmesi bir sorun teşkil etmez.
* **Link Discovery (CDP):** Standard Switch, **Cisco Discovery Protocol (CDP)** destekler. Bu sayede vSwitch, tıpkı fiziksel bir Cisco switch gibi komşu cihazlarla topoloji bilgisi paylaşabilir. (Not: **LLDP** desteği yalnızca Distributed Switch üzerinde mevcuttur; vendor bağımsız bir ortamda çalışıyorsanız bu ayrım tasarım kararınızı etkileyebilir.)

### Uplink Redundancy: En Sık Görülen Uyarının Anlamı

Tek bir fiziksel adaptöre bağlı bir vSwitch üzerinde şu uyarıyı görürsünüz:

> _"This virtual switch has no uplink redundancy. You should add another uplink adapter."_

Bu uyarının anlamı nettir: vSwitch'in fiziksel ağa çıkışı **tek bir NIC'e** bağlıdır ve bu NIC'te (ya da bağlı olduğu kabloda, portta veya fiziksel switch'te) bir arıza yaşanırsa, o vSwitch üzerindeki tüm sanal makineler ve VMkernel servisleri ağ bağlantısını kaybeder. Management trafiği de aynı switch üzerindeyse **host'a erişiminizi tamamen yitirirsiniz.**

#### Redundancy için en iyi pratikler

* Her vSwitch'e **en az iki fiziksel uplink** atayın. Modern sunucular zaten iki ve üzeri NIC ile geldiği için bu genellikle ek maliyet gerektirmez.
* İki uplink'i **farklı fiziksel switch'lere** bağlayın. Aynı fiziksel switch'e bağlı iki NIC, switch arızasında sizi korumaz; gerçek redundancy uçtan uca düşünülmelidir.
* Mümkünse NIC'leri sunucu üzerinde **farklı fiziksel kartlara** dağıtın (örneğin biri onboard, biri PCIe kart üzerinde). Böylece tek bir kart arızası tüm uplink'leri düşürmez.
* Kritik trafiği ayrıştırın: Management ve vMotion trafiğini VM trafiğinden ayrı port group'larda (ideal olarak ayrı VLAN'larda) taşıyın.

### VMkernel ve Management Network

Her ESXi host üzerinde **en az bir VMkernel portu management servisi etkin olarak** bulunmak zorundadır. Bu port, sizin vSphere Client veya SSH üzerinden host'a bağlanmanızı sağlayan arayüzdür. Management özelliği tamamen devre dışı bırakılmış bir host'a ağ üzerinden erişmek mümkün olmadığından, sistem bu senaryoya izin vermez; bu VMkernel portunu silmek, host ile aranızdaki tüm bağlantıyı koparmak anlamına gelir (kurtarma yalnızca DCUI/konsol üzerinden mümkün olur).

VMkernel portu üzerinde etkinleştirilebilecek servisler şunlardır:

* **Management:** Host yönetim trafiği (zorunlu, en az bir portta etkin olmalı)
* **vMotion:** Sanal makinelerin canlı taşınması
* **Provisioning:** Cold migration, cloning ve snapshot trafiği
* **Fault Tolerance Logging:** FT korumalı VM'lerin senkronizasyon trafiği
* **vSphere Replication / Replication NFC:** Replikasyon trafiği
* **vSAN:** vSAN storage trafiği

**Best practice:** Her servisi aynı VMkernel portuna yığmak yerine, her kritik servis için ayrı VMkernel portu ve ayrı VLAN kullanın. Özellikle vMotion trafiği şifrelenmemiş bellek içeriği taşıyabildiği için mutlaka izole bir ağda tutulmalıdır. IP yapılandırmasında DHCP yerine **static IP** kullanmak, management erişiminin DHCP altyapısına bağımlı kalmasını engeller ve production ortamlarında standart kabul edilir.

### Virtual Switch Security Policy'leri

vSwitch ve port group seviyesinde üç temel güvenlik politikası bulunur. Bu politikalar Layer 2 seviyesinde çalışır ve sanal makinelerin ağ üzerindeki davranışlarını sınırlar.

#### 1. Promiscuous Mode

**Ne yapar:** Etkinleştirildiğinde, port group'a bağlı bir sanal makine, yalnızca kendisine yönlendirilen trafiği değil, o vSwitch (veya VLAN) üzerindeki **tüm trafiği** görebilir.

**Önerilen değer: Reject.** Promiscuous Mode'un açık olması, ele geçirilmiş bir sanal makinenin ağdaki diğer VM'lerin trafiğini dinleyebilmesi (packet sniffing) anlamına gelir. Yalnızca IDS/IPS sensörleri, ağ monitörleme araçları veya nested virtualization gibi meşru ihtiyaçlar için, **yalnızca ilgili port group'ta** etkinleştirilmelidir; asla switch genelinde açılmamalıdır.

#### 2. MAC Address Changes

**Ne yapar:** Sanal makinenin guest OS'i, .vmx dosyasında tanımlı MAC adresinden farklı bir **effective MAC adresi** tanımlamaya çalıştığında gelen trafiğin kabul edilip edilmeyeceğini belirler.

**Önerilen değer: Reject.** MAC adresi değişikliğine izin vermek, MAC spoofing saldırılarının önünü açar: kötü niyetli bir VM, başka bir makinenin kimliğine bürünerek ona yönelik trafiği üzerine çekebilir. İstisnalar: Microsoft NLB (unicast mode), bazı cluster çözümleri ve iSCSI failover senaryoları gibi meşru MAC değişikliği gerektiren yapılandırmalar. Bu durumlarda politika yalnızca ilgili port group'ta Accept yapılmalıdır.

#### 3. Forged Transmits

**Ne yapar:** MAC Address Changes politikasının çıkış (outbound) trafiği için karşılığıdır. Sanal makine, kaynak MAC adresi kendi tanımlı MAC'inden farklı olan frame'ler göndermeye çalıştığında bu trafiğin iletilip iletilmeyeceğini kontrol eder.

**Önerilen değer: Reject.** Forged Transmits'in açık olması, VM'lerin sahte kaynak MAC adresiyle paket göndermesine ve dolayısıyla kimlik sahteciliği tabanlı saldırılara olanak tanır. Nested ESXi laboratuvarları ve bazı NLB senaryoları bu politikanın Accept olmasını gerektirir; yine kural aynıdır — istisna, yalnızca ihtiyaç duyulan port group ile sınırlı tutulur.

> **Not:** Standard Switch'te bu üç politikanın varsayılan değerleri sürüme göre farklılık gösterebilir (eski sürümlerde MAC Address Changes ve Forged Transmits varsayılan olarak Accept gelirdi). Distributed Switch'te ise üçü de varsayılan olarak Reject'tir. Ortamınızdaki mevcut değerleri denetlemek, security hardening çalışmasının ilk adımı olmalıdır. VMware'in **vSphere Security Configuration Guide (Hardening Guide)** dokümanı bu denetim için referans alınabilir.

### NIC Teaming ve Load-Balancing Politikaları

Birden fazla uplink eklediğinizde asıl soru şudur: trafik bu uplink'ler arasında nasıl dağıtılacak? Bu, **Teaming and Failover** ayarları altındaki load-balancing politikasıyla belirlenir.

#### Route Based on Originating Virtual Port ID (Varsayılan)

Her sanal makine (daha doğrusu her virtual port), vSwitch'e bağlandığı anda uplink'lerden birine atanır ve trafiği hep o uplink üzerinden akar.

* **Avantajları:** Fiziksel switch tarafında hiçbir özel yapılandırma gerektirmez, overhead'i düşüktür, öngörülebilirdir.
* **Sınırları:** Tek bir VM asla tek bir uplink'in bant genişliğinden fazlasını kullanamaz. Dağılım VM sayısına göredir, gerçek trafik yüküne göre değildir.
* **Ne zaman kullanılmalı:** Çoğu ortam için doğru ve güvenli varsayılan tercihtir.

#### Route Based on Source MAC Hash

Uplink seçimi, kaynak MAC adresinden üretilen hash ile yapılır. Davranış olarak Port ID yöntemine çok benzer; pratikte onu tercih etmek için özel bir neden nadiren vardır. Tek VM'de birden fazla vNIC olan bazı senaryolarda farklı dağılım sağlayabilir.

#### Route Based on IP Hash

Uplink seçimi, kaynak ve hedef IP adreslerinin hash'i ile yapılır. Böylece **tek bir VM**, farklı hedeflerle konuşurken farklı uplink'leri kullanabilir ve toplamda tek NIC bant genişliğinin üzerine çıkabilir.

* **Kritik gereksinim:** Fiziksel switch tarafında **static EtherChannel / Port Channel (LACP değil, static mode)** yapılandırması zorunludur. Bu yapılandırma olmadan IP Hash kullanmak, MAC flapping ve paket kaybına yol açar.
* **Dezavantajları:** Fiziksel ağ ekibiyle koordinasyon gerektirir, tüm uplink'lerin aynı fiziksel switch'e (veya stack/vPC yapısına) bağlı olmasını zorunlu kılar, troubleshooting'i karmaşıklaştırır.
* **Ne zaman kullanılmalı:** Tek bir VM'in çok sayıda farklı hedefe yüksek hacimli trafik ürettiği ve agregasyon ihtiyacının kanıtlandığı özel durumlarda.

#### Use Explicit Failover Order

Load balancing yapılmaz; trafik her zaman **Active Adapters** listesindeki ilk sağlıklı uplink'ten akar, arıza durumunda **Standby Adapters** devreye girer. Management ve vMotion gibi trafiklerin hangi NIC'ten akacağını deterministik olarak kontrol etmek istediğiniz tasarımlarda (örneğin iki VMkernel portunun aynı iki uplink'i çapraz active/standby kullandığı klasik tasarım) tercih edilir.

#### Route Based on Physical NIC Load (LBT)

Gerçek uplink kullanımını izleyip (%75 doluluk eşiği) port atamalarını dinamik olarak taşıyan tek politikadır ve fiziksel switch tarafında yapılandırma gerektirmez. **Yalnızca vSphere Distributed Switch (vDS) üzerinde kullanılabilir.** Enterprise Plus lisansınız ve vDS'iniz varsa, çoğu senaryoda en dengeli sonuç veren politika budur.

### Failover Davranışını Belirleyen Ek Ayarlar

Load-balancing politikasının yanında, failover'ın nasıl algılanıp yönetileceğini belirleyen dört ayar daha vardır:

* **Network Failure Detection:** Varsayılan yöntem **Link Status Only**'dir; yalnızca fiziksel link'in düşmesini algılar. **Beacon Probing** ise uplink'ler arasında probe paketleri göndererek upstream arızaları da yakalayabilir, ancak sağlıklı çalışması için en az üç uplink gerektirir. İki uplink'li standart yapılarda Link Status Only kullanın ve upstream arızalara karşı fiziksel switch tarafında Link State Tracking benzeri özelliklerden faydalanın.
* **Notify Switches:** **Yes** olarak kalmalıdır. Bir failover gerçekleştiğinde vSwitch, fiziksel switch'lere RARP paketleri göndererek MAC tablolarının anında güncellenmesini sağlar; bu, kesinti süresini saniyeler mertebesinden milisaniyelere indirir. (İstisna: unicast mode Microsoft NLB kullanan port group'larda No yapılması gerekir.)
* **Failback:** Arızalanan uplink tekrar ayağa kalktığında trafiğin otomatik olarak ona geri dönüp dönmeyeceğini belirler. Varsayılan **Yes**'tir; ancak port flapping yaşayan ortamlarda trafiğin sürekli gidip gelmesine neden olabilir. Management network gibi hassas trafiklerde **No** olarak ayarlamak stabiliteyi artırır.
* **Override at Port Group Level:** Tüm teaming ve security politikaları switch seviyesinde tanımlanır ancak port group seviyesinde override edilebilir. Doğru tasarım deseni şudur: switch seviyesinde en kısıtlayıcı/güvenli değerleri tanımlayın, istisnaları yalnızca ihtiyaç duyan port group'larda açın.

### Sonuç

Standard Switch, ilk bakışta "kur ve unut" bir bileşen gibi görünse de, production ortamının hem erişilebilirliğini hem de güvenliğini doğrudan belirleyen kritik bir katmandır. Özetlemek gerekirse:

* Her vSwitch'e **en az iki uplink** atayın ve bunları farklı fiziksel switch'lere bağlayın; uplink redundancy uyarısını asla görmezden gelmeyin.
* Management için her zaman en az bir VMkernel portu bulundurun, kritik servisleri ayrı VMkernel portları ve VLAN'larla izole edin, static IP kullanın.
* Üç güvenlik politikasını (**Promiscuous Mode, MAC Address Changes, Forged Transmits**) varsayılan olarak **Reject** yapın; istisnaları port group seviyesinde ve belgeleyerek açın.
* Load-balancing tercihinizde varsayılan **Port ID** politikası çoğu ortam için yeterlidir; **IP Hash**'i yalnızca fiziksel tarafta static EtherChannel ile, **LBT**'yi ise vDS kullanıyorsanız tercih edin.
* **Notify Switches: Yes** ayarını koruyun, flapping riski olan ortamlarda **Failback: No** değerlendirin.

Bu prensipleri uyguladığınızda, tek bir kablo arızasının tüm host'u ağdan düşürdüğü ya da tek bir ele geçirilmiş VM'in komşularının trafiğini dinleyebildiği senaryoların önüne geçmiş olursunuz. Sanal ağ tasarımında güvenlik ve erişilebilirlik, sonradan eklenen özellikler değil; ilk günden kurgulanması gereken temel gereksinimlerdir.
