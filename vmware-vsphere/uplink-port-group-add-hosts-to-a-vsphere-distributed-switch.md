---
icon: link-simple-slash
---

# Uplink Port Group: Add Hosts to a vSphere Distributed Switch

Şu ana kadar vDS tarafında merkezi tanımı kurduk: switch'in kendisini oluşturduk ve sanal makinelerin bağlanacağı distributed port group'ları tanımladık. Ancak bu yapı hâlâ trafik taşımıyor — çünkü switch'in **fiziksel dünyaya açılan tarafı** boş: hiçbir host eklenmemiş, hiçbir fiziksel adaptör uplink slotlarına eşlenmemiş durumda.

Bu makalede vDS'in fiziksel katmanını devreye alıyoruz: **uplink port group** kavramını netleştiriyor, host'ları vDS'e ekleme sürecini adım adım ele alıyor ve hangi fiziksel adaptörün seçileceği kararının neden kritik olduğunu inceliyoruz.

### Uplink Port Group Nedir?

Uplink port group, vDS'in **fiziksel tarafını** temsil eden özel bir port group'tur. Distributed port group'lar VM ve VMkernel trafiğini taşırken, uplink port group Distributed Switch'i her host'un fiziksel network adaptörlerine bağlar.

Buradaki en önemli tanım, uplink sayısının ne anlama geldiğidir:

> **vDS üzerindeki uplink sayısı, host başına izin verilen maksimum fiziksel bağlantı sayısıdır.**

Yani switch'i dört uplink ile oluşturduysanız, bu vDS'e katılan **her host** bu switch'e en fazla dört fiziksel adaptör bağlayabilir. Host'unuzda altı fiziksel NIC bulunsa bile, bu vDS üzerinden yalnızca dördünü kullanabilirsiniz. Sayı toplam bir havuz değil, host başına bir tavandır.

Bu tasarımın mantığı soyutlamadır: `Uplink 1`, `Uplink 2` gibi slotlar, fiziksel adaptör isimlerinden bağımsız birer yer tutucudur. Her host kendi vmnic'lerini bu slotlara eşler ve politikalar (teaming, failover) fiziksel port isimlerine değil, bu soyut slotlara göre tanımlanır. Böylece host'lar arasındaki donanım farklılıkları merkezi politikayı bozmaz.

### Host'ları vDS'e Ekleme

vDS'e sağ tıklayıp **Add and Manage Hosts** seçeneğiyle başlayın. Sihirbaz üç görev sunar:

* **Add hosts:** Yeni host'ları vDS'e ekler (bu makalenin konusu)
* **Manage host networking:** Zaten ekli host'ların adaptör ve VMkernel yapılandırmasını düzenler
* **Remove hosts:** Host'ları vDS'ten çıkarır

#### 1. Host seçimi

**New hosts** ile ekleyeceğiniz host'ları seçersiniz. Burada önemli bir kapsam kuralı vardır: listede yalnızca **aynı datacenter içindeki** host'lar görünür. Başka bir datacenter'daki host'ları bu vDS'e ekleyemezsiniz; orada ayrı bir vDS oluşturmanız gerekir. Bu, vDS'in datacenter seviyesinde tanımlı bir nesne olmasının doğal sonucudur.

Birden fazla host'u aynı anda seçebilirsiniz — vDS'in tekrar problemine getirdiği çözüm burada somutlaşır: bir kez yapılandırıp tüm host'lara uygularsınız.

#### 2. Hangi görevlerin yapılacağını seçme

Sihirbaz size üç kalemi ayrı ayrı yönetme imkânı verir ve her birini işaretleyip kaldırabilirsiniz:

* **Manage physical adapters:** Fiziksel NIC'leri uplink slotlarına eşleme
* **Manage VMkernel adapters:** Mevcut vmk portlarını vDS'e taşıma veya yeni oluşturma
* **Migrate virtual machine networking:** VM'leri distributed port group'lara taşıma

Production ortamında **doğru yaklaşım bu adımları ayrı ayrı, kademeli yapmaktır.** Önce yalnızca fiziksel adaptörleri eşleyin; yapının sağlıklı çalıştığını doğrulayın; ardından VMkernel ve VM migrasyonlarını ayrı işlemlerde gerçekleştirin. Üçünü tek seferde yapmak teknik olarak mümkündür, ancak bir hata durumunda neyin bozulduğunu tespit etmek çok daha zordur — özellikle management vmk'sı da taşınıyorsa host'un vCenter bağlantısını kaybetme riski vardır.

#### 3. Fiziksel adaptörleri uplink'lere eşleme

Bu adımda her host'un fiziksel adaptörleri, mevcut kullanım durumlarıyla birlikte listelenir. Tipik bir tablo şöyle görünür:

* `vmnic0` → `vSwitch0`'a bağlı (kullanımda)
* `vmnic1` → başka bir Standard Switch'e bağlı (kullanımda)
* `vmnic2`, `vmnic3` → boşta

Bir adaptörü seçip **Assign uplink** diyerek hedef uplink slotunu (`Uplink 1`, `Uplink 2`...) belirlersiniz.

**Kritik karar: Hangi adaptörü seçmeli?**

Buradaki temel prensip: **mümkün olduğunca boştaki fiziksel adaptörleri kullanın.**

Halihazırda bir Standard Switch tarafından kullanılan bir adaptörü vDS'e atamak teknik olarak mümkündür — adaptör otomatik olarak eski switch'ten ayrılır. Ancak bunun iki sonucu vardır:

* **Eski switch zayıflar:** İki uplink'li bir Standard Switch'ten adaptör çektiğinizde o switch tek uplink'e düşer ve redundancy'sini kaybeder. Yedekli olduğunu sandığınız yapı, farkında olmadan single point of failure haline gelir.
* **Yük yoğunlaşır:** Zaten trafik taşıyan bir adaptörü ikinci bir göreve koşmak, o NIC üzerindeki yükü artırır.

Boşta adaptör yoksa ve mevcut olanlardan birini taşımak zorundaysanız, bunu **kademeli** yapın: önce bir adaptörü taşıyıp yapıyı doğrulayın, sonra ikincisini. Her iki uplink'i aynı anda çekmek, geçiş sırasında bağlantı kaybına yol açabilir.

**Uplink slot numaraları host'lar arasında aynı olmak zorunda değil**

Sık sorulan bir konu: Host A'da `vmnic2` → `Uplink 1` eşlerken, Host B'de `vmnic1` → `Uplink 1` eşleyebilirsiniz. Slot numaralarının host'lar arasında birebir örtüşmesi zorunlu değildir; soyutlamanın amacı da tam olarak budur — farklı donanım yapılandırmalarına sahip host'lar aynı vDS'e sorunsuz katılabilir.

Yine de **best practice, tutarlı bir eşleme deseni kullanmaktır.** Örneğin "her host'ta ilk 10GbE portu Uplink 1'e, ikincisi Uplink 2'ye" gibi bir kural benimsemek, teaming politikalarının her host'ta beklendiği gibi davranmasını sağlar ve sorun teşhisini kolaylaştırır. Özellikle explicit failover order kullanıyorsanız (Uplink 1 Active, Uplink 2 Standby gibi), slot eşlemesindeki tutarsızlık her host'ta farklı bir fiziksel yolun aktif olması anlamına gelir.

#### 4. Etki analizi ve tamamlama

Sihirbaz, yapacağınız değişikliklerin etkisini değerlendirir ve iSCSI gibi servislerin etkilenip etkilenmeyeceğini bildirir. **"No impact"** çıktısı, seçtiğiniz adaptörlerin taşınmasının mevcut servisleri kesintiye uğratmayacağı anlamına gelir; uyarı çıkarsa değişikliği gözden geçirin.

Özet ekranında kaç host'un ekleneceğini ve kaç fiziksel adaptörün eşleneceğini doğrulayıp **Finish** ile işlemi tamamlarsınız.

### Sonucu Doğrulama

İşlem sonrası vDS'in fiziksel tarafını birkaç yerden kontrol edebilirsiniz:

* **Uplink port group → Ports:** Hangi uplink slotunun hangi host'un hangi vmnic'iyle eşlendiğini gösteren tablo. Örneğin `Uplink 1` altında Host-A/`vmnic2` ve Host-B/`vmnic1` yan yana görünür.
* **vDS → Hosts:** vDS'e katılmış host'ların listesi.
* **Topology:** Uplink slotlarının artık boş olmadığını, eşlenmiş adaptör sayılarını gösterdiğini görürsünüz.

Bu noktada önemli bir gerçeği vurgulamak gerekir: **vDS artık fiziksel ağa bağlı, ancak hâlâ VM trafiği taşımıyor.** Sanal makinelerin tamamı hâlâ Standard Switch'lerdeki port group'lara bağlıdır. Distributed port group'lar tanımlı ama boştur.

Bu ara durum aslında bir avantajdır: yeni ağ yolunu, üzerinde henüz hiçbir production iş yükü yokken kurup doğrulama imkânı verir. Geçişin doğru sırası da budur — önce yolu hazırla, sonra trafiği taşı.

### Geçiş Sırasında Dikkat Edilecekler

Standard Switch'ten vDS'e geçiş, planlı yapılması gereken bir operasyondur:

* **Konsol erişimini hazır tutun:** Fiziksel adaptör taşıma işlemleri sırasında beklenmedik bir bağlantı kaybına karşı iLO/iDRAC/IPMI erişiminiz ve DCUI kurtarma seçenekleri (Network Restore Options) hazır olmalıdır.
* **Fiziksel switch tarafını doğrulayın:** vDS'e atadığınız adaptörlerin bağlı olduğu fiziksel portların VLAN, MTU ve trunk yapılandırması, taşınan trafiğin gereksinimleriyle uyumlu olmalıdır. Bu adım atlandığında sorun ancak trafik taşınmaya başlayınca ortaya çıkar.
* **Adım adım ilerleyin:** Bir host'ta tüm süreci tamamlayıp doğrulamak, ardından diğerlerine geçmek; hepsini aynı anda taşımaktan çok daha güvenlidir. vDS'in gücü tekrarı ortadan kaldırmasıdır, ancak bu güç geçiş anında dikkatli kullanılmalıdır.

### Sonuç

Host ekleme ve uplink eşleme adımı, vDS'i kavramsal bir tanımdan çalışan bir ağ bileşenine dönüştüren aşamadır. Özetle:

* **Uplink port group**, vDS'in fiziksel tarafıdır; uplink sayısı toplam bir havuz değil, **host başına maksimum fiziksel bağlantı** tavanıdır.
* Host seçimi datacenter ile sınırlıdır; birden fazla host aynı anda eklenebilir ve vDS'in tekrarı ortadan kaldıran yapısı burada devreye girer.
* Adaptör seçiminde öncelik **boştaki NIC'lerdir**; kullanımdaki bir adaptörü çekmek, kaynak switch'in redundancy'sini bozar.
* Uplink slot numaraları host'lar arasında farklı olabilir, ancak **tutarlı bir eşleme deseni** teaming politikalarının öngörülebilir çalışması için önemlidir.
* İşlem sonrası vDS fiziksel ağa bağlıdır ama VM trafiği taşımaz; bu ara durum, geçişi güvenle doğrulama fırsatıdır.

Fiziksel yol artık hazır. Serinin devamında son adımı atacağız: sanal makinelerin ağ bağlantılarını Standard Switch'ten distributed port group'lara taşımak ve vDS'i tam anlamıyla devreye almak.
