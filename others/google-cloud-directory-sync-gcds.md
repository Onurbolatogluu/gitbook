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



***

***

{% embed url="https://support.google.com/a/answer/6258071?sjid=7451417005967322826-EU#user&zippy=%2Cuser-attribute-settings" %}

{% embed url="https://support.google.com/a/answer/6152425?sjid=7451417005967322826-EU" %}

{% embed url="https://support.google.com/a/answer/106368?hl=en&ref_topic=2679497&sjid=7451417005967322826-EU" %}
