---
icon: code-fork
---

# New VM Guest Customization Specification: Network Configuration

Bir sanal makineyi (Virtual Machine) klonlamak için bir "Customization Specification" (Özelleştirme Şablonu) oluştururken, işletim sisteminin adını ve şifrelerini belirledikten sonra sistem mimarisinin en kritik aşamasına geçilir: Ağ (Network) ve Kimlik (Identity) Yapılandırması.

Kurumsal bir altyapıda sunucular sürekli yer değiştiren cihazlar değildir; sabit adreslere ve eşsiz kimliklere sahip olmaları gerekir. Bu makalede, şablon oluşturma sürecindeki IP atama stratejilerini, Domain entegrasyonunu ve Windows SID (Security Identifier) mimarisinin kaputun altındaki işleyişini inceleyeceğiz.

**1. IP Adresi Yönetimi ve Conflict Tuzakları**

Sihirbazın ağ yapılandırması (Network) adımında, varsayılan olarak DHCP seçeneği gelir. Ancak sistem mühendisliğinin altın kuralı gereği; sunucular (Web, SQL, File Server vb.) dinamik (değişken) IP adresleriyle değil, Statik (Sabit) IP adresleriyle çalışmalıdır.

Statik IP atamak için özel ayarlar (Custom Settings) menüsüne girildiğinde, sistem yöneticisinin karşısına gelecekteki otomasyonu doğrudan etkileyecek 3 farklı strateji çıkar:

* Tuzak Seçenek (Use the following IP address): Eğer buraya statik bir IP (Örn: `192.168.1.31`) elle yazıp (hardcode) şablonu kaydederseniz, büyük bir sorunla karşılaşırsınız. Bu şablonu kullanarak ilk makineyi kurduğunuzda her şey kusursuz çalışır. Ancak aynı şablonla ikinci veya üçüncü makineyi klonladığınızda, hepsi `.31` IP adresini almaya çalışır ve ağda anında IP Çakışması (IP Conflict) yaşanır. Bu seçenek şablonun "tekrar kullanılabilirliğini" yok eder.
* En İyi Uygulama (Prompt the user for an address when the specification is used): Kurumsal ortamlarda şiddetle tavsiye edilen ayar budur. Bu seçeneği işaretlediğinizde vCenter, IP adresini şablona sabitlemez. Bunun yerine, siz ne zaman bu şablonu kullanarak yeni bir makine klonlamaya kalkarsanız, sihirbazın ortasında durup size _"Bu spesifik makine için hangi IP'yi atayayım?"_ diye sorar. Böylece aynı şablonu kullanarak hiçbir IP çakışması yaşamadan yüzlerce farklı sunucu kurabilirsiniz.

💡 Mimari İpucu: Alt Ağ (Subnet) ve Ağ Geçidi (Gateway)

IP adresini her kurulumda manuel girmeyi seçseniz bile, Subnet Mask, Default Gateway ve DNS Server adreslerini şablona kalıcı olarak yazabilirsiniz. Çünkü aynı VLAN/Ağ içinde kurulan tüm sunucularda bu adresler ortaktır ve her seferinde tekrar yazarak vakit kaybetmenize gerek yoktur.

**2. Domain Entegrasyonu (Active Directory)**

Ağ ayarlarından sonra sistem size bu makinenin bir Workgroup'ta mı kalacağını, yoksa bir Domain'e (Active Directory) mi katılacağını sorar.

Eğer makinenin açılır açılmaz otomatik olarak Domain'e katılmasını istiyorsanız, şablona Domain adını (Örn: `vmlab.local`) ve bu makineyi Domain'e alma (Join) yetkisine sahip bir servis hesabının (Service Account) kullanıcı adı ve şifresini girmelisiniz. Bu sayede makine Sysprep sonrasında açıldığında, Active Directory'ye pırıl pırıl, yepyeni bir sunucu olarak kendi kendini kaydeder.

**3. En Kritik Güvenlik Kilidi: Benzersiz SID Üretimi (Generate New Security ID)**

Tüm Customization Specification sürecinin belkemiği olan, unutulması halinde ortamda kaosa neden olacak en önemli seçenek "Generate New Security ID" kutucuğudur.

Windows işletim sistemlerinde her makinenin parmak izi niteliğinde benzersiz bir SID (Security Identifier) numarası vardır.

* Eğer bir makineyi klonlarken bu kutucuğu işaretlemezseniz, yeni makine eski makinenin SID'sini birebir kopyalar.
* Sonuç: Active Directory, aynı SID ile gelen iki farklı makineyi asla kabul etmez. Yeni makineyi Domain'e almaya çalıştığınızda sistem reddeder. Daha da kötüsü, Domain'de aynı SID'ye sahip iki makine tesadüfen çalışırsa, ortamdaki tüm yetkilendirmeler (Permissions) ve Grup Politikaları (GPO) birbirine girer.

Bu kutucuk işaretlendiğinde, vCenter arka planda Windows'a _"Sen artık eski makine değilsin, Sysprep ile tüm kimliğini sil ve ilk açılışta kendine yepyeni, benzersiz bir SID üret"_ emrini verir.

Özetle; Kusursuz bir "Customization Specification" şablonu; IP adresini kurulum anında soracak şekilde esnek bırakılmış, Subnet/Gateway/DNS ayarları önceden tanımlanmış ve yepyeni bir SID üretmek üzere yapılandırılmış bir şablondur. Bu sayede altyapınızda güvenli, ölçeklenebilir ve tam otomatik bir sunucu dağıtım fabrikası kurmuş olursunuz.

***

Bu iş akışı, sanallaştırma mimarisinin temel taşlarından biridir ve makalenizde okuyucuya "kaputun altındaki" mantığı aktarmak için harika bir bölümdür.

Blog yazınıza doğrudan kopyalayıp yapıştırabileceğiniz, profesyonel BT terminolojisiyle zenginleştirilmiş ve aşamalandırılmış revize metni aşağıda paylaşıyorum:

***

#### VMware Mimarisinde Şablon (Template) Yaşam Döngüsü ve Doğru İş Akışı

VMware mimarisi, kimliklendirme (IP, Hostname, SID ataması) işleminin şablonu "yaratırken" değil, şablonu "kullanırken" (dağıtım anında) yapılmasını şart koşar. Sistemin bir sistem yöneticisinden beklediği, hatalara kapalı ve tam otomatik doğru yaşam döngüsü (Lifecycle) şu 5 adımdan oluşur:

Adım 1: "Altın İmajın" (Golden Image) İnşası

Sıfırdan bir sanal makine (VM) oluşturulur. İşletim sistemi kurulur, en güncel güvenlik yamaları (Windows Updates) geçilir ve kurumun standart baseline yazılımları (Örn: Antivirüs ajanları, IIS, SQL Native Client) yüklenir. Otomasyonun gelecekte çalışabilmesi için bu makineye VMware Tools mutlaka kurulur. Bu makine, altyapınızın referans sunucusudur.

Adım 2: Kalıbı Mühürlemek (Clone to Template)

Hazırlanan bu kusursuz sanal makine kapatılır (Power Off) ve arayüz üzerinden `Clone to Template` işlemi başlatılır. Dikkat: Bu aşamada hiçbir "Customization" (Sysprep veya Özelleştirme) işlemi YAPILMAZ. Makine olduğu gibi kopyalanıp, salt okunur (read-only) bir kalıp olarak Template Kütüphanesine (Content Library) mühürlenir. Bu şablon artık anonim, kimliksiz ve dışarıdan müdahaleyle bozulmaz bir fabrikasyon kalıbıdır.

Adım 3: İhtiyaç Anı ve Dağıtım (Deploy from Template)

Aradan günler veya haftalar geçer. Altyapınıza yepyeni bir sunucu (veya aynı anda 50 sunucu) kurmanız gerekir. Kütüphanedeki o mühürlü kalıba sağ tıklar ve `New VM from This Template` (Bu Şablondan Yeni Sanal Makine Üret) komutunu verirsiniz. Klonun yaşayacağı donanım (Host) ve depolama (Datastore) alanı seçilir.

Adım 4: Orkestrasyon Noktası (Customization Kararı)

Dağıtım sihirbazının tam ortasında, sistem size o kritik orkestrasyon sorusunu sorar: _"Customize the operating system kutucuğunu işaretlemek ister misin?"_

İşte sihrin programlandığı yer burasıdır. Siz bu kutucuğu işaretler ve önceden hazırladığınız "Cevap Anahtarını" (Customization Specification) seçersiniz. vCenter, yeni makineye verilecek IP, Hostname ve Domain bilgilerini hafızasına alır.

Adım 5: Uyanış ve Kimlik Giyme (Sysprep Otomasyonu)

Fiziksel disk kopyalama işlemi biter ve yeni sanal makine Template'ten tamamen bağımsız, canlı bir VM olarak ilk kez çalıştırılır (Power On). Makine açılırken vCenter, VMware Tools köprüsü üzerinden o cevap anahtarını gizlice işletim sistemine enjekte eder. Windows otomatik olarak Sysprep'i çalıştırır, makine kendini yeniden başlatır ve eski genetik kimliğinden tamamen arınmış, yepyeni bir SID numarasına sahip, ağa anında hizmet vermeye hazır bir sunucu olarak hayata gözlerini açar.
