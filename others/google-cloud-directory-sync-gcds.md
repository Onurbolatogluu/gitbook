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

