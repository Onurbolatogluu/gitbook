# 6️⃣ Config

**Config (Yapılandırma)**

"Config," bir uygulamanın yapılandırma ayarlarını nasıl ele alması gerektiğini tanımlar. Bu ilke, uygulamanın çevresel farklılıklara (development, staging, production) uyum sağlamasını ve yapılandırma ayarlarını kod tabanından ayırmasını hedefler. İşte bu ilkenin anahtar unsurları:

<figure><img src="../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

**1. Çevresel Farklılıkları Ayırma:** Uygulama, farklı çevreler (development, staging, production) için farklı yapılandırma ayarlarına sahip olmalıdır. Bu, her çevrede uygulamanın farklı şekilde davranabilmesini sağlar.

**Örnek:** Geliştirme (development) çevresinde veritabanı bağlantısı, test verileri kullanılabilirken, üretim (production) çevresinde farklı bir veritabanı sunucusu ve veriler kullanılabilir.

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

**2. Yapılandırma Ayarlarını Koddan Ayırma:** Yapılandırma ayarları, uygulama kodundan ayrılmalıdır. Kod tabanında yapılandırma ayarlarının sabit olarak yerine yazılması önlenmelidir.

**Örnek:** Veritabanı bağlantı bilgileri, kod yerine yapılandırma dosyalarında veya çevresel değişkenlerle sağlanmalıdır.

**3. Çevresel Değişkenler Kullanma:** Çevresel değişkenler (environment variables), uygulamanın yapılandırma ayarlarını taşımak ve gizlemek için yaygın olarak kullanılır.

**Örnek:** Veritabanı şifresi veya API anahtarı gibi hassas bilgiler çevresel değişkenlerle korunabilir.

**4. Tek Kaynak**: Yapılandırma ayarları, uygulamanın tek bir kaynaktan (örneğin, bir yapılandırma dosyası veya çevresel değişkenler) alınmalıdır. Farklı kaynaklardan yapılandırma ayarlarını birleştirmekten kaçınılmalıdır.

**Örnek:** Yapılandırma ayarları hem bir JSON dosyasında hem de çevresel değişkenlerde bulunmamalıdır.

**5. Sürdürülebilirlik:** Yapılandırma ayarları, kolayca güncellenebilir ve yönetilebilir olmalıdır. Herhangi bir değişiklik, uygulamanın yeniden başlatılmasını gerektirmemelidir.

**Örnek:** Yapılandırma ayarlarını değiştirmek, uygulamayı yeniden başlatmadan yürütülebilir.

**6. Güvence:** Yapılandırma ayarlarının güvence altına alınması, yetkisiz erişimi ve değişiklikleri önler.

**Örnek:** Çevresel değişkenlere sadece yetkilendirilmiş kişiler veya süreçler erişebilmelidir.

Bu ilke, uygulamanın yapılandırma ayarlarını düzenli ve güvenli bir şekilde yönetmesini sağlar. Her çevre için uygun yapılandırmayı sağlamak, uygulamanın her aşamada doğru şekilde çalışmasını garanti eder. Çevresel değişkenlerin kullanılması, hassas bilgilerin güvenli bir şekilde saklanmasına yardımcı olur.
