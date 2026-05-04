---
icon: grip
---

# Introduction to vCenter Server: Windows Hardware Requirements

Sanallaştırma dünyasına adım atarken genellikle her şeyi tek bir ESXi host üzerinden yöneterek başlarız. Tıpkı önceki konularda yaptığımız gibi, doğrudan host'un IP adresine bağlanır ve sanal makinelerimizi oluştururuz. Ancak sunucu sayınız (host) ve sanal makine sayınız arttıkça, her bir host'a ayrı ayrı IP, kullanıcı adı ve şifre ile bağlanıp onları bireysel olarak yönetmek bir kabusa dönüşür.

İşte tam bu noktada, veri merkezinizin tüm bileşenlerini tek bir ekrandan yönetmenizi sağlayan vCenter Server devreye girer. vCenter sadece merkezi bir yönetim arayüzü değildir; High Availability, vMotion (Çalışır Durumda Taşıma), DRS (Dinamik Kaynak Yönetimi) gibi sanallaştırmanın o "büyülü" gelişmiş özelliklerini kullanabilmenizin tek anahtarıdır.

#### 🧩 vCenter Server Kurulum Mimarisi ve Platform Services Controller (PSC)

vCenter Server kurulumundaki en önemli adım, Platform Services Controller (PSC) bileşenini nereye kuracağınıza karar vermektir.

Platform Services Controller (PSC), VMware altyapısının omurgasını oluşturan temel servisleri tek bir noktada toplayan kritik bir bileşendir. Tam olarak ne yaptığını anlamak için barındırdığı dört temel servise bakmak gerekir:

1. vCenter Single Sign-On (SSO): Kimlik doğrulama (authentication) sürecini yönetir. Active Directory veya OpenLDAP gibi sistemlerle entegre olarak, ortamdaki tüm vSphere bileşenlerine (vCenter, NSX, vb.) tek bir hesap üzerinden giriş yapılabilmesini sağlar.
2. VMware Certificate Authority (VMCA): Ortamdaki ESXi host'lar ve vCenter bileşenleri arasındaki iletişimin şifreli ve güvenli olması için gereken dijital sertifikaları otomatik olarak üretir, dağıtır ve yönetir.
3. Lookup Service: Ortamdaki tüm vSphere servislerinin birbirini bulabilmesi ve haberleşebilmesi için bir kayıt ve adres defteri (service registry) görevi görür.
4. License Service: Altyapıdaki tüm VMware lisanslarının merkezi olarak depolandığı ve atamalarının yapıldığı servistir.

**Kurulum için iki farklı mimari bulunur:**

* 1\. Embedded PSC: vCenter servisleri ile PSC bileşeninin aynı sunucuya kurulduğu yapıdır. Kurulumu, yedeklenmesi ve yönetimi en pratik yöntemdir. Eğer ortamınızda sadece tek bir vCenter çalıştıracaksanız, kullanmanız gereken standart mimari budur.
* 2\. External PSC: Ortamınızda birden fazla vCenter varsa ve hepsinin ortak bir Single Sign-On (SSO) ve lisans altyapısıyla tek bir merkezden (Enhanced Linked Mode) yönetilmesini istiyorsanız kullanılır. Bu yapıda PSC bileşeni ayrı bir sunucuya, vCenter servisleri ise başka bir sunucuya kurulur. Sistem mimarisi gereği kurulumda her zaman önce PSC sunucusu ayağa kaldırılır, ardından vCenter sunucuları kurularak bu merkezi PSC'ye bağlanır.

**💾 Veritabanı Seçimi: Embedded vs. External**

vCenter Server, binlerce sanal makinenin durumunu, loglarını ve performans verilerini aklında tutabilmek için ciddi bir veritabanına ihtiyaç duyar.

Eğer ortamınız küçük veya orta ölçekliyse (Maksimum 20 Host ve 200 Sanal Makine), vCenter kurulumuyla birlikte ücretsiz gelen ve sisteme gömülü olarak kurulan PostgreSQL veritabanı sizin için fazlasıyla yeterli olacaktır.

Ancak bu limitleri aştığınız devasa bir veri merkezine sahipseniz, gömülü veritabanı yetersiz kalır ve vCenter'ı dışarıdaki güçlü bir Oracle veya Microsoft SQL veritabanı sunucusuna bağlamanız zorunlu hale gelir.

**⚙️ Donanım İhtiyaçları ve Boyutlandırma**

vCenter Server sıradan bir uygulama değildir; çok ciddi RAM ve CPU tüketen ağır bir servistir. Kurulum sırasında sihirbaz size ortamınızın büyüklüğünü sorar ve donanımı buna göre yapılandırır:

* Tiny (Çok Küçük - Lab Ortamları): 10 Host ve 100 VM'e kadar. En az 2 vCPU ve 10 GB RAM talep eder. (Eğitim amaçlı çok kısıtlı ortamlarda 8 GB RAM ile kurulum zorlanabilir ancak performans sorunları yaşatır).
* Small: 100 Host ve 1000 VM'e kadar. 4 vCPU ve 16 GB RAM gerektirir.
* Medium: 8 vCPU ve 24 GB RAM.
* Large: 16 vCPU ve 32 GB RAM.

Depolama tarafı günümüz sunucuları için genellikle sorun teşkil etmez, ancak vCenter'ın kendi işletim sistemi ve üreteceği devasa log dosyaları için yeterli disk alanı ayrılmalıdır.

**🐧 vCenter Server Appliance (vCSA) vs. Windows Tabanlı vCenter**

Bazı eğitim ortamlarında vCenter'ın bir Windows Server (2008, 2012 veya 2016) üzerine kurulduğu senaryo işlenmektedir. Ancak VMware dünyasında çok daha popüler ve güçlü bir alternatif vardır: vCenter Server Appliance (vCSA).

Eğer elinizde boş bir Windows Server lisansınız yoksa veya vCenter'ı Windows güncellemeleriyle uğraşmadan izole bir sistemde çalıştırmak istiyorsanız vCSA tam size göredir. vCSA, VMware tarafından özel olarak optimize edilmiş, kendi Linux tabanlı (Photon OS) işletim sistemine sahip, kapalı bir kutu (Appliance) olarak gelir.

Günümüzde VMware, Windows tabanlı vCenter kurulumlarını yavaş yavaş terk etmekte ve tüm sektörü bu Linux tabanlı vCSA modeline yönlendirmektedir.

Özetle; vCenter Server, dağınık haldeki sunucularınızı (Host) tek bir zeka altında birleştirip onlara orkestrasyon yeteneği kazandıran veri merkezi beynidir. Doğru mimariyi, doğru veritabanını ve yeterli donanımı seçmek, bu beynin sağlıklı çalışması için en kritik adımdır.
