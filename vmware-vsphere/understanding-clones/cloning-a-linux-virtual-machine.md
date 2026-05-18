---
icon: linux
---

# Cloning a Linux Virtual machine

Sanallaştırma dünyasında Windows sunucuları klonlamak ve Sysprep ile otomatize etmek ne kadar standart bir işlemse, Linux (Ubuntu, CentOS, RedHat vb.) sunucuları klonlamak da modern altyapıların (özellikle Kubernetes ve web sunucu çiftliklerinin) o kadar vazgeçilmez bir parçasıdır.

Bir Linux makinesinin klonlama süreci; Host seçimi, Datastore tahsisi ve Disk formatı gibi temel adımlar açısından Windows ile birebir aynıdır. Ancak iş "Customize the operating system" (İşletim Sistemini Özelleştirme) adımına geldiğinde, vCenter mimarisinin keskin kuralları devreye girer.

Bu makalede, Linux makine klonlarken karşılaşılan temel farklılıkları, vCenter'ın şablon izolasyon mantığını ve sihirbazdaki ince ayarları inceleyeceğiz.

**1. Kesin Çizgi: Şablon İzolasyonu (Liste Neden Boş Gelir?)**

Linux makineyi klonlarken "Özelleştirme" (Customization) ekranına geldiğinizde karşılaşacağınız ilk durum, listenin tamamen boş olmasıdır. (Halihazırda ortamınızda Windows için hazırlanmış şablonlarınız olsa bile).

Bunun kaputun altındaki sebebi, vCenter'ın "Şablon İzolasyonu" (Template Isolation) kuralıdır:

* vCenter, klonlamaya çalıştığınız kaynak sanal makinenin işletim sistemine bakar. Eğer bu makine bir Linux ise, özelleştirme adımında size _sadece_ Linux için hazırlanmış şablonları gösterir.
* Windows şablonları (içinde Sysprep komutları ve Microsoft lisans anahtarları barındırdığı için) Linux klonlama ekranında bilerek gizlenir.
* Yeni bir şablon (Specification) yaratmak istediğinizde, Target OS (Hedef İşletim Sistemi) otomatik olarak Linux şeklinde kilitli gelir ve bunu değiştiremezsiniz. Bu mimari, sistem yöneticisinin yanlışlıkla bir Linux makineye Windows kimliği basmasını fiziksel olarak engeller.

**2. Linux Customization Specification: Windows'tan Farkları Neler?**

Yeni bir Linux şablonu oluştururken karşınıza çıkan sihirbaz, Windows sihirbazına çok benzer ancak Linux çekirdeğinin (Kernel) ve ağ yapısının doğası gereği bazı temel farklılıklar barındırır:

* Domain Name (Alan Adı) Zorunluluğu: Windows'ta makineyi Workgroup'ta (Bağımsız) bırakma seçeneğiniz varken, Linux şablonu oluştururken sihirbaz sizden mutlaka bir Domain Name (Örn: `vmlab.local` veya `sirket.com`) girmenizi ister. Linux makineler ağda kendilerini FQDN (Tam Nitelikli Alan Adı - Örn: `web-server.sirket.com`) ile anons ettikleri için bu alanın doldurulması mimari bir gerekliliktir.
* Saat Dilimi ve Donanım Saati (Hardware Clock): Sihirbaz size saat dilimini sorar. Ayrıca Linux sistemlerine has olan "Donanım saati UTC mi yoksa Yerel Saat mi olsun?" ayrımı devreye girer (Sunucu mimarilerinde log tutarlılığı için genellikle UTC tercih edilir).
* DNS ve WINS Farkı: Windows sistemlerde ağ isim çözümlemesi için WINS ve NetBIOS gibi Microsoft'a özel eski protokoller bulunurken, Linux tamamen saf DNS ve DNS Search Domains yapısına güvenir. Bu nedenle Linux şablonunda WINS yapılandırma sekmesi bulunmaz, DNS ayarları doğrudan ve net bir şekilde girilir.

**3. Kritik Bir Hata ve Gerçek Nedeni: "Customization Not Supported"**

İşlemi tamamlayıp klonlamayı başlattığınızda, özellikle test ortamlarında sıkça karşılaşılan kırmızı bir hataya değinmekte fayda var: _"Customization of the guest operating system is not supported in this configuration."_

Bu hatanın vCenter'ın yeteneksizliğiyle hiçbir ilgisi yoktur. Hata, klonlanmaya çalışılan kaynak sanal makinenin içinin tamamen boş olmasından (İşletim sistemi ve VMware Tools kurulmamış olmasından) kaynaklanır.

Linux Özelleştirmesinin Arka Planı:

Bir Linux makineyi bu şablonla klonladığınızda, vCenter makine açılırken içeriye (dağıtımın yeniliğine göre) ya bir Perl Scripti (Script) ya da bir Cloud-Init Metadata'sı gönderir.

Eğer makinenin içinde bir Linux işletim sistemi yoksa veya vCenter ile konuşacak olan open-vm-tools (Linux VMware Tools) paketi yüklü değilse, vCenter bu şablonu içeriye enjekte edemez ve işlemi iptal ederek o meşhur hatayı fırlatır.

Özetle; vCenter'da Linux klonlamak, kuralları kendi doğasına göre belirlenmiş, ancak otomasyon gücü açısından Windows ile tamamen aynı seviyede olan bir süreçtir. Doğru hazırlanmış bir Linux şablonu (Customization Spec) ve içinde `open-vm-tools` barındıran altın bir imaj ile, yüzlerce Linux sunucusunu saniyeler içinde benzersiz IP ve isimlerle ağa dahil edebilirsiniz.
