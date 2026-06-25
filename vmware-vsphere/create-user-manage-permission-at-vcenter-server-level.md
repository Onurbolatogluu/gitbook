---
icon: layer-plus
---

# Create User, Manage Permission at vCenter Server Level

Önceki makalelerimizde, ESXi sunucularında yerel (local) hesaplar oluşturarak yetki dağıtımı yapmanın sınırlarını ve özel rollerin (Custom Roles) mimari yapısını detaylıca incelemiştik. Bağımsız (Stand-alone) sunucularda yerel hesap yönetimi hayat kurtarsa da, işin içine onlarca ESXi sunucusu, devasa veri depoları (Datastores) ve binlerce sanal makine girdiğinde oyunun kuralları tamamen değişir.

Tekil bir ESXi host üzerinde oluşturduğunuz kullanıcı, yalnızca o donanımın sınırları içinde yaşar. Ancak sanallaştırma mimarisinin beyni olan vCenter Server seviyesinde bir yetkilendirme yaptığınızda, bu yetki tüm veri merkezine (Datacenter), Clusterlara ve ağ kaynaklarına hükmedebilir. Bu makalede, vCenter mimarisindeki merkezi yetki dağıtım mekanizmalarını ve kullanıcıların çekildiği "Kimlik Kaynaklarını" (Identity Sources) inceleyeceğiz.

**1. Yetkinin Merkezileşmesi: vCenter "Permissions" Sekmesi**

vCenter arayüzünde en üst seviyeden (Root veya vCenter nesnesi üzerinden) `Permissions` (İzinler) sekmesine girdiğinizde, ortamı yöneten yöneticilerin ve servis hesaplarının genel bir haritasını görürsünüz. Sistemi ilk kurduğunuzda burada varsayılan olarak `administrator@vsphere.local` hesabını ve VMware'in kendi iç süreçleri için oluşturduğu bazı sistem hesaplarını bulursunuz.

Sisteme yeni bir uzman eklemek ve ona bir rol atamak için "Add Permission" (İzin Ekle) sihirbazını başlattığınızda, vCenter size sıfırdan bir isim ve şifre yazmanızı istemez. Bunun yerine, size "Bu kullanıcıyı hangi veri tabanından (Domain) çekip getireyim?" diye sorar. İşte VMware mimarisinin kurumsal dünyayla köprü kurduğu yer burasıdır.

**2. Kimlik Kaynakları (Identity Sources): Kullanıcılar Nereden Gelir?**

vCenter'da yeni bir izin tanımlarken açılır listede (Domain Dropdown) temelde üç farklı kimlik kaynağı ile karşılaşırsınız:

* 1\. vSphere.local (vCenter SSO Veritabanı): vCenter'ın kurulumuyla birlikte gelen, tamamen kendi içinde barındırdığı bağımsız ve yerel kimlik doğrulama veritabanıdır (Single Sign-On). Ortamda hiçbir Microsoft Active Directory sunucusu olmasa bile, vCenter bu domain üzerinden kendi kullanıcılarını oluşturmanıza ve yönetmenize olanak tanır.
* 2\. Yerel İşletim Sistemi Kullanıcıları (Local OS Users): Eğer vCenter Server altyapınız bir Windows Server üzerine kuruluysa (Özellikle eski mimarilerde), açılır listede sunucunun kendi ismini (Örn: `WIN-CENTER`) görürsünüz. Bu seçenek, doğrudan Windows Server'ın "Local Users and Groups" menüsünde açılmış olan standart Windows kullanıcılarına vCenter yetkisi verebilmenizi sağlar.
* 3\. Active Directory / LDAP (Kurumsal Dizin): Kurumsal sanallaştırma mimarisinin "Altın Standardı" budur. Altyapınızdaki Microsoft Active Directory domain'i (Örn: `kurum.local`) vCenter'a entegre edildiğinde, tüm şirket çalışanlarınızın veya IT personelinizin hesapları doğrudan bu listede belirir.

**3. Active Directory Entegrasyonunun Önemi ve Sorun Giderimi**

Bir sanallaştırma altyapısını tasarlarken veya yönetirken, sistem yöneticilerinin `vsphere.local` altında sürekli yeni kullanıcılar (`ahmet@vsphere.local`, `mehmet@vsphere.local`) açması kesinlikle önerilen bir pratik (Best Practice) değildir. Bu, şifre politikalarının zayıflamasına ve personel işten ayrıldığında hesap kapatma süreçlerinin unutulmasına yol açar.

Gerçek bir veri merkezinde kullanıcılar daima Active Directory (AD) üzerinden çekilir. Ancak bazen "Add Permission" ekranındaki domain listesini açtığınızda kurumunuzun AD domain ismini göremeyebilirsiniz.

> 💡 Uzman İpucu: AD Domain'i Neden Listede Görünmez?
>
> Eğer vCenter Server, işletim sistemi seviyesinde domaine (Domain Controller'a) alınmış olmasına rağmen vCenter yetki ekranında AD domain'ini göremiyorsanız, bu genellikle vCenter SSO (Single Sign-On) yapılandırmasındaki bir eksiklikten kaynaklanır.
>
> Sadece sunucuyu domaine almak yetmez. vCenter'ın yönetim (Administration) paneline girip, `Single Sign-On > Configuration > Identity Sources` sekmesinden Active Directory'nizi bir "Kimlik Kaynağı" olarak manuel olarak eklemeniz (Add Identity Source) gerekir. Aksi takdirde vCenter, AD veritabanındaki kullanıcıları okuyamaz. Eğer yapılandırma doğru yapılmasına rağmen liste hala boşsa, vCenter ile Domain Controller arasındaki DNS çözümleme (Resolution) sorunları veya Zaman Senkronizasyonu (NTP) farklılıkları incelenmelidir.

Özetle; vCenter Server üzerinde yetkilendirme yapmak, doğru kullanıcıyı doğru kaynaktan bulmakla başlar. Active Directory entegrasyonu kusursuz bir şekilde kurgulandığında, BT operasyon merkezinizdeki tüm ekip üyeleri kendi kurumsal şifreleriyle vCenter'a giriş yapabilir ve siz sadece hangi grubun, envanterin hangi parçasına müdahale edebileceğini merkezi olarak yönetirsiniz.
