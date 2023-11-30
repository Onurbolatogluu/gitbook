# 7⃣ Build, release, run

<figure><img src="../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

**Build, Release, Run (Derleme, Sürümleme, Çalıştırma) İlkesi**

**Build, Release, Run** uygulamanın geliştirme, sürümleme ve çalıştırma süreçlerini düzenler. Bu süreçlerin ayrı tutulması ve net bir şekilde tanımlanması, uygulamanın güvenilirliğini ve ölçeklenebilirliğini artırır.&#x20;

**1. Build (Derleme):**

* **Amaç:** Uygulama kodunun ve bağımlılıklarının, çalıştırılabilir bir paket haline getirilmesi.
* **İşlemler:**
  * **Kaynak Kodun Derlenmesi:** Uygulama kaynak kodu, derlenerek çalıştırılabilir bir hale getirilir. Özellikle derlenen diller (compiled languages) için önemlidir.
  * **Bağımlılıkların Çözülmesi:** Uygulama, çalıştırılabilmesi için ihtiyaç duyduğu bağımlılıkları (kütüphaneler, modüller, paketler) belirli bir sürümle çözer ve dahil eder.
  * **Yapılandırma Ayarlarının Eklenmesi:** Uygulamanın çalışması için gerekli olan yapılandırma ayarları (configurations) bu aşamada belirlenir ve eklenir. Bu ayarlar uygulamanın çevresine (environment) göre farklılık gösterebilir.

**2. Release (Sürümleme):**

* **Amaç:** Hazırlanan uygulama paketi, bir sürüm haline getirilir ve dağıtılmaya hazır hale gelir.
* **İşlemler:**
  * **Sürüm Etiketlemesi:** Her sürüm, sürüm numarası veya etiketi ile belirtilir (örneğin, v1.0.0).
  * **Sürüme Özgü Yapılandırma:** Her sürüm, çalıştırma anında kullanılacak özgün yapılandırma ayarlarına sahip olabilir. Bu, farklı sürümlerde farklı yapılandırmaları destekler.
  * **Sürüm Paketinin Oluşturulması:** Uygulama ve yapılandırma, bir sürüm paketi oluşturacak şekilde bir araya getirilir. Bu paket, sürümleme işlemi sonucu elde edilir.

**3. Run (Çalıştırma):**

* **Amaç:** Hazırlanan sürüm paketi, bir sunucu veya konteyner içinde çalıştırılarak uygulama canlıya alınır.
* **İşlemler:**
  * **Sürüm Paketinin Çalıştırılması:** Hazırlanan sürüm paketi, bir sunucu üzerinde veya konteyner içinde çalıştırılır. Bu adım, uygulamanın gerçek dünyada çalışmaya başladığı aşamadır.
  * **Çevresel Değişkenlerin Kullanılması:** Uygulama, çalışma zamanında ihtiyaç duyduğu yapılandırma ayarlarını çevresel değişkenler veya yapılandırma dosyaları kullanarak alır.
  * **İzleme ve Günlükleme:** Uygulamanın çalışma durumu izlenir, hatalar raporlanır ve günlük dosyaları oluşturulur.

Bu ilke, uygulama geliştirme sürecini daha düzenli ve ölçeklenebilir bir hale getirir. Her aşama farklı sorumlulukları üstlenir ve bu sayede hata ayıklama, geri dönüş yapma ve güncelleme işlemleri daha kolay hale gelir. Bu ayrı aşamalar aynı zamanda uygulamanın daha güvenilir ve istikrarlı bir şekilde çalışmasını sağlar.



