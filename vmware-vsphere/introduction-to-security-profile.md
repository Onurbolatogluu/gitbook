---
icon: paintbrush-fine
---

# introduction to Security Profile

Herhangi bir sunucuya, fiziksel bir cihaza veya bir Virtual Machine içerisine işletim sistemi kurduğunuzda, karşınıza çıkan ilk güvenlik adımlarından biri ağ trafiğini denetlemektir. Windows'ta, Linux'ta veya MacOS'ta olduğu gibi, kurumsal sanallaştırma platformlarının kalbi olan VMware ESXi da kendi iç güvenliğini sağlamak zorundadır.

Bu makalede, ESXi üzerindeki Security Profile kavramının ne anlama geldiğini, neden kritik olduğunu ve kurumsal veri merkezlerinde bu mimarinin nasıl konumlandırılması gerektiğini inceleyeceğiz.

**1. Security Profile Nedir ve Neden Hayatidir?**

Standart bir fiziksel ağda, dışarıdan gelen tehditleri durdurmak için donanımsal Firewall cihazları (Örn: Palo Alto, Fortinet) kullanılır. Ancak veri merkezinizin iç ağına sızmış bir saldırgan veya hatalı yapılandırılmış bir iç ağ cihazı, doğrudan ESXi sunucunuzun yönetim arayüzüne saldırabilir.

İşte bu noktada ESXi'ın kendi işletim sistemi (VMkernel) seviyesinde çalışan yerleşik Firewall mimarisi devreye girer. vCenter veya ESXi arayüzünde Security Profile sekmesi altından yönetilen bu sistem, temel olarak iki şeyi kontrol eder:

* Inbound / Outbound Traffic: Host'a dışarıdan hangi portlardan erişilebileceği ve Host'un dışarıya hangi portlardan çıkabileceği.
* Services: Ağ portlarıyla doğrudan ilişkili olan sistem servislerinin (Örn: SSH, NTP, iSCSI) çalışma durumları.

**2. ESXi Firewall'un Altın Kuralı: "Default Deny"**

Güvenlik mimarilerinde iki temel yaklaşım vardır: Ya her şeye izin verir, tehlikeli olanları engellersiniz  ya da her şeyi yasaklar, sadece ihtiyaç duyulanlara izin verirsiniz.

ESXi işletim sistemi, katı bir Default Deny mantığıyla gelir. Siz bir ESXi Host kurduğunuzda, sadece vSphere Client ile bağlanabilmeniz için gereken `443` (HTTPS) veya ping atabilmeniz için ICMP gibi çok temel birkaç port açıktır. Bunun dışındaki tüm Inbound ve Outbound bağlantılar Security Profile tarafından bloklanır.

Örneğin, sunucunuzun saatini eşitlemek için bir NTP (Network Time Protocol) sunucusuna bağlanmasını istiyorsanız, Security Profile arayüzüne girip `NTP Client` kuralını manuel olarak aktif etmeniz gerekir. Aksi takdirde ESXi, dışarıdaki zaman sunucusuna doğru Outbound trafik bile başlatamaz.

**3. Security Profile vs. Kurumsal Firewall**

Sahada sistem yöneticilerinin sıkça sorduğu bir soru vardır: _"Ön tarafta zaten devasa bir fiziksel Firewall cihazımız var, ESXi içindeki Security Profile ayarlarıyla neden uğraşalım ki? Hepsini açıp geçelim."_

Bu, siber güvenlik dünyasında yapılabilecek en büyük mimari hatalardan biridir. Bu yaklaşıma karşı "Defense in Depth" prensibiyle yanıt vermek gerekir:

1. Fiziksel Firewall cihazınız, veri merkezinizi dış dünyaya karşı korur.
2. ESXi üzerindeki Security Profile ise, veri merkezinizin içindeki diğer sunuculardan, enfekte olmuş sanal makinelerden veya kötü niyetli iç kullanıcılardan gelecek saldırılara karşı Host'u korur.

Bir saldırgan iç ağa sızıp vCenter sunucusunu atlatsa bile, ESXi Host'un kendi Security Profile kalkanını aşmak zorunda kalacaktır.

**4. Kilit Operasyonlar: SSH ve Lockdown Mode İlişkisi**

Security Profile arayüzünde en çok manipüle edilen kural şüphesiz SSH erişimidir. Kurumsal ortamlarda sorun giderme işlemleri için SSH portu (`TCP 22`) zaman zaman açılır. Ancak bir Host üzerinde SSH servisi başlatılıp Inbound kuralı aktif edildiğinde, ESXi arayüzü yöneticileri sürekli sarı bir uyarı bandıyla rahatsız eder: _"SSH for the host has been enabled."_

Sistem bunu bilerek yapar; çünkü Security Profile size şu mesajı vermektedir: _"Güvenlik kapılarından birini açtın, işin bittiğinde onu kapatmayı unutma."_ Daha da ileri seviye güvenlik (Örneğin PCI-DSS uyumluluğu) gerektiren ortamlarda, Security Profile ile birlikte Host'un yerel erişimini tamamen kesen ve sadece vCenter üzerinden iletişime izin veren Lockdown Mode mimarisi birlikte kullanılır.

Özetle; ESXi üzerindeki Security Profile, basit bir "Aç/Kapat" menüsü değildir. Sunucunuzun dış dünyayla olan iletişim sınırlarını çizen, servislerin kaderini belirleyen ve sanallaştırma altyapınızın yönetim kısmını ayakta tutan en kritik savunma kalkanıdır. Sonraki operasyonlarda bu kalkanın kurallarını nasıl eğip bükeceğimizi ve kendi ihtiyaçlarımıza göre nasıl şekillendireceğimizi incelemek, başarılı bir sistem yönetiminin anahtarıdır.
