---
icon: grid-4
---

# Host Image Profile and Host Encryption Level

Sanallaştırma altyapılarında Security Profile denildiğinde akla ilk olarak Firewall kuralları ve sistem servisleri (SSH, DCUI) gelir. Ancak ESXi hypervisor mimarisi, kaputun altında sistemin çekirdeğini ve veriyi korumaya yönelik çok daha spesifik iki ileri düzey güvenlik ayarı barındırır.

Host Image Profile Acceptance Level ve Host Encryption Mode. Bu makalede, bu iki özelliğin teknik altyapısını, sanallaştırma güvenliğindeki yerini ve neden sadece Host seviyesinde yapılandırılabildiklerini inceleyeceğiz.

**1. Host Image Profile Acceptance Level**

ESXi sunucularına kurulan yamalar (patch), donanım sürücüleri veya eklentiler sisteme VIB isimli paketler halinde yüklenir. Kaynağı doğrulanmamış her VIB dosyasını sunucuya kurmak, sistemin çökmesine veya güvenlik açıklarına yol açabilir.

İşte Host Image Profile Acceptance Level, ESXi sunucusuna yüklenecek olan bu yazılım paketlerinin "Güvenilirlik Seviyesini" belirleyen filtredir. Sistem yöneticisi bu seviyeyi belirlediğinde, belirlenen seviyenin altındaki hiçbir VIB paketi sunucuya kurulamaz ve kurulum işlemi anında reddedilir.

VMware ekosisteminde bu güvenilirlik hiyerarşisi şu şekilde sıralanır:

* VMwareCertified: En katı seviyedir. Sadece VMware tarafından doğrudan test edilmiş ve onaylanmış kodlar kabul edilir.
* VMwareAccepted: VMware iş ortakları tarafından geliştirilmiş, VMware'in testlerinden geçmiş yazılımlardır.
* PartnerSupported: Üçüncü parti üreticilerin (HPE, Dell vb.) kendi testlerinden geçirdiği yazılımlardır.
* CommunitySupported: Topluluk tarafından geliştirilen, hiçbir resmi garantisi olmayan esnek seviyedir (Örn: Resmi olmayan veya modifiye edilmiş bazı ağ kartı sürücüleri).

**2. Host Encryption Mode**

Veri güvenliğinde ağ saldırıları kadar kritik olan bir diğer risk donanımın izinsiz taşınmasıdır. Bir ESXi sunucusunun içindeki diskler fiziksel olarak sökülüp başka bir sisteme takıldığında, eğer veriler açık  haldeyse, içindeki tüm Virtual Machine verileri saniyeler içinde kopyalanabilir.

Host Encryption Mode, bu fiziksel tehdide karşı geliştirilmiş bir şifreleme mekanizmasıdır. Varsayılan olarak kapalı gelir. Aktif edildiğinde, hypervisor seviyesinde bir şifreleme başlatılır ve sunucu donanım bazında kilitlenir.

Ekstra Mimari Detay: Host Encryption Mode aktif edildiğinde, vCenter bir KMS sunucusundan şifreleme anahtarları çeker ve ESXi sunucusuna gönderir. Şifrelenmiş bir ESXi sunucusu fiziksel olarak çalınırsa, hırsızlar kurum ağındaki KMS sunucusuna erişemeyeceği için içindeki veriler tamamen okunamaz ve anlamsız bir veri yığını olarak kalır.

**3. Security Profile Neden Sadece Host Seviyesindedir?**

_Neden?_ Çünkü Security Profile altındaki Firewall, Services, Image Profile ve Encryption konseptleri doğrudan ESXi işletim sisteminin (VMkernel) çekirdeğine aittir. vCenter, ortamı merkezden yöneten bir orkestrasyon katmanıdır; ancak asıl güvenlik mekanizmaları ve şifreleme işlemleri doğrudan fiziksel sunucunun kendi işletim sistemi tarafından ele alınmak zorundadır. Bu yüzden ortamınızda 50 adet Host varsa, her birinin Security Profile ayarları; kendi görev tanımına, barındırdığı Virtual Machine hassasiyetine ve donanım konumuna göre bireysel olarak tasarlanmalıdır.
