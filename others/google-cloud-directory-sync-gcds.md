---
icon: google
---

# Google Cloud Directory Sync ( GCDS )

### Nedir?

Google Cloud Directory Sync ( GCDS ) Google Workspace ile yerel kullanıcı dizininizi ( AD veya LDAP ) senkronize etmenizi sağlar. GCDS, kullanıcılar, gruplar, takma adlar (alias) ve paylaşılan kişileri google hesabınız ile eşleştirir. Kurallar ve hariç tutmalarla özelleştirilebilir. Tek yönlü senkronizasyon yapar ve LDAP sunucusundaki verileri değiştirmez. Senkronizasyon sonrası raporlar sağlar.

### Ne Değildir?

GCDS, e-posta mesajları, takvim etkinlikleri veya dosyalar gibi içerikleri Google hesabınıza taşımak için kullanılmaz. Bunun yerine, GCDS 'yi LDAP sunucunuzdaki kullanıcılar, grupları Google Workspace ile senkronize etmek için kullanırsınız.



### Önemli Kısımlar,

* GCDS, sunucuzda çalışan bir uygulamadır ve tüm gerekli bileşenler kurulum paketine dahildir.&#x20;
* GCDS best practies olarak, LDAP sunucunuza kurulmaz. Bunun yerine, GCDS ayrı bir sunucuya veya cihaza kurulur ve bu sunucu veya cihazdan LDAP sunucunuza erişir.&#x20;
* Senkronizasyon sırasında yanlızca iç ağ üzerinden erişim sağlar ve dış erişime ihtiyaç duyulmaz.
* GCDS, Google hesabınızdaki verilerin AD veya LDAP sunucunuzdaki verilerle eşleşmesini sağlar. Senkronizasyon tek yönlüdür; yani LDAP sunucunuzdaki veriler asla güncellenmez veya değiştirilmez.
* Kullanıcılar, gruplar, kullanıcı profilleri, takma adlar ve istisnalar gibi çeşitli öğelerin sync olması için özel kurallar ayarlayabilirsiniz.
* GCDS, senkronizasyon oluşturma ve çalıştırma sürecinde bizi adım, adım yönlendirir. Ayrıca ayarlardan emin olmamızı sağlamak için bir simülasyon aşaması içerir.
* Kullanıcılar, profiller, gruplar, organizasyon birimleri (ou) veya takvim kaynakları (toplantı odaları, ekipman vs) gibi verileri senkronizasyon dışında tutmak için exclude kuralları belirleyebiliriz.

### Nasıl Çalışır?

1. Verilerin listesinin nasıl oluşturulacağını belirlemek için kurallar oluşturulur.
2. Senkronizasyon sırasında, bu liste LDAP sunucunuzdan dışa aktarılır.
3. GCDS, Google hesabımıza bağlanır ve LDAP sunucumuzdaki belirttiğimiz kullanıcılar, gruplar, organizasyon birimlerinin (ou) vs listesini oluşturur.
4. GCDS, bu listeleri karşılaştırır ve Google hesabımızı LDAP sunucumuzdaki verilerle eşleşecek şekilde günceller.
5. Senkronizasyondan sonra, süreci izleyebilmemiz için bir e-posta raporu alırısınız.



### GCDS Ön Hazırlık,

* Ram miktarına dikkat edilmeli. Eğer LDAP dizininden çok sayıda varlık senkronize etmeyi planlıyorsak, ram kullanımı burada önemli olacaktır.
* GCDS 'nin kullanıldığı makinenin güvenliğinden emin olmak gerekmektedir. Yetkisiz olmayan dış kullanıcıların sunucu'ya bağlantı kurmaması gerekmektedir.
* LDAP verilerinizin hazır olduğundan emin olduktan sonra; ayarlarınızdan emin olmak için senkronizasyon simülasyonu çalıştırılmalıdır. Ardından, Google hesabınıza güncellemeleri aktarmak için gerçek bir senkronizasyon çalıştırılmaldır.
* GCDS senkronizasyonu yapmadan önce, yönetilmeyen kulanıcıları tespit etmeniz gerekmektedir ve gerekirse bu kullanıcılara kuruluşunuzun alan adı için davet göndermeniz gerekir. Kullanıcılar bu davetle kendi kişisel Gmail hesaplarını organizasyonunuzun yönetilen Google hesabına (yani Google Workspace hesabına) transfer edebilirler. Bu transfer işlemi, senkronizasyon sırasında bu kullanıcıların hesaplarıyla çakışan yeni hesaplar oluşturulmasını engeller.

### Hesapları Yönetme,

* LDAP dizininizde artık bulunmayan kullanıcı hesaplarınız varsa, GCDS 'yi bu hesapları silmek yerine askıya alacak şekilde ayarlayın. Çünkü silinen hesaplar 20 gün sonra geri getirilemez ve bu hesaplara ait tüm veriler kalıcı olarak kaybolur.  Ancak askıya alınan hesaplar için veriler saklanır ve bu verilere ( örneğin e-posta ve google drive içerikleri ) hala erişebilir veya bunları başka bir hesaba aktarabilirsiniz. Bu yanlışlıkla veri kaybını önlemek için önemli bir güvenlik önlemidir.
* LDAP dizininde bir kullanıcı hesabı oluşturulduğunda veya değiştirildiğinde, bu değişiklikleri hızlıca Google Workspace 'e yansıtmak için kullanıcı hesaplarını daha sık bir aralıkta senkronize edebilirsiniz. Örneğin, grup üyelikleri ve diğer acil olmayan değişiklikler daha seyrek aralıklarla senkronize edilebilir. Eğer sadece kullanıcı hesaplarını senkronize etmek istersek, bunu komut satırı kullanarak yapabiliriz.
* GCDS, LDAP dizininizde yer almayan Google yönetici hesaplarını varsayılan olarak askıya almaz veya silmez. Bu ayarı korumak önemlidir. Çünkü böylece yönetici hesaplarının yanlışlıkla silinmesini veya askıya alınmasını önleyebiliriz.

### Kurallar ve Limitler,

* GCDS, senkronizasyon sırasında belirli sayıda kullanıcı hesabını veya diğer öğeleri silebilir. Bu silme işlemlerinin yanlışlıkla çok fazla öğeyi silmesini önlemek için bir limit ayarlayabiliriz. Bu limiti kuruluşumuzun hesap sayısı ve silme işleminin etkileyeceği öğelerin sayısına göre belirlemeliyiz. Böylece GCDS belirlenen limiti aşmaya çalıştığımızda bizi uyarır ve kazara büyük miktarda veri kaybını önleyebiliriz. Peki ya GCDS neden hesaplarımızı silsin? Misal, eğer bir kullanıcı hesabı veya grup LDAP dizininden kaldırılırsa, GCDS bu değişikliği Google Workspace'e yansıtmak için ilgili hesabı veya grubu silmeye çalışır. Bu örneğin, işten ayrılan bir çalışan veya artık kullanılmayan bir grup için geçerli olabilir. Veya, LDAP dizininde yanlışlıkla yapılan bir değişiklik (örneğin, bir grup veya kullanıcının yanlışlıkla silinmesi) GCDS tarafından senkronize edilirse, bu hatalı değişiklik Google workspace'teki hesapları da etkileyebilir. Veya, En basit haliyle bir kullanıcının artık LDAP dizininde yer almaması durumunda bu kullanıcının Google Workspace'teki hesabı da silinir.
* Eğer LDAP dizinimizde bulunmayan, ancak Google Workspace'te mevcut olan kullanıcı hesapları veya gruplar varsa, bu hesapların silinmesini önlemek için **dışlama (exclude) kuralları** kullanmalıyız. Bu kurallar, belirli kullanıcı veya grupların senkronizasyon sırasında silinmemesini sağlar.&#x20;
* Eğer LDAP dizinimizdeki bazı kullanıcı veya grupların Google hesabına senkronize edilmesini istemiyorsak, bu verileri hariç tutmak için **odaklanmış arama kurallarını** kullanabiliriz. Bu kurallar, belirli kullanıcıları veya grupları seçerek senkronizasyon dışında bırakmamızı sağlar.

### Gereksinimler,

#### Google Hesabı

* Google hesabı veya Cloud identity hesabı.
* Bir Google Workspace veya Cloud identity süper yönetici hesabı.
* GCDS Windows Client ve Server sürümlerini destekler.
* Linux Desktop 32 ve 64 bit sürümlerini destekler. (GUI Şart)

#### Donanım Gereksinimleri

* Minimum 2 çekirdekli bir işlemci.
* Log dosyaları için en az 5 GB disk alanı.
* Ram mikrarı, verilerin boyutuna bağlıdır.
  * 10.000'den az varlık: 1 GB RAM
  * 10.000-200.000 varlık: 2-4 GB RAM
  * 200.000'den fazla varlık 8 GB RAM

NOT: Memory kullanımı bu süreç zarfında dikkatli bir şekilde takip edilmelidir.

#### LDAP Sunucusu

* AZURE AD desteklemez, eğer AZURE AD ile senkronizasyon yapmak istiyorsanız Directory Sync kullanılmalıdır.
* Tüm LDAP sürümleri desteklenir. Aşağıdaki bilgilere ihtiyacımız olacak;
  * LDAP sunucu üzerinde, LDAP yönetici erişimi.
  * LDAP sunucusuna ağ erişimi. (GCDS 'yi LDAP sunucunuzda çalıştırmanız gerekmez)
  * Senkronize etmek istediğiniz OU 'lar için, LDAP sunucusunda okuma izinleri gerekli.
  * LDAP üzerinde verileri incelemek ve teyit etmek için LDAP tarayıcısına ihtiyaç gerekli.
  * GCDS 'nin LDAP üzerinde bulunan bilgilere ve verilere erişimi olmaldır.



### Yönetilmeyen Hesaplar Hakkında,

Bir kişi 'user@gmail.com' gibi kişisel bir google hesabı kullanır. Bu hesap, google tarafından yönetilir ve kişisin kontrolündedir. Ancak bu kişi, şirketinizin alan adını kullanarak örneğin 'user@domain.com' gibi bir google hesabı oluşturmuş olabilir. Bu hesap, şirketin Google workspace yönetiminden bağımsız olarak oluşturulmuş olur. Bu durumda, 'user@domain.com' adresi ile açılan Google hesabı, şirketinizin Workspace yöneticileri tarafından kontrol edilemez ve yönetilemez. Bu nedenle bu tür hesaplar "Yönetilemeyen hesaplar" olarak adlandırılır.

Özetle, yönetilemeyen hesaplar demek, bir kişisinin şirketin alan adıyla açtığı Google hesabının, şirketin Workspace yöneticileri tarafından kontrol edilmediği anlamına gelir. Bu hesaplar kişisel hesaplardır ve şirket tarafından yönetilmezler. Bu sorunu çözmek işin 2 adet yol bulunmaktadır. BKNZ:&#x20;

{% embed url="https://support.google.com/a/answer/6178640?sjid=7451417005967322826-EU" %}

### Adım Adım Ön hazırlık ve Kontroller;

* LDAP sunucunuzun yapısı hakkında bilgi toplamak için bir LDAP tarayıcı indirip, yükleyin. Örneğin Softerra LDAP Administrator veya JXplorer gibi araçları kullanabilirsiniz. LDAP tarayıcıları, LDAP dizininizdeki kullanıcılar, gruplar ve diğer nesneler hakkında bilgi toplamanıza ve bunları incelemenize olanak tanır. Bu araçları kullanarak, LDAP sunucunuzun yapısını daha iyi anlayabilir ve yönetebilirsiniz.
* LDAP sunucuya bağlantı sağlamak için kullanılan hostname adı veya IP adresi gerekli.
* Standart LDAP bağlantısı mı, yoksa SSL üzerinden LDAP bağlantısı mı kullanmamız gerektiğini belirlemeliyiz.
* GCDS 'in LDAP sunucusundaki kullanıcı ve grup bilgilerini senkronize edebilmesi için, LDAP sunucuda okuma izinlerine sahip bir hesap oluşturması veya varsa mevcut bir sınırlı izinlere sahip yönetici hesabı kullanabiliriz. Bu hesap, GCDS 'in gerekli verileri okuyabilmesini sağlar. Eğer sadece belirli kullanıcıları ve grupları senkronize etmek istiyorsak, bu hesap için ouma izinlerini sınırlandırabiliriz.
* GCDS, sadece tek bir LDAP dizininden veri alabilir. Birden fazla LDAP dizini varsa, LDAP sunucusu verilerini tek bir dizinde birleştirmeliyiz.
* Tam bir senkronizasyon yapmadan önce, bunu kapsamlı bir şekilde test etmeliyiz (simülasyon yaparak). Bunun nedeni global katalogdan gelen verilerin ana LDAP dizinindeki verilerden farklı olabilme ihtimalidir. Bu farklılıklar senkronizasyon işleminde sorunlara yol açabilir.
* GCDS, LDAP sorguları için temel DN'yi (Base DN) en üst düzey olarak kullanır. GCDS kullanıcıları ve grupları bu temel DN 'den arar, bu yüzden senkronize etmek istediğiniz kullanıcılar ve grupları içeren bir düzeyde temel DN belirltmeliyiz.
  * Bir yapılandırmada birden fazla temel DN kullanabiliriz. Her senkronizasyon kuralı için ayrı bir temel DN belirtebiliriz.
* Senkronize etmek istediğimiz kullanıcılar ve gruplar için hangi bilgileri önemli olduğunu belirtmek gerekiyor. Bu bilgiler LDAP dizininde "öznitelik" olarak adlandırılır. Öznitelikler, kullanıcı adı, e-posta adresi, telefon numarası gibi bilgiler içerir. Bu bilgileri bulmak için, bir LDAP tarayıcı kullanarak LDAP dizini incelenmelidir. Yani, LDAP dizininde örnek kullanıcıları ve grupları açıp, hangi bilgilerin orada bulunduğunu ve hangi bilgilerin senkronize edilmesi gerektiğini kontrol etmeliyiz. Örneğin, bu kullanıcı hangi grupta? Bu kullanıcının e-posta adresi ne? gibi soruları cevaplamaya çalışmalıyız ki bu adım senkronize etmek istediğimiz bilgileri netleştirmek içindir.
* Şİrket içerisinde belirli kullanıcıları veya kaynakları yöneten gruplar vardır. Bu gruplar security group olarak geçer. Bu gruplar genellikle belirli yetkilere veya güvenlik ayarlarına sahip kullanıcıları içerir. Senkronizasyon sırasında bu grupların da Google Workspace 'e aktarılmasını istiyorsanız, hani grupların senkronize etmek istediğinizi belirlemek gerekir. Bu grupların Google Workspace ile doğru bir şekilde senkronize olabilmesi için, her grubun kendi benzersiz bir e-posta adresi olmalıdır. Örneğin, "Güvenlik Grubu 1" adında bir grubumuz varsa, bu grubun "securitygroup1@domain.com" gibi bir benzersiz e-posta adresine sahip olması gerekir. Bu e-posta adresi, grubun Google Workspace 'de doğru bir şekilde tanınmasını ve senkronize edilmesini sağlar. Özetle, senkronize etmek istediğiniz güvenlik gruplarının her biri için benzersiz bir e-posta adresi ayarlamak gerekmektedir.
* LDAP içerisinde deskteklenmeyen karakterler olmadığından emin olunmalı. [https://support.google.com/a/answer/9193374?sjid=7451417005967322826-EU](https://support.google.com/a/answer/9193374?sjid=7451417005967322826-EU)



Senkronizasyon yapmadan önce aşağıdaki adımlardan ilham alarak kullanıcıları işaretleyebiliriz.

* LDAP dizininde, Google ile senkronize etmeyi planladığımız kullanıcıları belirlemek için onlara  "GoogleUsers" gibi özel bir isim veya etiket vermeliyiz. Senkronizasyon işlemi başarılı bir şekilde tamamlandıktan sonra, bu kullanıcıları "GoogleActiveUsers" gibi farklı bir isimle etiketleyerek aktif Google kullanıcılarını ayırt edebiliriz. Bu şekilde, hangi kullanıcıların Google ile senkronize edildiğini ve aktif olarak kullanıldığını takip ederiz.
* LDAP üzerinde bir OU oluşturup ve taşınacak kullanıcıları bu OU altına taşıyabiliriz. GCDS 'i yalnızca bu grubun üyelerini okuyacak şekilde ayarlayabiliriz.
* Taşınacak kullanıcılar için, özel bir öznitelik oluşturup ve bu özniteliği yeni kullanıcılar için ayarlayabilir ardından GCDS 'yi yanlızca bu özniteliğe sahip kullanıcıları okuyacak şekilde ayarlayabiliriz.

#### Hangi Kullanıcı Verileri Senkronize Edilmeli?

* Kullanıcılar: LDAP dizininizdeki kullanııcları bir LDAP tarayıcı kullanarak gözden geçirmeliyiz ve doğru sayıda kullanıcıyı sync ettiğimizden emin olmalıyız. Lisans sayısından fazla kullanıcı içe aktarırsak, senkronizasyon sırasında hatalarla karşılaşabiliriz.
* Kullanıcı Profilleri: Eğer LDAP dizininizde adresler, telefon numaraları veya diğer iletişim bilgileri gibi ek bilgiler varsa, bu bilgileri de senkronize edebiliriz.
* Aliases: LDAP dizininde takma ad özniteliklerini Google adres takma adlarına senkronie edebiliriz.
* Unique ID:  Eğer kullanıcıların kullanıcı adlarını (e-posta adreslerini) değiştirme olasılığı varsa, senkronizasyonu ayarlamadan önce bir "Unique ID" özniteliği oluşturulmalı (varsa gerek yok). Bu, kullanıcı bilgileri kaybolmadan e-posta değişikliği yapabilmemizi sağlar. Bu ayarlama yapılmazsa, sistem o kullanıcıyı yeni bir kullanıcı olarak algılar ve aynı kullanıcı için yeni bir hesap oluşturabilir. Bu durumda eski hesapta yer alan veriler kaybolabilir. Unique ID ile, kullanıcı e-posta aresi değişse bile,  kullanıcının tüm verilerini koruyarak aynı kişi olarak tanımasını sağlar.



* LDAP sunucudan Google hesabına senkronize etmek istediğiniz mailing list'leri belirlemek gerekmektedir. LDAP sunucusundaki mailing listler, Google hesabında gruplar olarak içe aktarılır.
* Bazı mailing listeleri doğrudan e-posta adresi içerir ve şu formatta olabilir: 'user@example.com' Bazıları ise tanımlanmış isim (DN) referansı içerir ve şu formatta olabilir `cn=Terri Smith,ou=Executive Team,dc=example,dc=com`."
*   Mailing listelerini Google hesabında korumak istersek,

    * Mailing listelerinin üyelerini içeren öznitelik (genellikle 'member veya 'MailAddress' özniteliği) bulun.
    * LDAP' üzerinde mailing listesi için kullanılan özniteliğin bir e-posta adresi mi yoksa bir DN mi içerdiğini öğrenin.



**Organizasyon Yapısı**

* &#x20;**Varsayılan olarak**, GCDS (Google Cloud Directory Sync), tüm kullanıcıları tek bir düz yapı içinde senkronize eder. Bu, küçük bir organizasyonunuz varsa veya tüm kullanıcıların aynı ayar ve haklara sahip olmasını istiyorsanız işe yarar. Ayrıca, daha büyük bir yayılımdan önce küçük bir grup üzerinde test yapıyorsanız da uygundur.
* Eğer Google Hesabınızda bir OU hiyerarşisi kullanmak istiyorsanız, organizasyon hiyerarşisini LDAP dizin sunucunuzdan Google Hesabınıza senkronize edebilirsiniz. Bunu yapmadan önce, LDAP tarayıcı ile OU 'ları gözden geçirin ve doğru yapıyı senkronize ettiğinizden emin olun. Örneğin, yazıcılar için özel bir OU olabilir ve bu birimi Google Hesabınıza aktarmak istemeyebilirsiniz.
* Eğer OUları Google Hesabınızda manuel olarak oluşturmak isterseniz, bunları Google'da ayarlayabilir ve ardından GCDS'yi kullanıcıları mevcut organizasyonlara taşıyacak şekilde yapılandırabilirsiniz. Mevcut organizasyonları değiştirmeden bu seçeneği ayarlamak için Configuration Manager'da **Org Units** sayfasını seçin. Her kullanıcı arama kuralı için, kullanıcıların hangi organizasyona ait olacağını veya uygun organizasyonun adını içeren bir LDAP özniteliğini belirtin.

#### Parolaları Nasıl Senkronize Ediliyor?

* GCDS, sınırlı sayıda parola işlemini destekler. Parolaları yalnızca düz metin (plain text), Base64, MD5 veya SHA-1 gibi ek güvenlik katmanları (salt) eklenmemiş formatlarda depolayan bir LDAP özniteliğinden içe aktarabilir. Daha karmaşık şifreleme yöntemleriyle korunmuş parolalar desteklenmez.

### Google Cloud Directory Sync (GCDS) ile Neler Senkronize Edilebilir?

#### **Gruplar ve Organizasyon Birimleri**

* **Organizational Units:** LDAP dizinindeki organizasyon birimlerini Google Hesabınıza senkronize edebilirsiniz.
* **Mailing Lists**: LDAP'taki posta listeleri, Google Grupları olarak senkronize edilebilir.
* **Desteklenmeyen Özellikler**: GCDS, güvenlik gruplarını (security groups) senkronize etmez. LDAP'taki bir güvenlik grubu, Google'da normal bir grup olarak senkronize edilir.

#### **Kullanıcılar**

* **Kullanıcılar**: LDAP dizinindeki kullanıcılar, Google Hesabınıza senkronize edilebilir. Bu senkronizasyona yönetici yetkilerine sahip kullanıcılar da dahildir.
* **User Aliases**: Birden fazla kullanıcı takma adını Google e-posta takma adları olarak senkronize edebilirsiniz.
* **Extended User Information**: Telefon numaraları, adresler ve diğer LDAP bilgileri Google kullanıcı profiline senkronize edilebilir.

#### **Takvim**

* **Odalar ve Diğer Takvim Kaynakları**: Toplantı odaları gibi takvim kaynakları, Google Takvim kaynakları olarak senkronize edilebilir.

#### **Kişiler**

* **Shared Contacts**: LDAP dizinindeki paylaşılan harici kişiler, Google Hesabınıza senkronize edilebilir.
* **Desteklenmeyen Özellikler**: GCDS, kişisel kişileri senkronize etmez. Kişisel kişiler için alternatif bir geçiş aracı kullanılması gereklidir.

#### **Parolalar**

* **Parolalar**: LDAP dizinindeki parolalar Google Hesabınıza senkronize edilebilir, ancak GCDS tüm parola formatlarını desteklemez. Eğer Microsoft Active Directory kullanıyorsanız, parolaları Password Sync aracı ile senkronize edebilirsiniz.

***

***

{% embed url="https://support.google.com/a/answer/6258071?sjid=7451417005967322826-EU#user&zippy=%2Cuser-attribute-settings" %}

{% embed url="https://support.google.com/a/answer/6152425?sjid=7451417005967322826-EU" %}

{% embed url="https://support.google.com/a/answer/106368?hl=en&ref_topic=2679497&sjid=7451417005967322826-EU" %}

{% embed url="https://support.google.com/a/answer/6126578?hl=en&ref_topic=6126585&sjid=17751314726558257934-EU" %}
