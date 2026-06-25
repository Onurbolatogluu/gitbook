---
icon: timeline
---

# Create a New User and Roles at vCenter Server Level

Sanallaştırma altyapılarında yönetim tek bir ESXi sunucusundan çıkıp merkezi bir vCenter Server üzerine taşındığında, kimlik doğrulama ve yetkilendirme mekanizmaları da boyut atlar. Artık sadece bir fiziksel sunucuyu değil; devasa veri merkezlerini, Clusterlara ve yüzlerce sanal makineyi kapsayan geniş bir hiyerarşiyi yönetiyorsunuz demektir.

Bu geçiş, "Kimin, hangi kaynak üzerinde, ne yapmaya yetkisi var?" sorusunu kurumsal bir güvenlik politikası çerçevesinde yanıtlamayı zorunlu kılar. Bu makalede, vCenter arayüzü üzerinden (`Administration` menüsü kullanılarak) kendi kimlik veritabanımızda nasıl yeni kullanıcılar oluşturacağımızı, varsayılan (Default) ve özel (Custom) rolleri nasıl yapılandıracağımızı ve bu izinleri sisteme nasıl entegre edeceğimizi inceleyeceğiz.

**1. vCenter SSO ve `vsphere.local` Domain'i**

vCenter Server, Microsoft Active Directory gibi dış bir kimlik kaynağına bağlanmadan da kendi içinde tam teşekküllü bir kimlik doğrulama mekanizması (SSO - Single Sign-On) barındırır. Bu sistemin varsayılan etki alanı (Domain) `vsphere.local` olarak adlandırılır.

Yönetim paneline (Administration) girip `Users and Groups` (Kullanıcılar ve Gruplar) sekmesine ulaştığınızda, yeni bir kullanıcı profili oluşturabilirsiniz.

* Kimlik İsimlendirmesi: Burada oluşturduğunuz bir kullanıcı (Örneğin ismi "Jack" olsun), sisteme giriş yaparken doğrudan adını değil, kimlik alanını da belirterek `jack@vsphere.local` formatını kullanmak zorundadır.
* Şifre Politikaları (Password Policy): vCenter, sıradan sistemlere kıyasla çok daha katı bir güvenlik politikasına sahiptir. Yeni bir kullanıcıya "123456" gibi basit bir şifre veremezsiniz; sistem bunu anında reddeder ve büyük/küçük harf, rakam ve özel karakter barındıran kompleks bir şifre girmenizi zorunlu kılar.

**2. Yetki Şablonları: Varsayılan ve Özel Roller (Roles)**

Kullanıcıyı oluşturduktan sonra, ona bir "Karakter" veya "Görev Tanımı" yüklemeniz gerekir. vCenter `Roles` menüsünde, ortamdaki her türlü donanım ve yazılım bileşenine müdahale edebilen devasa bir "Ayrıcalıklar" (Privileges) kütüphanesi bulunur.

Bu ekranda sistem yöneticisini iki farklı yol bekler:

* Varsayılan Roller (Default Roles): VMware'in sektör standartlarına göre önceden hazırladığı şablonlardır. _Administrator_ (Tam yetkili), _Read Only_ (Sadece İzleyici) veya _Virtual Machine Power User_ gibi hazır paketler bulunur.
* Özel Roller (Custom Roles): Eğer mevcut şablonlar ihtiyaçlarınızı tam karşılamıyorsa, yepyeni bir rol inşa edebilirsiniz. Örneğin; sadece Veri Deposu (Datastore) üzerindeki alarmları yönetebilecek (Acknowledge/Modify Alarm) veya datastoreları yeniden isimlendirebilecek (Rename Datastore) çok spesifik ve mikro yetkilerle donatılmış, tamamen size özgü bir rol oluşturabilirsiniz.

**3. Kutsal Üçgen: İzinleri (Permissions) Entegre Etmek**

Kullanıcıyı oluşturduk ve rolü tasarladık (veya varsayılanlardan birini seçtik). Ancak VMware mimarisinde iş burada bitmez. Sistemde, kullanıcının bu rolü tam olarak nerede uygulayacağını bilmesi gerekir.

İşte bu noktada ana vCenter nesnesine (veya belirli bir Datacenter'a) tıklayıp Permissions (İzinler) sekmesine geçiş yapılır. "Add Permission" (İzin Ekle) sihirbazı, sanallaştırma mimarisinin şu "Kutsal Üçgenini" kurduğunuz yerdir:

1. Domain (Kaynak): `vsphere.local` seçilir.
2. Kullanıcı/Grup: Oluşturulan hesap (`jack`) seçilir.
3. Rol (Role): Seçilen hesaba hangi yetki paketinin giydirileceği (Örn: Virtual Machine Power User) belirlenir.

> 💡 Uzman İpucu: "VM Power User" Rolünün Kaputunun Altında Ne Var?
>
> Birçok sistem yöneticisi, kullanıcılara hazır bir şablon olan "Virtual Machine Power User" rolünü atar ve bu kişilerin makineler üzerinde tam yetkili olduğunu zanneder. Ancak vCenter mimarisi inanılmaz derecede detaylıdır.
>
> Bu rolü atadığınız bir kullanıcı; sanal makineyi açıp kapatabilir, zamanlanmış görevler oluşturabilir veya makineye yepyeni bir sanal disk ekleyebilir. Ancak bu rolün katı kısıtlamaları da vardır: Bu kullanıcı ortamdaki bellek (RAM) limitlerini değiştiremez veya mevcut bir sanal diskin boyutunu genişletemez (Extend Virtual Disk). Yani, hazır rolleri kullanırken bile alt menülere girip o rolün neleri kapsayıp neleri kapsamadığını dikkatlice okumak, gelecekte yaşanacak "Yetkim yok" operasyonel krizlerini önlemenin en garanti yoludur.
