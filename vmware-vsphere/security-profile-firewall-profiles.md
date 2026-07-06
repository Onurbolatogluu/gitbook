---
icon: id-card
---

# Security Profile: Firewall Profiles

Önceki makalemizde ESXi üzerindeki Security Profile yapısının neden sanallaştırma mimarisinin "İlk Savunma Hattı" olduğunu tartışmış ve "Defence in Depth" prensibini incelemiştik. ESXi işletim sisteminin (VMkernel) kendi güvenlik duvarı, katı bir "Default Deny" mantığıyla çalışır.

Peki, bu kalkanı nasıl yöneteceğiz? İhtiyacımız olan servislerin çalışması için hangi kapıları, kime açacağız? Bu makalede, vCenter arayüzü üzerinden ESXi Host'lara özel Firewall kurallarının nasıl yapılandırıldığını, Inbound/Outbound mantığını ve IP Tabanlı Kısıtlama mimarisini detaylıca inceleyeceğiz.

**1.Firewall Kuralları Neden Host Bazlıdır?**

Bir vCenter ortamında 50 adet ESXi sunucunuz olduğunu varsayalım. Doğal bir içgüdü olarak, _"Firewall kurallarını vCenter'dan bir kere yazayım, 50 sunucuya aynı anda uygulansın"_ diyebilirsiniz. Ancak VMware mimarisi (özel eklentiler veya Host Profiles kullanılmadığı sürece) varsayılan arayüzde buna izin vermez.

Security Profile yapılandırması tamamen Host'a (Donanıma) özgüdür.

Inventory'den `Host-10` cihazına tıklayıp `Configure > Security Profile` sekmesine girdiğinizde yaptığınız değişiklik, sadece o cihazı bağlar.

_Neden?_ Çünkü her Host'un ortamdaki görevi farklı olabilir. Bir Host sadece test makinelerini barındırırken, diğeri dışarıya açık DMZ bölgesindeki web sunucularını barındırıyor olabilir. Bu nedenle, her sunucunun güvenlik yeleği kendi risk profiline göre terzi usulü dikilmelidir.

**2. Çift Yönlü Trafik Yönetimi: Inbound vs. Outbound**

`Security Profile` altındaki Firewall sekmesinde `Edit`  dediğinizde, devasa bir kural listesi ile karşılaşırsınız. Burada her kuralın iki farklı yönü vardır:

* Incoming Connections: Dışarıdaki bir cihazın, ESXi sunucunuza doğru başlattığı bağlantılardır.
  * _Örnek (SSH Server):_ Dışarıdan bir yöneticinin PuTTY kullanarak ESXi'ın 22 numaralı portuna bağlanma isteğidir. Bu kutucuk işaretliyse, ESXi kapıyı açar.
* Outgoing Connections: ESXi sunucusunun kendisinin dışarıdaki bir hedefe doğru başlattığı bağlantılardır.
  * _Örnek (NTP Client):_ ESXi sunucusunun saatini eşitlemek için internetteki bir zaman sunucusuna (`time.windows.com`) UDP 123 portundan gitme isteğidir.
  * _Örnek (SSH Client):_ Çok az bilinen ama çok kritik bir özelliktir. Eğer ESXi'ın konsolundayken, _oradan başka bir sunucuya_ SSH çekmek isterseniz, giden (Outbound) trafik olan `SSH Client` kutucuğunu işaretlemeniz gerekir.

Sistem yöneticisinin kuralı şudur: "Bir servisin sadece size gelmesini mi istiyorsunuz, yoksa sunucunuzun o servise gitmesini mi?" Kutucukları buna göre işaretlemelisiniz. İhtiyacınız olmayan her şey (Örneğin IPv6 veya kullanılmayan FTP Client) mutlaka devre dışı kalmalıdır.

**3. IP Tabanlı Kısıtlama**

Listeden bir kuralı işaretleyip aktif ettiniz. Varsayılan durumda, Firewall arayüzünün sağ alt köşesindeki "Allow connections from any IP address" (Her IP'den gelen bağlantıya izin ver) seçeneği aktiftir.

Bu, bir kapıyı kilitli tutmaktan vazgeçip, sokağa tamamen ardına kadar açmak demektir. Kötü niyetli bir yazılım iç ağa sızdığında, bu açık SSH portunu tarayıp saldırı başlatabilir.

Kurumsal güvenliğin (ve ISO 27001 / PCI-DSS gibi standartların) standardı, o portu açtıktan hemen sonra "Only allow connections from the following networks" (Sadece şu ağlardan gelen bağlantılara izin ver) seçeneğini işaretlemektir.

* Bu alana bir virgül bırakarak sadece Sistem Yöneticilerinin bulunduğu VLAN bloğunu (Örn: `192.168.50.0/24`) veya sadece IT yöneticisinin spesifik dizüstü bilgisayar IP'sini (`192.168.50.15`) yazarsınız.
* Sonuç: Port açık görünmesine rağmen, yetkisiz bir IP bloğundan (`192.168.100.x` gibi bir kullanıcı bilgisayarından) gelen istekler, Firewall tarafından sessizce düşürülür ve sunucuya ulaşamaz.

> 💡 vCenter Kurulum Sonrası (Post-Deployment) Port Temizliği
>
> Yeni bir ESXi Host kurup onu vCenter'a eklediğinizde, kurulum sihirbazları arka planda haberleşmeyi sağlamak için birçok portu (Örn: vSphere High Availability (HA) agentları, Fault Tolerance loglama trafiği) otomatik olarak aktif eder.
>
> Kurulumlar tamamen bittikten sonra Host'un Security Profile ekranına mutlaka geri dönmelidir. Kullanılmayan eski servisler (Örneğin ortamda vSAN yoksa vSAN'a ait portlar) gözden geçirilmeli ve açık unutulmuş gereksiz kapılar kapatılmalıdır.
