---
icon: square-dribbble
---

# Creating a Private Network Port Group

Sanal ağ tasarımının şimdiye kadar hep bir hedefi vardı: sanal makineleri dış dünyaya, kullanıcılara ve diğer sistemlere **bağlamak**. Bu makalede ise tam tersini bilinçli olarak yapacağız: hiçbir fiziksel adaptöre çıkmayan, dışarıdan kimsenin erişemediği ve içindeki makinelerin yalnızca birbirleriyle haberleşebildiği **private (izole) bir ağ** kuracağız.

Daha önce uplink'siz bir switch'in kavramsal olarak ne anlama geldiğine değinmiştik; şimdi bu kavramı uçtan uca hayata geçiriyoruz: izole switch'in oluşturulması, private port group'un tanımlanması, sanal makinelerin bu ağa yerleştirilmesi ve — en az bunlar kadar önemlisi — bu tasarımın sınırlarının doğru anlaşılması.

### Tasarımın Mantığı: İzolasyon Nereden Gelir?

Bir sanal makinenin erişim alanını belirleyen zincir şudur: vNIC → port group → vSwitch → uplink. İzolasyonun sırrı bu zincirin son halkasını **bilerek eksik bırakmaktır**: uplink'i olmayan bir switch üzerindeki makineler, fiziksel ağa çıkacak hiçbir yola sahip değildir. Trafikleri host'un belleği içinde kalır; kabloya hiç dokunmaz.

Bu noktada port group'u **hangi switch'e** bağladığınız, tasarımın tamamını belirleyen karardır:

* Yeni port group'u **`vSwitch0`'a** bağlarsanız: içindeki makineler, aynı switch'teki diğer port group'larla (örneğin VM Network) aynı Layer 2 altyapısını paylaşır ve uplink üzerinden dış dünyaya çıkar. İzolasyon yoktur.
* **Uplink'li yeni switch'e** bağlarsanız: makineler o switch'in uplink'i üzerinden yine dış ağa ulaşır.
* **Uplink'siz switch'e** bağlarsanız: makineler yalnızca aynı izole switch üzerindeki diğer makinelerle haberleşebilir. Aradığımız private network budur.

Önemli bir teknik incelik: aynı switch üzerindeki port group'ların birbiriyle haberleşmesi, **aynı VLAN ID'ye sahip olmalarına** bağlıdır. Aynı switch'te bile farklı VLAN ID'li port group'lar birbirinden yalıtılmıştır. Yani izolasyonun iki aracı vardır — uplink'siz switch **fiziksel dünyadan**, VLAN ID ise **aynı switch içindeki diğer ağlardan** yalıtır. Private network tasarımında ilkini kullanıyoruz; ikincisi, kalabalık switch'lerde ek bir katman olarak akılda tutulmalıdır.

### Adım Adım: Private Network Kurulumu

#### 1. Uplink'siz switch'i oluşturun

**Networking → Virtual Switches → Add standard virtual switch** ile yeni bir switch oluşturun ve anlamlı bir isim verin: `vSwitch-Isolated` veya `LocalNetwork` gibi. Kritik nokta: **uplink adımında hiçbir fiziksel adaptör seçmeyin.** Boşta vmnic varsa sihirbaz otomatik önerecektir; öneriyi kaldırıp switch'i adaptörsüz oluşturun.

Switch topolojisine baktığınızda farkı hemen görürsünüz: switch'in fiziksel tarafa uzanan bağlantısı yoktur — dış dünyaya giden yol, tasarım gereği kesiktir.

#### 2. Private port group'u ekleyin

Uplink'siz de olsa kural değişmez: **sanal makine bağlamak için switch üzerinde bir port group bulunmak zorundadır.** **Add port group** ile yeni bir port group oluşturun:

* **Name:** `PG-Private` veya `PG-Isolated-Lab` gibi amacı belli bir isim
* **Virtual Switch:** Az önce oluşturduğunuz uplink'siz switch — yukarıda anlattığımız nedenle bu seçim işlemin en kritik adımıdır; yanlışlıkla `vSwitch0` seçilirse "izole" sandığınız makineler production ağına açılır
* **VLAN ID ve Security:** Varsayılan değerler (0 ve inherit) bu senaryo için yeterlidir

#### 3. Sanal makineleri private ağa yerleştirin

İki yol vardır ve ikisi de aynı sonuca ulaşır:

**Yeni VM oluştururken:** VM oluşturma sihirbazının network adapter adımında açılır listede artık üç seçenek görürsünüz — `VM Network`, daha önce oluşturduğunuz port group'lar ve `PG-Private`. Private'ı seçtiğinizde makine doğrudan izole ağa doğar.

**Mevcut bir VM'i taşırken:** Çalışan bir makineyi izole etmek için **Edit Settings → Network Adapter** üzerinden port group'u private ağa çevirip kaydetmeniz yeterlidir. İşlem anlıktır ve VM'i kapatmayı gerektirmez — bu "anahtar çevirme" hızının operasyonel değerine birazdan döneceğiz.

#### 4. İzolasyonu doğrulayın

Yapılandırma bittikten sonra sonucu iki düzeyde kontrol edin:

* **Topolojide:** Switch görünümünde private port group altındaki makineleri ve fiziksel adaptör bağlantısının bulunmadığını görürsünüz.
* **Ağ düzeyinde:** Private ağdaki bir makineden diğer ağlardaki herhangi bir makineye, gateway'e veya dış dünyaya ping atın — hiçbirine ulaşamamalıdır. Aynı private ağdaki iki makine ise (IP'leri aynı subnet'te olmak kaydıyla) birbirine sorunsuz erişmelidir. İzolasyon iddiası, her zaman testle kanıtlanmalıdır.

Bir pratik detay: bu ağda DHCP yoktur — dış dünyaya kapalı olduğu için ortamınızdaki DHCP sunucusuna da ulaşılamaz. Makinelere ya **static IP** verin ya da gerekiyorsa private ağın içine küçük bir DHCP servisi (bir VM olarak) siz kurun.

### Private Network Nerelerde Kullanılır?

Uplink'siz ağ, ilk bakışta bir lab merakı gibi görünse de production operasyonlarında somut karşılıkları olan bir araçtır:

* **Klon ve restore çakışmalarını önlemek:** Bir domain controller'ın veya production sunucusunun klonunu ya da backup restore'unu test etmeniz gerektiğinde, makineyi doğrudan production ağına açmak IP ve hostname çakışmasına, hatta dizin servislerinde ciddi tutarsızlıklara yol açar. Klonu private ağda açmak, orijinaliyle asla karşılaşmamasını garanti eder.
* **Şüpheli makineyi karantinaya almak:** Ele geçirildiğinden şüphelenilen bir VM'i kapatmak, bellekteki delilleri yok eder. Bunun yerine makinenin port group'unu private ağa çevirmek, ağ erişimini saniyeler içinde keserken makineyi analiz için canlı tutar — incident response ekiplerinin standart ilk hamlelerinden biridir.
* **Yama ve upgrade testleri:** Production kopyaları üzerinde riskli değişiklikleri, dış dünyayı hiç etkilemeden deneyebilirsiniz.
* **Zararlı yazılım analizi ve güvenlik lab'ları:** Analiz makinelerinin dışarıya sızıntı yapamayacağı doğal bir sandbox sağlar.
* **Çok katmanlı appliance mimarileri:** Çift vNIC'li bir firewall/router VM'in bir bacağı uplink'li switch'e, diğeri private switch'e bağlanarak, iç ağdaki tüm makinelerin dış dünyaya yalnızca bu denetim noktasından çıktığı bir topoloji kurulabilir.

### Sınırları Doğru Anlamak: Üç Kritik Uyarı

Private network güçlü bir araçtır; ancak ne olduğu kadar **ne olmadığını** da bilmek gerekir:

#### 1. İzolasyon host'a özgüdür

Uplink'siz bir Standard Switch, yalnızca **oluşturulduğu host'un içinde** var olur. Bunun iki önemli sonucu vardır: aynı private ağı iki farklı host'ta aynı isimle oluştursanız bile bu iki ağ birbirine bağlı **değildir** — birindeki makine diğerindekine ulaşamaz. Ve bu ağdaki bir VM başka host'a vMotion edilirse, private ağdaki komşularıyla bağlantısı kopar. Cluster ortamında bu makineler için **DRS kurallarıyla host sabitlemesi** yapmak veya izole makine grubunu tek host'ta tutmak gerekir. Host'lar arası uzanan izole ağlar gerekiyorsa, çözüm Standard Switch değil; vDS üzerinde private VLAN veya NSX gibi overlay teknolojileridir.

#### 2. Karantina, güvenliğin tamamı değildir

Private ağ dış dünyadan yalıtır; ancak **ağın içi filtrelenmemiş bir Layer 2 segmentidir**. İçerideki makineler birbirine tam erişime sahiptir — karantinaya aldığınız şüpheli bir makineyle analiz makinenizi aynı private ağa koyarsanız, şüpheli makine analiz makinesine saldırabilir. Hassas senaryolarda her şüpheli makineye **kendi ayrı private switch'ini** vermek en temiz yaklaşımdır; uplink gerektirmediği için bunun donanım maliyeti sıfırdır.

#### 3. Erişim yolu yalnızca konsoldur

Private ağdaki makinelere SSH/RDP ile ulaşamazsınız — ağ yolu yoktur. Tek erişim kapınız vSphere Client üzerindeki **VM konsoludur**. Bu, tasarımın hatası değil doğasıdır; ancak izole makinelerde yerel yönetici parolalarının bilinir ve güncel olmasını, konsol erişiminin yeterli olmasını önceden planlamayı gerektirir. Ağdan kestiğiniz makinenin parolasını bilmiyorsanız, karantina kararı sizi de dışarıda bırakır.

### Sonuç

Private network port group, sanal ağ araç kutusundaki en basit ama en keskin araçlardan biridir: uplink'i olmayan bir switch ve üzerinde tek bir port group — hepsi bu. Özetle:

* İzolasyon, zincirin son halkasını (uplink) bilinçli olarak boş bırakmaktan doğar; port group'un **doğru switch'e** bağlanması tasarımın en kritik adımıdır.
* Mevcut bir makineyi Edit Settings ile private ağa taşımak anlık ve kesintisizdir — klon testlerinden incident response karantinasına kadar geniş bir operasyonel kullanım alanı açar.
* Sınırlar nettir: izolasyon host'a özgüdür (vMotion senaryoları planlanmalıdır), ağın içi filtrelenmemiştir (gerekirse makine başına ayrı switch) ve tek erişim yolu konsoldur.

Bu makaleyle birlikte Standard Switch'in sunduğu tüm yapı taşlarını uçtan uca kurmuş olduk: uplink yönetimi, VM port group'ları, VMkernel portları ve izole ağlar. Farklı kombinasyonları — çoklu switch mimarileri, VLAN'lı tasarımlar, appliance zincirleri — kendi lab ortamınızda kurup test etmek, bu bilgiyi kalıcı hale getirmenin en etkili yoludur; sanal ağda ustalık, topolojiyi ezberlemekten değil, "bu kabloyu çekersem ne olur" sorusunu güvenle cevaplayabilmekten gelir.
