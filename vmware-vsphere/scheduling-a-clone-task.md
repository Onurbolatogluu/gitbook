---
icon: clipboard-list
---

# Scheduling a Clone Task

Bir sistem yöneticisinin en değerli varlığı zamanıdır. Sanallaştırma altyapılarında yeni sunucular hazırlamak veya klonlama gibi ağır disk okuma/yazma (I/O) işlemleri yapmak, Production ortamının mesai saatleri içinde gerçekleştirilmemesi gereken kritik operasyonlardır.

Mesai saatleri içinde başlatılan yüzlerce gigabaytlık bir klonlama işlemi, Storage ünitesini darboğaza sokarak veritabanlarına veya aktif kullanıcılara hizmet veren diğer sunuculara yavaşlama (Latency) olarak yansır. Gerçek hayatta sistem yöneticileri bu işlemler için ofiste mesaiye kalır veya hafta sonlarını feda ederlerdi.

Bu makalede, iş-yaşam dengesini korurken altyapı operasyonlarını kusursuzca otomatize etmenizi sağlayan vCenter "Scheduled Tasks" (Zamanlanmış Görevler) mimarisini ve kurumsal senaryolardaki doğru kullanımını inceleyeceğiz.

**1. Zamanlanmış Görevler (Scheduled Tasks) Nereden Yönetilir?**

vCenter arayüzünde otomasyonun gizli merkezi `Monitor > Tasks and Events > Scheduled Tasks` menüsüdür. Ancak vCenter hiyerarşisinde her nesnenin (Host, Datastore, VM) kendine has zamanlanmış görev yetenekleri vardır.

Örneğin, bir ESXi Host'u klonlayamazsınız; bu nedenle klonlama zamanlaması yapmak için doğrudan hedef Sanal Makineyi (Virtual Machine) seçip onun `Configure` sekmesine gitmeniz gerekir. Burada karşınıza sadece klonlama değil; sunucuyu gece yarısı yeniden başlatma (Restart Guest OS), kapatma (Power Off) veya başka bir Host'a taşıma (vMotion/Migrate) gibi birçok operasyonel görev çıkar.

**2. Stratejik Zamanlama Opsiyonları**

Klonlama sihirbazını zamanlanmış görev olarak başlattığınızda, klasik donanım ve lokasyon seçimi adımlarından sonra karşınıza "Scheduling Options" (Zamanlama Seçenekleri) gelir. vCenter burada yöneticilere esnek bir takvim sunar:

* Gecikmeli Başlatma (Run action after X minutes): Mesai bitimine 15 dakika kalmıştır. Çıkarken butona basıp beklemek yerine "30 dakika sonra başlat" diyerek ofisten rahatça ayrılabilirsiniz.
* İleri Bir Tarihe Kurma (Run later): Cuma günü gündüz saatlerinde tüm ayarları yapıp, görevin en ölü saat olan "Cumartesi gecesi 03:00'te" başlamasını emredebilirsiniz.
* Periyodik Tekrar (Recurring Schedule): Belirli bir görevi (Örneğin makinenin yeniden başlatılması veya kopyasının alınması) her gün, her hafta (Örn: Her Pazar) veya her ay otomatik olarak tekrar etmesi için döngüye sokabilirsiniz.

**3. Bildirim Mekanizması**

Siz evinizde hafta sonu dinlenirken, cumartesi gecesi çalışacak olan bu görevin başarılı olup olmadığını bilmek en doğal hakkınızdır. Sihirbazın son adımında bir e-posta adresi girerek vCenter'ın görev bittiğinde size rapor atmasını sağlayabilirsiniz. Ancak bunun çalışması için vCenter'ın genel ayarlarında bir SMTP (Mail) sunucusunun önceden tanımlanmış olması şarttır.



**🚨 1.**&#x20;

* Bir makineyi her hafta klonlarsanız, her seferinde o makinenin diski kadar (Örn: 100 GB) yeni bir alanı Datastore'da kalıcı olarak işgal edersiniz.
* Modern sanallaştırma mimarilerinde yedekleme işlemi vCenter'ın klonlama özelliğiyle yapılmaz. Bunun yerine Veeam, Nakivo veya Commvault gibi 3. parti yazılımlar kullanılır. Bu yazılımlar VMware'in VADP (vStorage APIs for Data Protection) mimarisini kullanarak sadece "Değişen Blokları" (CBT - Changed Block Tracking) yedekler ve alandan %90'a varan oranda tasarruf sağlar. Klonlama sadece test ortamları hazırlamak veya şablon üretmek içindir.

**🎯 2.**&#x20;

* vSphere HTML5 arayüzündeki Scheduled Tasks menüsü "Customization Specification" (Şablon/Sysprep) adımını desteklemez.
* Özetle; vCenter Scheduled Tasks menüsü, mesai dışı operasyonları yönetmek ve yöneticinin yükünü hafifletmek için harika bir araçtır. Ancak bu aracın sınırlarını bilmek, yedekleme stratejisiyle karıştırmamak gerekmektedir.

{% hint style="info" %}
Zamanlanmış görevlerde (Scheduled Tasks) yer alan "After vCenter startup" seçeneği, görevi belirli bir saate değil, doğrudan vCenter servisinin yeniden başlama olayına (event) bağlar. Sunucu güncellemesi veya elektrik kesintisi gibi durumlarda, vCenter ayağa kalktığı an bu görev otomatik olarak devreye girer. Genellikle kritik sunucuların açılış (Power On) sırasını vCenter'ın hazır olma durumuna bağlamak için kullanılır.

Hemen yanındaki "Delay" (Gecikme) kutucuğu ise mimari bir güvenlik sübabıdır. vCenter servisleri başlasa bile, arka planda ortamdaki tüm ESXi hostlarla yeniden iletişim kurulması ve disklerin taranması zaman alır. Burayı "0" bırakmak görevin hata vermesine yol açabilir. Bunun yerine 5 veya 10 dakika gibi bir gecikme süresi girmek, vCenter'ın ortamı tamamen tarayıp stabilize olmasını sağlar ve görevin kusursuzca çalışmasını garanti altına alır.
{% endhint %}
