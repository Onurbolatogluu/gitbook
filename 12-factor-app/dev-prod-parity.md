# 🔟 Dev/prod parity

<figure><img src="../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

**Dev/Prod Parity (Geliştirme ve Üretim Uyumsuzluğu)**

"Dev/Prod Parity," geliştirme (development) ve üretim (production) ortamlarının mümkün olduğunca benzer olması gerektiğini vurgular. Bu ilke, geliştirme sırasında kullanılan çevreler ile üretimde çalışan çevreler arasındaki uyumsuzluğun minimum seviyede tutulmasını amaçlar. İşte bu ilkenin anahtar unsurları:

**1. Çevresel Farklılıkları Azaltma:**

* **Amaç:** Geliştirme ve üretim çevrelerinin birbirine benzemesini sağlamak.
* **İşlem:** Geliştirme (development) çevresi ile üretim (production) çevresi arasındaki farklılıkları azaltmak için yapılandırma ayarları, bağımlılıklar ve diğer çevresel değişkenler aynı tutulmalıdır.

**2. Aynı Veritabanı ve Hizmetlere Erişim:**

* **Amaç:** Aynı veritabanlarına ve hizmetlere erişim sağlamak.
* **İşlem:** Geliştirme ve üretim çevreleri, aynı veritabanlarına ve dış hizmetlere erişebilmelidir. Bu, geliştiricilerin gerçek dünyada nasıl çalıştığını daha iyi anlamalarını sağlar.

**3. Geliştirme Çevresinin Kolay Kurulması:**

* **Amaç:** Geliştirme çevresini hızlı ve kolay bir şekilde kurabilmek.
* **İşlem:** Geliştiriciler, geliştirme çevresini hızlıca kurabilmelidir. Bu, geliştirme sürecinin hızlanmasına ve yeni ekip üyelerinin çabucak çalışmaya başlamasına yardımcı olur.

**4. Üretimde Kullanılan Verileri Kullanma:**

* **Amaç:** Geliştirme sırasında üretimde kullanılan verilerin benzerlerini kullanmak.
* **İşlem:** Geliştirme sırasında, üretimde kullanılan verilere benzer verilerin kullanılması teşvik edilmelidir. Örneğin, gerçek üretim verileri yerine benzer test verileri kullanılabilir.

**5. Geliştirme ve Üretimde Benzer Uygulama Davranışı:**

* **Amaç:** Geliştirme ve üretimde uygulamanın benzer şekilde davranmasını sağlamak.
* **İşlem:** Geliştirme çevresi, uygulamanın üretimde nasıl davranacağına dair gerçekçi bir önizleme sunmalıdır. Bu, beklenmedik sorunların üretimde ortaya çıkmasını azaltır.

**6. Test ve Doğrulama:**

* **Amaç:** Geliştirme ve üretim çevrelerinde test ve doğrulama yapmak.
* **İşlem:** Geliştirme çevresinde uygulamanın testleri ve doğrulamaları yapılmalıdır. Bu, üretimde beklenen sonuçların sağlanmasını kolaylaştırır.

Dev/Prod Parity ilkesi, geliştirme ve üretim arasındaki uyumsuzlukları en aza indirerek hataların ve sürprizlerin önlenmesine yardımcı olur. Geliştirme sürecini daha güvenilir ve tahmin edilebilir hale getirir ve yeni özelliklerin hızlı bir şekilde dağıtılmasını kolaylaştırır. Bu ilke ayrıca geliştirme ekibinin üretim ortamında daha rahat çalışmasını sağlar.

