---
icon: octagon-plus
---

# Creating A Virtual Machine In vCenter Server

Bağımsız bir ESXi host üzerinde sanal makine (VM) yaratmak ile vCenter gibi merkezi bir yönetim platformunda sanal makine yaratmak temel mantık olarak aynı olsa da, kurumsal bir ortamın getirdiği esneklik ve seçenek zenginliği açısından çok ciddi farklılıklar barındırır.

Bu rehberde, vCenter arayüzünü kullanarak sıfırdan bir sanal makine oluşturma sürecinin mühendislik detaylarını ve bu sürecin eski usul host arayüzüne göre bize sağladığı avantajları inceleyeceğiz.

**1. Başlangıç Noktası: Hedefi Belirlemek veya Seçimi vCenter'a Bırakmak**

vCenter'da bir sanal makine yaratmaya başlarken nereden tıkladığınız, sihirbazın size soracağı soruları doğrudan etkiler. Bu, vCenter'ın hiyerarşik ve Context-Aware yapısının bir sonucudur.

* Host Üzerinden Tıklamak: Eğer doğrudan ağaç yapısındaki spesifik bir host'a (örneğin Host-10) sağ tıklayıp "New Virtual Machine" derseniz, vCenter zımni olarak "Bu makineyi bu donanımın üzerine kurmak istediğinizi" anlar ve size kurulum aşamasında "Hangi host'u istiyorsun?" diye sormaz.
* Datacenter Üzerinden Tıklamak: Eğer en üstteki "Morocco" adlı Datacenter'a sağ tıklayıp işlemi başlatırsanız, vCenter size sırayla "Hedef Datacenter neresi?" ve ardından "Hangi Host üzerinde çalışmasını istiyorsun?" sorularını sorar. Kurumsal mimarilerde bu esneklik, donanım kaynaklarını optimize etmek için çok değerlidir.

**2. Ufku Genişletmek: 6 Farklı Yaratılış Seçeneği**

Sıradan bir ESXi arayüzünde VM yaratırken sadece "Sıfırdan Yarat" (Create New) seçeneğiniz varken, vCenter ortamında "Yeni Sanal Makine" sihirbazı size tam 6 farklı operasyonel seçenek sunar. Bu seçenekler, sistem yöneticisinin günlük mesaisini saatlerden dakikalara indiren en güçlü özelliklerdir:

* Create New Virtual Machine: İşletim sistemini bir ISO dosyasından kurduğunuz, RAM ve disk gibi donanım ayarlarını tamamen manuel olarak yapılandırdığınız klasik sıfırdan Virtual Machine oluşturma yöntemidir.
* Deploy from a Template: Önceden hazırladığınız, içinde işletim sistemi ve gerekli yazılımların kurulu olduğu bir Template üzerinden, dakikalar içinde tamamen kullanıma hazır yeni bir Virtual Machine oluşturma (Deploy) işlemidir.
* Clone an Existing Virtual Machine: Halihazırda var olan bir Virtual Machine'in diskini ve tüm konfigürasyonunu birebir kopyalayarak, sistemde tamamen aynı özelliklere sahip yeni bir makine daha üretme işlemidir.
* Clone a Virtual Machine to a Template: Orijinal Virtual Machine'in yapısını bozmadan, onun birebir kopyasını alarak sistemde yeni bir Template oluşturma işlemidir. Bu sayede asıl makineniz normal hayatına devam ederken, kopyası ileride yeni makineler Deploy etmek için arşivlenmiş olur.
* Clone Template to Template: Mevcut bir Template'in farklı bir isimle kopyasını (yedeğini) alma işlemidir. Genellikle bir Template üzerinde değişiklik yapmadan önce orijinal halini korumak amacıyla kullanılır.
* Convert Template to Virtual Machine: Salt okunur haldeki bir Template'i tekrar normal, çalıştırılabilir bir Virtual Machine'e dönüştürme işlemidir. Genellikle Template içindeki işletim sistemine Update geçmek veya yeni bir program kurmak gerektiğinde makine formuna dönülür; güncelleme bitince makine tekrar Template formuna geri çevrilir.

**3. Donanım Planlaması: Kaynakların İnce Ayarı**

İşletim sistemi türünü ve donanım versiyonunu (ESXi 6.5 vb.) seçtikten sonra en kritik aşamaya gelinir: Customize Hardware (Donanım Özelleştirme). vCenter bu ekranda size körü körüne bir menü sunmaz; seçtiğiniz işletim sisteminin mimari sınırlarını (Best Practices) bildiği için sizi yönlendirir.

* İşletim Sistemine Duyarlı RAM Yapılandırması: Yeni bir Virtual Machine oluştururken CentOS gibi spesifik bir işletim sistemi seçtiğinizde, vCenter donanım ekranında sadece manuel bir değer alanı sunmaz. Sistem yöneticisine rehberlik etmek üzere, seçilen işletim sisteminin mimarisine uygun Minimum, Default ve Best Performance RAM değerlerini referans olarak gösterir. Ayrıca, işletim sisteminin desteklediği Maximum limiti (örneğin 64 GB) belirterek mimariye aykırı kaynak tahsisi (resource allocation) yapılmasının önüne geçer.
* Disk Provisioning: Varsayılan olarak sistem güvenli olan Thick Provision Lazy Zeroed seçeneğini getirir (Yani 50 GB disk verdiyseniz, fiziksel Datastore'da o an 50 GB yer kapatılır). Ancak depolama alanından tasarruf etmek için, sadece kullandıkça büyüyen Thin Provision mantığını seçmek kurumsal ortamlarda en sık yapılan donanım müdahalesidir. Thin seçeneği sayesinde fiziksel disk boyutunuzu aşan sanal diskler tanımlayabilirsiniz (Ancak bu durum gelecekte Datastore'un tamamen dolması riskini yaratır, bu yüzden sürekli izlenmelidir).

**4. Konsol ve ISO Bağlama Pratiği**

Sanal makine başarıyla oluşturulduktan (Power On) sonra, içindeki boş diske bir işletim sistemi kurmanız gerekir. vCenter, makineyi uzaktan yönetmek için size iki ana konsol sunar:

* Web Console: Hiçbir eklenti kurmadan doğrudan tarayıcınızın içinde açılan pratik ekrandır.
* VM Remote Console (VMRC): Bilgisayarınıza kurulan ayrı bir masaüstü uygulamasıdır. Mouse ve klavye etkileşimi çok daha sağlıklıdır ve ISO dosyalarını bağlamak için daha güvenilirdir.

Kurulum medyasını (ISO) bağlarken "Edit Settings > CD/DVD Drive" yolunu izlersiniz. ISO dosyasını kendi bilgisayarınızdan (Client Device), ESXi host'un takılı olan fiziksel CD-ROM'undan veya doğrudan fiziksel sunuculardaki disk alanı olan Datastore üzerinden bağlayabilirsiniz. (Kurumsal yapılarda ISO dosyaları genellikle Datastore'da özel bir "ISO\_Klasoru" içinde tutulur ki her host bu kurulum dosyalarına ağ üzerinden çok hızlı erişebilsin).

Özetle: vCenter üzerinden makine yaratmak, sadece bir bilgisayar oluşturmak değil; şablonlar, klonlar ve akıllı kaynak yönetimi sayesinde devasa bir sunucu çiftliğini standartlaştırma ve yönetme sanatıdır.
