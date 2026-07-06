---
icon: lock-a
---

# Enabling or Disabling a Service in the Security Profile

Sanallaştırma altyapılarında güvenlik mimarisini tasarlarken genellikle sadece Firewall kurallarına odaklanılır. Ancak unutulmaması gereken çok temel bir ağ kuralı vardır: Firewall sadece kapıyı açar; içeri girdiğinizde sizi karşılayacak olan şey ise o kapının arkasında çalışan "Servis"tir.

Eğer Port açık ama arkasındaki servis kapalıysa, bağlantı yine başarısız olacaktır. Önceki makalelerimizde ESXi Security Profile üzerinden ağ portlarını nasıl yöneteceğimizi detaylandırmıştık. Bu makalede ise aynı profilin diğer yarısı olan "Services" sekmesini, kritik sistem servislerinin nasıl yönetildiğini ve servis otomasyon politikalarını inceleyeceğiz.

**1. DCUI**

ESXi sunucularında işletim sistemi ayağa kalktığında, fiziksel monitörde (veya iLO/iDRAC/XCC gibi out-of-band yönetim ekranlarında) karşımıza çıkan o meşhur sarı-siyah arayüzün teknik adı DCUI'dır.

Sistem yöneticileri genellikle bu ekranın sunucunun donanımına ait ayrılmaz bir parça olduğunu düşünür. Oysa DCUI, tıpkı diğer yazılımlar gibi arka planda çalışan ve istendiğinde durdurulabilen bir servistir.

Eğer ortamınızda çok yüksek bir fiziksel güvenlik standardı varsa, Security Profile üzerinden DCUI servisini "Stop" (Durdur) konumuna getirebilirsiniz. Bu yapıldığında:

* Sunucunun başındaki monitöre klavye takıp "F2" tuşuna basan bir kişi, doğru root şifresini bilse dahi "Authentication Denied" (Kimlik Doğrulama Reddedildi) hatası alır.
* Sistem sadece vCenter veya web arayüzü üzerinden uzaktan yönetilebilir hale gelir. Fiziksel konsol tamamen sağırlaştırılır.

**2. ESXi Shell**

Bir diğer kritik servis ise ESXi Shell'dir. DCUI arayüzünde "Troubleshooting Mode Options" altından veya web arayüzünden aktif edilebilen bu servis, sunucunun derinliklerine inip doğrudan Linux benzeri komutlar çalıştırmanızı sağlar.

Varsayılan güvenlik politikaları gereği, ESXi Shell kapalı gelir. Gelişmiş bir sorun giderme operasyonu yapılmayacağı sürece, bu servisin kesinlikle durdurulmuş (Stopped) konumda kalması gerekir. Shell servisinin gereksiz yere açık bırakılması, içeriden gelebilecek yanal hareket saldırılarına karşı sunucunuzu tamamen savunmasız bırakır.

**3. Startup Policies**

Bir servisi manuel olarak başlatmak (Start) veya durdurmak anlık bir çözümdür. Asıl mühendislik, sunucu yeniden başlatıldığında bu servislerin nasıl davranacağını belirleyen Startup Policies yapılandırmasıdır. ESXi, yöneticilere üç farklı otomasyon seçeneği sunar:

* Start and stop with host: Bu seçenek, sürekli açık kalması gereken kritik altyapı servisleri (Örn: Active Directory entegrasyon servisleri veya NTP Daemon) için kullanılır. ESXi işletim sistemi yüklendiği an, bu servisler hiçbir insan müdahalesi beklemeden arka planda çalışmaya başlar.
* Start and stop manually: Kontrolün tamamen sistem yöneticisinde olduğu moddur. Sunucu yeniden başlasa bile bu servis kapalı kalır. Sadece siz arayüze girip bizzat "Start" butonuna bastığınızda çalışır. ESXi Shell veya SSH gibi yüksek riskli yönetim servisleri için en ideal ve güvenli seçenektir.
* Start and stop with port usage: VMware mühendisliğinin en akıllıca tasarlanmış seçeneklerinden biridir. Bu mod seçildiğinde servis "Uyku" durumundadır. Eğer Firewall üzerinden o servise ait olan port (örneğin port 22) açılırsa ve içeri bir istek gelirse, sistem o an servisi otomatik olarak uyandırır. Kullanıcının işi bitip ağ trafiği kesildiğinde veya Firewall portu kapatıldığında, servis de kendi kendini otomatik olarak durdurur. Sistem kaynaklarını verimli kullanmak için kusursuz bir tetikleme mekanizmasıdır.

> 💡 Sorun Giderme ve Servis Bağımlılıkları
>
> "Sunucuya dışarıdan ping atamıyorum" veya "NTP sunucusu ile senkronizasyon yapamıyorum" gibi sorunlarla karşılaştığınızda, sadece Firewall portlarını kontrol etmek vakit kaybına yol açabilir.
>
> Bir kural olarak; ağ erişimiyle ilgili bir problem çözülemiyorsa, arıza tespiti daima iki bacaklı yapılmalıdır. Önce Security Profile'dan Firewall sekmesine bakılır (Port açık mı?), hemen ardından Services sekmesine geçilir (Arkada dinleyen servis/daemon çalışıyor mu?). Ağ trafiği ve servis durumu bir elmanın iki yarısı gibi birlikte hareket etmediği sürece, konfigürasyonunuz her zaman eksik kalacaktır.
