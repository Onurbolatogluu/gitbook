---
icon: power-off
---

# Virtual Machine Power User

vCenter Server üzerinde kullanıcıları oluşturup Active Directory veya `vsphere.local` veritabanı üzerinden yetkilendirmeyi tamamladıktan sonra, işin en kritik aşaması başlar: Test. Sistem yöneticileri genellikle VMware'in sunduğu Default şablon rolleri kullanmayı tercih eder. Bu rollerden en popüler olanı, kullanıcılara sanal makineler üzerinde geniş yetkiler tanıdığı varsayılan "Virtual Machine Power User" rolüdür. Peki ama bu rol, isminin vaat ettiği kadar güçlü müdür?

Bu makalede, `vsphere.local` üzerinden bu rolle yetkilendirilmiş bir kullanıcının arayüzdeki hareket alanını inceleyecek, hangi duvarlara çarptığını ve sanallaştırma mimarisindeki "Çapraz Yetki Çakışması" adı verilen o meşhur tuzağı analiz edeceğiz.

**1. "Power User" Ne Yapabilir?**

Bu rolle yetkilendirilmiş bir kullanıcıyla sisteme giriş yapıldığında, kullanıcının gücünün doğrudan ve sadece mevcut sanal makineler üzerinde yoğunlaştığı görülür. Bir sanal makineye sağ tıklandığında şu temel operasyonlar sorunsuzca gerçekleştirilebilir:

* Güç Operasyonları (Power Ops): Makineyi başlatma (Power On), güvenli kapatma veya yeniden başlatma (Restart Guest).
* Snapshots: Kritik işlemler öncesi Snapshot alma veya mevcut bir yedeğe geri dönme.
* Konsol Erişimi: İşletim sisteminin arayüzüne doğrudan bağlanabilme.
* Donanım Düzenleme (Kısmi): Sanal makinenin ismini değiştirme, mevcut RAM miktarını artırma veya işlemci (CPU) çekirdek sayısını düzenleme.

Kullanıcı, sanal makinenin sınırları içinde (Compute) kendi krallığını ilan etmiş gibi görünür. Ancak donanım ekleme aşamasında işin rengi değişir.

**2. Yanılsama ve Kısıtlamalar: Gri Menüler (Grayed-out Options)**

"İleri düzey" bir kullanıcı olmasına rağmen, bu rolün altyapısal bir gücü yoktur. Kullanıcı vCenter nesnesine veya ESXi Host'lara tıkladığında, Context menülerindeki birçok hayati seçeneğin pasif (Gri) durumda olduğunu görür.

* Sıfırdan yepyeni bir sanal makine oluşturamaz.
* Sistem ağlarını (vSwitch / Port Groups) yönetemez.
* Kaynak Havuzu (Resource Pool) yaratamaz.
* Bir makineyi "Klonlayamaz" (Clone) veya bir makineyi vMotion ile başka bir Host'a "Taşıyamaz" (Migrate).

Çünkü bu işlemler sadece makineyi değil, ortamdaki donanım kümesini (Cluster) ilgilendiren üst düzey mimari operasyonlardır.

**3. Datastore Tuzağı**

Kullanıcı, yetkisi dahilinde olan bir makineye sağ tıklar, "Edit Settings" (Ayarları Düzenle) der ve makineye yeni bir Sanal Disk (Hard Drive) eklemek ister. Boyutu (Örn: 10 GB) belirler, sihirbazı onaylar ve sistem karşısına şu kırmızı hatayı fırlatır:

> _"Permission to perform this operation was denied. You do not hold privilege Datastore allocate space..." (Bu işlemi gerçekleştirme izni reddedildi. Datastore üzerinde alan ayırma ayrıcalığına sahip değilsiniz.)_

Neden Başarısız Oldu?

Kullanıcının sanal makinenin donanım ayarlarını değiştirme yetkisi vardır; makinenin adını başarıyla değiştirmiştir. Ancak yeni bir disk eklemek demek, arka planda Veri Deposuna (Datastore) gidip fiziksel depolama alanından gigabaytlarca yer işgal etmek (Provisioning / Allocate Space) demektir.

"VM Power User" rolü, depolama alanını yönetme yetkisini içermez. İşlem, sanal makine sınırlarından çıkıp Datastore sınırlarına girdiği an, vCenter'ın yüksek güvenlikli bariyerine çarpar ve reddedilir.

> 💡 Uzman Analizi: RBAC (Role-Based Access Control) Mimarisi İçin Çıkarımlar
>
> Yukarıdaki disk ekleme hatası, VMware'in Roller ve İzinler mimarisinin ne kadar keskin ve kuralcı çalıştığının en güzel kanıtıdır.
>
> Bir işlemi otomatize ederken veya bir ekibe yetki verirken, operasyonun arka planda hangi altyapı bileşenlerine dokunduğunu (Compute, Network, Storage) çok iyi hesaplamak gerekir. Eğer bu kullanıcıların disk ekleyebilmesini istiyorsanız, onlara salt bir "VM Power User" rolü vermek yetmeyecek; makalelerimizde daha önce bahsettiğimiz gibi, `Datastore > Allocate Space` yetkisini barındıran yepyeni bir Özel Rol (Custom Role) inşa edip bu kullanıcılara atamanız gerekecektir.
