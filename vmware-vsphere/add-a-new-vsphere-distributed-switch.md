---
icon: sliders-up
---

# Add a New vSphere Distributed Switch

Bir önceki bölümde vSphere Distributed Switch'in (vDS) kavramsal temelini kurmuştuk: Standard Switch host katmanında yaşarken, vDS vCenter katmanında — daha somut olarak **datacenter** seviyesinde — tanımlanır ve buradan host'lara dağıtılır. Bu makalede kavramı uygulamaya döküyoruz: sıfırdan bir Distributed Switch oluşturmanın adımlarını, sihirbazdaki her kararın ne anlama geldiğini ve doğru yapılandırmanın gerekçelerini ele alıyoruz.

### Neden Host Değil, Datacenter Seviyesi?

Standard Switch işlemlerini host'a bağlanarak yapıyorduk; host seviyesinde yalnızca Standard Switch görür ve oluşturabilirsiniz. Distributed Switch ise bu seviyede **hiç görünmez** — çünkü ait olduğu katman farklıdır.

vDS oluşturmak için vCenter envanterinde **datacenter** nesnesine çıkmanız gerekir. Datacenter'ın **Networking** görünümü, o datacenter'daki tüm ağ nesnelerini bir arada listeler:

* Mevcut Standard Switch kaynaklı ağlar (host'lardaki VM Network, private network'ler, önceki bölümlerde oluşturduğumuz port group'lar burada görünür)
* **Distributed Switch** kategorisi — henüz vDS oluşturmadıysanız boştur
* **Distributed Port Group** ve **Uplink Port Group** kategorileri — vDS ile birlikte dolacaktır
* Ağ nesnelerini düzenlemek için kullanılan **network folder** yapısı

Bu görünüm, vDS'in neden datacenter seviyesinde durduğunu da açıklar: switch, tek bir host'a değil, o datacenter'daki birden fazla host'a birden hizmet verecek merkezi bir nesnedir.

### Adım Adım: Distributed Switch Oluşturma

Datacenter'a sağ tıklayıp **Distributed Switch → New Distributed Switch** yolunu izleyin (aynı işleme Networking görünümündeki Distributed Switch kategorisinden de ulaşılır). Sihirbaz birkaç adımdan oluşur ve her biri bir tasarım kararıdır.

#### 1. Ad ve Konum

Switch'e anlamlı bir isim verin. Kalabalık ortamlarda amacı yansıtan bir isimlendirme — `vDS-Production`, `vDS-Storage` gibi — sonraki yönetimi ciddi biçimde kolaylaştırır. Konum olarak switch'in oluşturulacağı datacenter otomatik seçilidir.

#### 2. Sürüm Seçimi (En Kritik Karar)

Sihirbaz sizden vDS sürümünü seçmenizi ister ve buradaki kural nettir: **vDS sürümü, o switch'e katılacak en eski ESXi host sürümünü desteklemek zorundadır.**

* Ortamınızdaki tüm host'lar aynı ve güncel sürümdeyse (örneğin hepsi ESXi 6.5+), en yüksek sürümü seçin.
* Farklı sürümler bir arada bulunuyorsa (örneğin bazı host'lar 6.5, bazıları daha eski), **en eski host'un desteklediği sürümü** seçmelisiniz — aksi halde eski host'lar bu vDS'e katılamaz.

Buradaki takas şudur: daha eski bir sürüm seçtiğinizde, yeni sürümlerle gelen özellikleri kaybedersiniz. Sürümler arasındaki fark yalnızca uyumluluk değil, **yetenek setidir**:

* Daha yeni sürümler port mirroring geliştirmeleri, **Network I/O Control v3 (NIOC v3)** ve gelişmiş LACP gibi özellikler sunar.
* Eski sürümler traffic filtering/marking gibi temel özelliklerle sınırlı kalır.

Bu nedenle vDS'in tüm yeteneklerinden faydalanmak istiyorsanız, önce host'larınızı güncel sürüme taşımak, sonra en yüksek vDS sürümünü seçmek doğru yaklaşımdır. Not: vDS sürümü sonradan **yükseltilebilir** (upgrade), ancak düşürülemez — bu yüzden host uyumluluğunu baştan doğru değerlendirmek önemlidir.

#### 3. Uplink Sayısı ve Temel Ayarlar

Sihirbazın son yapılandırma adımında birkaç önemli seçim yaparsınız:

* **Number of uplinks:** Bu vDS'e katılacak her host'un sağlayabileceği **maksimum fiziksel adaptör (uplink) sayısıdır**. Kritik nokta: bu uplink'ler vCenter sunucusunda değil, **her host'un kendi fiziksel NIC'lerinde** bulunur. Buradaki sayı, soyut uplink slotlarının (Uplink 1, Uplink 2...) kaçını tanımlayacağınızı belirler; her host sonradan kendi vmnic'lerini bu slotlara eşler. Varsayılan 4'tür; ortamınızın host başına ayırabileceği NIC sayısına göre belirleyin — gereğinden fazla tanımlamak zarar vermez, ama gerçekçi bir değer topolojiyi okunur tutar.
* **Network I/O Control:** Varsayılan olarak etkindir ve **etkin bırakılması önerilir.** NIOC, konsolide uplink'lerde trafik türlerine (vMotion, VM, storage) bant genişliği payı tanımlayarak birinin diğerini boğmasını engeller — vDS'in en değerli yeteneklerinden biridir ve baştan açık olması ileride ince ayara zemin hazırlar.
* **Default port group:** Sihirbaz, switch ile birlikte bir varsayılan distributed port group oluşturmayı teklif eder ve adını özelleştirmenize izin verir. Bu port group, Standard Switch'te oluşturduğumuz VM port group'unun merkezi karşılığıdır; VM'ler buraya bağlanacaktır.

Standard Switch mantığı burada birebir korunur: nasıl orada switch + port group + VMkernel yapısı vardıysa, vDS'te de distributed port group'lar ve (sonraki adımlarda ekleyeceğimiz) VMkernel portları aynı rolü üstlenir — yalnızca merkezi tanımlı olarak.

#### 4. Gözden Geçirme ve Oluşturma

Özet ekranında sürüm, uplink sayısı, NIOC durumu ve LACP gibi ayarları doğrulayıp **Finish** ile switch'i oluşturursunuz.

### Oluşturuldu — Peki Şu An Ne Var, Ne Yok?

Bu noktada kritik bir gerçeği vurgulamak gerekir: **vDS oluşturuldu, ancak henüz hiçbir trafik taşımıyor.** Yeni switch şu an "boş bir kabuk"tur:

* Henüz **hiçbir host** bu vDS'e eklenmedi.
* Dolayısıyla **hiçbir fiziksel uplink** eşlenmedi.
* Default port group tanımlı ama içinde henüz **VM yok**.

vDS'in çalışır hale gelmesi için gereken adımlar bundan sonra gelir: host'ların vDS'e eklenmesi, her host'un fiziksel vmnic'lerinin soyut uplink'lere eşlenmesi, ve VM ile VMkernel trafiğinin distributed port group'lara taşınması. Switch oluşturmak, yalnızca merkezi tanımı yaratmaktır; onu işlevsel kılan, host'ların ona bağlanmasıdır.

### vDS'in Vaadi: Tutarlı Ağ ve Sorunsuz vMotion

vDS'i seçmenin asıl gerekçesi, oluşturma ekranının kendi açıklamasında da özetlenir: **bir Distributed Switch, ilişkili tüm host'lar genelinde tek bir sanal switch gibi davranır** ve bu sayede sanal makineler host'lar arasında taşınırken tutarlı ağ yapılandırmasını korur.

Bu, Standard Switch'in en can sıkıcı sınırını çözer. Standard Switch'li bir ortamda bir VM'i başka host'a vMotion ettiğinizde, hedef host'ta **birebir aynı isimde** bir port group bulunmak zorundaydı; en küçük bir isim veya VLAN farkı migration'ı başarısız kılıyordu. vDS'te bu risk ortadan kalkar: VM, host'a değil, tüm host'lar genelinde ortak olan **distributed port group'a** bağlıdır. Nereye taşınırsa taşınsın, ağ tanımı onunla birlikte gelir.

Kavramsal olarak vDS yapılandırması iki katmana yayılır:

* **Datacenter katmanı:** Distributed switch'in kendisi ve distributed port group'lar burada oluşturulur ve yönetilir. Ağın "beyni" burada tanımlıdır.
* **Host katmanı:** Her host'un fiziksel adaptörleri ve VMkernel servisleri, ya tek tek host yapılandırmasıyla ya da **Host Profiles** ile bu merkezi switch'e ilişkilendirilir. Trafiğin fiilen aktığı katman budur.

Bu iki katmanlı yapı, önceki bölümde değindiğimiz control plane (vCenter) / data plane (host) ayrımının pratikteki görünümüdür: tanım merkezde, icra host'ta.

### Sonuç

Yeni bir vSphere Distributed Switch oluşturmak, sihirbaz açısından birkaç adımlık bir işlem olsa da, her adım ileride tüm cluster'ı etkileyecek bir tasarım kararıdır. Özetle:

* vDS **datacenter seviyesinde** oluşturulur; host seviyesinde görünmez, çünkü doğası gereği birden fazla host'a hizmet eden merkezi bir nesnedir.
* **Sürüm seçimi** en kritik karardır: en eski ESXi host'u desteklemek zorundadır ve seçilen sürüm, kullanabileceğiniz yetenek setini (NIOC v3, gelişmiş LACP, port mirroring) belirler. Sonradan yükseltilebilir ama düşürülemez.
* **Uplink sayısı** her host'un sağlayacağı fiziksel adaptör slotlarını tanımlar; bu adaptörler vCenter'da değil host'larda bulunur. **NIOC'u etkin bırakın.**
* Oluşturulan vDS henüz boştur; host eklenene ve uplink'ler eşlenene kadar trafik taşımaz.

Merkezi switch nesnesi artık hazır. Serinin devamında bu boş kabuğu işlevsel hale getireceğiz: host'ları vDS'e ekleyecek, fiziksel uplink'leri soyut uplink slotlarına eşleyecek ve VM trafiğini Standard Switch'ten Distributed Switch'e taşıyacağız.
