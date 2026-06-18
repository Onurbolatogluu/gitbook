---
icon: circle-play
---

# Virtual Machine Startup

Bir sistem yöneticisinin en büyük kabuslarından biri, veri merkezinde yaşanan genel bir elektrik kesintisi veya donanımsal bir arıza sonrası tüm sunucuların aniden kapanmasıdır. Enerji geri geldiğinde fiziksel sunucular (ESXi Host) ayağa kalkar, ancak içlerindeki onlarca veya yüzlerce sanal makine varsayılan olarak kapalı (Powered Off) durumda bekler.

Acil bir durumda 20-30 kritik sunucuyu manuel olarak tek tek açmak hem büyük bir zaman kaybıdır hem de insan hatasına inanılmaz derecede açıktır. Çünkü kurumsal mimarilerde sunucular rastgele açılamaz; aralarında kırılmaz bir bağımlılık zinciri vardır.

Bu makalede, felaket anlarında sistemin kendi kendini doğru bir mantıkla yeniden inşa etmesini sağlayan VMware'in "VM Startup/Shutdown" (Sanal Makine Açılış/Kapanış Sıralaması) özelliğini ve kurumsal altyapılardaki yapılandırma stratejilerini inceleyeceğiz.

**1. Temel Sorun: Servis Bağımlılığı (Neden Sıralama Şarttır?)**

Bir ESXi host üzerindeki tüm makineleri aynı anda aç emri verirseniz, sistemde büyük bir kaos yaşanır.&#x20;

Eğer Microsoft Exchange veya bir SQL Veritabanı sunucusu, Active Directory'den _önce_ açılırsa, ağda kimliğini doğrulayacak merkezi bulamayacağı için servisleri "Error" (Hata) veya "Stop" durumuna geçer. Active Directory 3 dakika sonra açılsa bile, Exchange sunucusu çoktan hata verip kilitlenmiş olacaktır.

İşte bu yüzden altın kural şudur: Önce altyapı servisleri (DNS, AD, Firewall) açılır, sonra veritabanları (SQL/Oracle) açılır, en son uygulama ve web sunucuları ayağa kaldırılır.

**2. Kural Setini Oluşturmak: 3 Farklı Kategori**

ESXi Host menüsünde `Configure > Virtual Machines > VM Startup/Shutdown` sekmesine geldiğinizde bu özellik varsayılan olarak kapalıdır (Disabled). Aktif ettiğinizde vCenter size sanal makineleri sürükleyip bırakabileceğiniz (Move Up / Move Down) 3 farklı havuz sunar:

* 1\. Automatic Startup (Sıralı ve Gecikmeli Açılış): Bağımlılığı olan kritik sunucuların listesidir. Listenin 1 numarasına Domain Controller konur, 2 numarasına Exchange konur, 3 numarasına Veritabanı konur. Host ayağa kalktığında bu listeyi yukarıdan aşağıya doğru sırayla işlemeye başlar.
* 2\. Any Order (Sırasız Otomatik Açılış): Sıralaması veya başka bir sunucuya bağımlılığı olmayan (örneğin bağımsız bir test sunucusu veya izole bir Linux web makinesi) makineler buraya konur. Host, "Automatic Startup" listesindeki kritik görevleri bitirdikten sonra bu havuza geçer ve buradaki makineleri sırasız bir şekilde topluca açar.
* 3\. Manual Startup (Elle Açılış): Sadece sistem yöneticisinin inisiyatifiyle açılması gereken, kapalı kalması tercih edilen pasif veya yedek makineler bu havuzda bırakılır. Host bu makinelere kesinlikle dokunmaz.

**3. Zamanı Yönetmek: "Startup Delay" vs. "VMware Tools"**

Makineleri sıraya dizmek tek başına yetmez. Domain Controller makinesine "Açıl" (Power On) emri verildikten sadece 5 saniye sonra Exchange makinesine açıl emri verilirse yine sistem çöker. Çünkü Domain Controller'ın Windows'u yüklemesi ve servislerini başlatması birkaç dakika alacaktır. Burada iki farklı gecikme (Delay) mimarisi kullanılır:

* Süre Bazlı Gecikme (Startup Delay): Host 1. makineyi açar, kronometreyi başlatır, tam 3 dakika bekler ve ardından 2. makineye "Power On" emri gönderir.
* Akıllı Tetikleyici (Continue immediately if VMware Tools starts): Bu, sistemin en zeki ayarıdır. Bazen 1. makine 3 dakika değil, 40 saniyede tamamen açılabilir. Geriye kalan 2 dakika 20 saniye boşuna kaybedilmiş olur. Bu kutucuğu işaretlerseniz ESXi host kronometreyi beklemez; makinenin içindeki VMware Tools servisi "Çalışıyor" (Running) durumuna geçtiği an, o işletim sisteminin tamamen ayağa kalktığını anlar ve beklemeden hemen 2. makineyi açmaya geçer. Bu, kurtarma (Recovery) süresini inanılmaz derecede hızlandırır.

#### 💡

Eğer sunucularınız bir vCenter altında HA (High Availability) Cluster içinde çalışıyorsa, bu menüyü KESİNLİKLE KULLANAMAZSINIZ ve vCenter bunu bilerek kilitler. Neden mi? Çünkü HA ortamında bir host çökerse, makineler o host'ta değil, ortamdaki _diğer_ sağlam hostlar üzerinde otomatik olarak yeniden başlatılır. Eğer siz lokal bir host üzerinde bu "Startup Sıralamasını" yapmaya kalkarsanız, vCenter'ın kendi DRS ve HA zekasıyla devasa bir çatışmaya girersiniz.

Özetle Kural Şudur:

1. Eğer tek tabanca çalışan, bağımsız (Standalone) bir ESXi Host'unuz varsa: Makinelerin güvenliği için VM Startup/Shutdown özelliğini kullanın.
2. Eğer devasa bir HA Cluster ortamınız varsa: Bu menüyü tamamen unutun. Sıralama ve bağımlılık kurallarını vCenter üzerinden "VM/Host Rules" (DRS Kuralları) yazarak merkezi olarak yönetin.
