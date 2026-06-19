---
icon: user-tie-hair
---

# Create User, Manage Permission at ESXi Level

Sanallaştırma altyapılarında güvenlik, fiziksel sunucunun (Host) kendisine erişimle başlar. Çoğu ortamda tüm sistem yöneticileri, ESXi sunucularına doğrudan bağlanmak için varsayılan `root` hesabını kullanma eğilimindedir. Ancak bu, hesap verilebilirliği tamamen ortadan kaldıran ve kurumsal güvenlik politikalarına (özellikle ISO 27001 veya PCI-DSS gibi standartlara) ters düşen tehlikeli bir pratiktir.

Her ne kadar devasa altyapılar merkezi olarak vCenter üzerinden yönetilse de, vCenter'ın ulaşılamadığı felaket anlarında veya tekil çalışan (Stand-alone) ESXi sunucularında yerel erişim kontrollerinin doğru yapılandırılması hayati önem taşır. Bu makalede, ESXi seviyesinde yerel kullanıcı oluşturma, rolleri atama ve Read Only profillerin davranış biçimlerini mimari bir bakış açısıyla inceleyeceğiz.

**1. İlk Adım: Yerel Kullanıcıyı (Local User) Yaratmak**

ESXi arayüzünde yerel bir hesap oluşturmak için `Manage > Security & users > Users` sekmesi kullanılır. Sisteme yeni bir uzman veya sadece izleme (monitoring) yapacak bir operatör eklendiğinde, onlara özgü bir kullanıcı adı ve güçlü bir parola tanımlanır.

Ancak burada VMware mimarisinin çok kritik bir mantığı devreye girer: Sadece bir kullanıcı oluşturmak, o kişiye sisteme giriş yapma (Login) hakkı vermez. Oluşturulan bu yeni hesap, tıpkı Microsoft Active Directory'de yaratılmış ama henüz hiçbir yetki grubuna dahil edilmemiş, "pasif" bir kimlik gibidir. Bu kullanıcının arayüze girebilmesi için ikinci adıma, yani "İzinler" (Permissions) atamasına geçilmesi zorunludur.

**2. Rolleri Eşleştirmek: "Permissions"**&#x20;

Oluşturduğumuz bu çıplak kimliğe bir "Rol" (Role) giydirmek için doğrudan Host üzerindeki `Permissions` (İzinler) menüsüne gidilir. Burada, kullanıcının sistemde ne kadar derinliğe inebileceğini belirleyen varsayılan kurallar dizisi bulunur:

* Administrator: Tıpkı `root` hesabı gibi donanımdan sanal makinelere kadar her şeyi silip oluşturabilen, sistemin mutlak hakimidir.
* Read Only: Sisteme giriş yapabilen, ayarları, donanım kaynaklarını ve makinelerin durumunu "görebilen" ancak hiçbir şeye dokunamayan roldür.
* No Access: Bu, sistem yöneticilerinin hayatını kurtaran gizli bir silahtır. İşten ayrılan veya yetkisi geçici olarak dondurulan bir kullanıcının hesabını tamamen silmek yerine, rolünü `No Access` olarak değiştirirsiniz. Kullanıcı anında sistemden aforoz edilir.

**3. Sistemi Test Etmek: "Read Only" Kullanıcısı Arayüzde Ne Yaşar?**

Bir Operasyon Merkezi (NOC) çalışanına veya bir denetçiye (Auditor) `Read Only` yetkisi verip sisteme giriş yapmasını sağladığınızda, VMware arayüzü çok ilginç bir savunma mekanizması sergiler.

Kullanıcı arayüze girdiğinde her şey normal görünür, ancak işlem yapmaya kalktığında arka plandaki katı kurallara çarpar:

* Yetki Görüntüleme Engeli: Bu kullanıcı `Permissions` sekmesine girip sistemde başka kimlerin yetkisi olduğunu görmek isterse, sayfa sürekli yükleniyor (loading) modunda kalır veya doğrudan erişim reddedildi hatası verir. Düşük yetkili bir gözün, yüksek yetkili hiyerarşiyi görmesi engellenir.
* Güç Operasyonları (Power Ops): Sistemdeki herhangi bir sanal makineyi kapatma (Power Off) veya kapalı bir makineyi açma (Power On) butonları işlevsizdir.
* Sihirbaz Tuzağı (Wizard Trap): VMware arayüzünün en belirgin özelliklerinden biridir. `Read Only` bir kullanıcı, "Yeni Sanal Makine Oluştur" butonuna basabilir. İsim verir, işletim sistemi seçer, donanım ayarlarını yapılandırır. Sihirbaz buna izin verir. Ancak en son adımdaki "Finish" (Bitir) butonuna bastığı an, ESXi'ın arka plan API'si bu isteği yakalar ve işlemi sert bir şekilde reddeder: _"The permission to perform this operation was denied." (Bu işlemi gerçekleştirme izni reddedildi)._ ---

> 💡 Uzman İpucu: Neden vCenter Varken ESXi Yerel Kullanıcılarıyla Uğraşıyoruz?
>
> Birçok sistem yöneticisi, _"Zaten tüm yetkileri vCenter ve Active Directory entegrasyonu ile yönetiyorum, ESXi yerel hesaplarına neden ihtiyaç duyayım?"_ yanılgısına düşer.
>
> Kurumsal mimarilerde "Kırılmaz Cam" hesapları adı verilen bir konsept vardır. Eğer vCenter sunucunuz çökerse, Active Directory (Kimlik Doğrulama) sunucularınız şifreleme virüsüne (Ransomware) kurban giderse, ortamdaki tüm merkezi yetkilendirme mekanizmanız saniyeler içinde buharlaşır.
>
> İşte bu "Kıyamet Günü" senaryosunda, sistemleri manuel olarak kurtarabilmek için doğrudan ESXi IP adreslerine giderek giriş yapmanız gerekir. Kök (root) parolasını tüm ekibe dağıtmak yerine, kıdemli mühendislere ESXi seviyesinde lokal ve güçlü şifreli `Administrator` hesapları açmak, felaket anında kriz yönetimini güvenli ve loglanabilir (kimin hangi sunucuya girdiğinin izlenebilir) hale getiren en kritik mimari standarttır.
