---
icon: book-copy
---

# Cloning a Windows Virtual machine: Storage Options

Sanal makine klonlama (Clone) işlemi sadece bir kopyala-yapıştır operasyonu değildir; donanım kısıtlamalarının ve depolama mimarisinin yeniden yapılandırıldığı stratejik bir geçiş noktasıdır.

Sistem yöneticileri, klonlama işlemi sırasında orijinal makinenin disk formatına körü körüne bağlı kalmak zorunda değildir. Hedef Datastore'un (Depolama Alanı) kapasitesine veya yeni kurulacak sunucunun performans ihtiyaçlarına göre, diskin fiziksel davranış şekli (Thick vs. Thin Provisioning) değiştirilebilir.

**1. Kapasite Çatışması ve Thin/Thick Provisioning Mühendisliği**

Bir sanal makineyi klonlarken "Select virtual disk format" (Sanal disk formatını seç) ekranında vCenter, "Same format as source" (Kaynak ile aynı formatı koru) seçeneğini varsayılan olarak sunar. Ancak iki kritik senaryoda bu varsayılan ayara müdahale edilmesi hayati önem taşır:

Senaryo A: "Büyük Disk, Dar Alan" (Thick'ten Thin'e Geçiş)

Diyelim ki orijinal veritabanı sunucunuz `Thick Provision Lazy Zeroed` formatında oluşturulmuş 50 GB'lık bir sanal diske (VMDK) sahip. Bu format, sanal makinenin o an içinde sadece 10 GB verisi olsa bile, fiziksel Datastore üzerinde 50 GB'lık alanın tamamını bloke ettiği (rezerve ettiği) anlamına gelir.

Siz bu makineyi, sadece 31 GB boş alanı olan hedef bir Datastore'a klonlamak isterseniz vCenter "Yetersiz Disk Alanı" (Insufficient disk space) hatası verecektir.

* Çözüm: Diskin yapısını klonlama esnasında Thin Provision olarak değiştirirseniz, vCenter boş olan 40 GB'ı kopyalamaz; sadece makinenin içinde gerçekten dolu olan o 10 GB'lık veriyi hedef Datastore'a yazar. Sanal makine kendi diskinin hala 50 GB olduğunu sanır, ancak arka planda fiziksel Datastore sadece 10 GB eksilmiş olur. Bu sayede 31 GB'lık alana sığmayı başarırsınız.

Senaryo B: "Esneklikten Garanti Performansa" (Thin'den Thick'e Geçiş)

Varsayalım ki orijinal test makineniz depolamadan tasarruf etmek için `Thin Provision` formatında kurulmuştu ve 16 GB'lık diski sadece kullandığı kadar büyüyordu.

Ancak siz bu makineyi klonlayıp, çok yoğun I/O (Okuma/Yazma) yapacak ve alanın tamamına ihtiyaç duyacak "Canlı (Production) Veritabanı Sunucusu" yapacaksınız. Thin provision diskler, büyürken (fiziksel diskten yeni bloklar talep ederken) ufak da olsa bir performans kaybı yaşatır.

* Çözüm: Klonlama sırasında formatı Thick Provision olarak değiştirirsiniz. vCenter, hedef Datastore'daki o 16 GB'lık bloğun tamamını anında rezerve eder. Böylece yeni veritabanı sunucunuz, disk büyüme bekleme süreleri yaşamadan anında maksimum performansla (garanti alanla) çalışmaya başlar.

**2. Ufuk Açıcı Katkı: Klonlamada Neden IP ve İsim Çatışması Yaşanır? (SID Krizi)**

<figure><img src="../../.gitbook/assets/Screenshot 2026-05-18 at 11.13.40.png" alt=""><figcaption></figcaption></figure>

Bir sanal makineyi klonlamak, kurumsal altyapılarda saatler sürecek kurulum süreçlerini dakikalara indiren muazzam bir yetenektir. Ancak bu gücün kontrolsüz kullanımı, sistem yöneticilerini ağ mimarisinin en karanlık kabusuyla karşı karşıya bırakabilir: Kimlik Çatışması.

Sihirbazın son adımında yer alan "Customize the operating system" (İşletim Sistemini Özelleştir) seçeneği, sadece basit bir kutucuk değil; bu kabusu önleyen, arka planda devasa bir otomasyonu tetikleyen stratejik bir güvenlik kilididir.

Bu sürecin neden zorunlu olduğunu, altyapıda nasıl yapılandırıldığını ve kaputun altında nasıl çalıştığını tüm detaylarıyla inceleyelim.

**1. "Genetik İkiz" Krizi**

Bir Windows sanal makinesini olduğu gibi (hiçbir ayarına dokunmadan) klonlarsanız, yeni makine orijinalinin birebir genetik ikizi olarak doğar. İki makine aynı anda çalışmaya başladığında sistemde şu kritik çatışmalar yaşanır:

* Aynı Hostname (Sunucu Adı): İki sunucu da ağda aynı isimle (Örn: "SQL-Server") anons yapar. İsim çözümleme (DNS) mekanizmaları çöker.
* Aynı IP Adresi: Her iki makine de aynı statik IP adresiyle ağa çıkmaya çalışır. Ağdaki Switch'ler ARP tablolarını güncelleyemez ve "IP Conflict" (IP Çakışması) yaşanarak her iki sunucunun da bağlantısı kesilir.
* Aynı SID (Security Identifier): En tehlikelisi ve sinsi olanı budur. Windows dünyasında her bilgisayarın benzersiz bir kimlik numarası (SID) vardır. Active Directory Domain yapısına bağlı iki makinenin SID'si aynı olursa, ortamdaki tüm yetkilendirmeler, grup politikaları (GPO) ve güvenlik sınırları iflas eder.

İşte bu felaketi önlemek için, klonlama sihirbazında "Customize the operating system" kutucuğu mutlaka işaretlenmelidir. Bu işlem, Windows'un içindeki kimlik sıfırlama aracı olan Sysprep'i (System Preparation) otomatik olarak tetikleyerek yeni makineye yepyeni bir kimlik kazandırır.

**2. Eksik Halka: "Customization Specification" (Özelleştirme Şablonu)**

Klonlama sihirbazında ilgili kutucuğu işaretleyip "Next" dediğinizde, karşınıza anında sihirli bir çözüm çıkmaz. Eğer ortamı ilk kez kuruyorsanız, karşınıza "No items found" (Öğe bulunamadı) uyarısı veren boş bir liste çıkar.

Bunun sebebi vCenter'ın şu veriye ihtiyaç duymasıdır: _"Evet, kimliği sıfırlayacağım ama bu yeni makinenin adı ne olacak? Şifresi ne olacak? Hangi Domain'e girecek?"_

İşlemin başarıyla tamamlanabilmesi için vCenter üzerinde önceden bir Cevap Anahtarı hazırlanmış olması şarttır. Bu mimari şöyle kurulur:

1. Şablon Kütüphanesi: vSphere Client ana menüsünden Policies and Profiles > VM Customization Specifications bölümüne gidilir.
2. Kural Setinin İnşası: Burada "New" diyerek (örneğin hedef OS Windows seçilerek) yeni bir kural seti yaratılır. Bu setin içine şu parametreler girilir:
   * Computer Name: Genellikle "Use the virtual machine name" seçilir. (Klon sihirbazında yazdığınız isim, otomatik olarak Windows'un içine Hostname olarak gömülür).
   * Administrator Password: Yeni yerel yönetici şifresi.
   * Network (Ağ): Makinelerin DHCP'den mi IP alacağı yoksa kurulum esnasında manuel mi girileceği belirlenir.
   * Domain Join: Makine otomatik olarak Active Directory'ye eklenecekse, Domain adı ve yetkili hesap bilgileri buraya tanımlanır.

**3. Kaputun Altında Sistem Nasıl İşler? (Tam Otomasyon)**

Özelleştirme şablonunuzu seçip klonlama işlemini başlattığınızda arka planda kusursuz bir orkestrasyon çalışır:

1. vCenter, yeni makinenin disk kopyalama işlemini bitirir ve makineye "Power On" komutu gönderir.
2. Makine açılırken vCenter, o hazırladığınız "Cevap Anahtarını" (Yeni isim, yeni IP, Domain şifresi vb.) makinenin içindeki işletim sistemine gizlice enjekte eder.
3. Windows açıldığında otomatik olarak Sysprep (Generalize) komutunu çalıştırır ve kendini yeniden başlatır.
4. İkinci kez açılışında; eski kimliğinden tamamen arınmış, yepyeni bir SID üretmiş, verdiğiniz yeni IP'yi almış ve Active Directory'ye tertemiz bir şekilde katılmış yepyeni bir sunucu olarak karşınıza çıkar.

> 💡 Kritik Bağımlılık: VMware Tools Köprüsü
>
> Tüm bu otomasyonun çalışabilmesi için vCenter ile Windows arasında bir iletişim köprüsü olması şarttır. "Customize the operating system" komutlarının Windows'un içine iletilip Sysprep'in tetiklenebilmesi için, klonlanan orijinal (kaynak) sanal makinenin içinde VMware Tools uygulamasının kurulu ve çalışıyor olması %100 zorunludur. VMware Tools yoksa, vCenter işletim sisteminin içine müdahale edemez ve otomasyon başarısız olur.

