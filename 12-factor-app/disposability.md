# 9⃣ Disposability

<figure><img src="../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

**Disposability (Atılabilirlik)**

12 Factor App metodolojisinin dokuzuncu ilkesi olan "Disposability," bir uygulamanın herhangi bir zaman hızlı ve güvenli bir şekilde başlatılabilir ve sonlandırılabilir olmasını vurgular. Bu ilke, uygulamanın hata durumlarına nasıl yanıt verdiğini ve yedeklilik (failover) için nasıl hazırlandığını düzenler.&#x20;

**1. İşlemi Başlatma ve Sonlandırma:**

* **Amaç:** Uygulama işlemi hızlıca başlatılabilir ve sonlandırılabilir olmalıdır.
* **İşlem:** Uygulama, başlatma ve sonlandırma işlemlerini hızlıca gerçekleştirebilmelidir. Bu, uygulamanın ölçeklenmesi, güncellenmesi veya hata durumlarına yanıt vermesi için önemlidir.

**2. Durumun Korunmaması:**

* **Amaç:** Uygulama durumunun korunmaması, her başlatma öncesinde temiz bir durumda başlamayı sağlar.
* **İşlem:** Uygulama başlatıldığında herhangi bir önceki durumu (state) korumamalıdır. Uygulama, her başlatma öncesinde temiz bir durumda başlamalıdır. Bu, ölçeklenme ve hata toleransı için önemlidir.

**3. Durumun Harici Olarak Saklanması:**

* **Amaç:** Uygulama durumu harici bir veri deposunda saklanmalıdır.
* **İşlem:** Uygulama durumu, uygulama sunucusu dışında bir veri deposunda saklanmalıdır. Bu, uygulamanın daha kolay yedeklenmesini ve paylaşılmasını sağlar.

**4. Yedekliliğin Sağlanması:**

* **Amaç:** Uygulama, hata durumlarına yanıt olarak otomatik olarak yedeklilik sağlamalıdır.
* **İşlem:** Uygulama, hata durumlarına karşı otomatik olarak yedeklilik sağlamalıdır. Örneğin, bir sunucu çöktüğünde başka bir sunucu otomatik olarak devralmalıdır.

**5. Hızlı Ölçeklenme:**

* **Amaç:** Uygulama, yük artışlarına hızlı bir şekilde yanıt verebilmelidir.
* **İşlem:** Uygulama, yük artışlarına hızlı bir şekilde yanıt verebilmelidir. Yeni işlem örnekleri hızlıca başlatılabilir ve gerektiğinde sonlandırılabilir.

**6. İzleme ve Günlükleme:**

* **Amaç:** Uygulama işlem durumunu ve hataları izlemeli ve günlüklemelidir.
* **İşlem:** Uygulama, işlem durumunu ve hataları izlemeli ve günlüklemelidir. Bu, hata ayıklama ve hata tespiti için önemlidir.

Disposability ilkesi, uygulamanın herhangi bir zamanda başlatılabilir, sonlandırılabilir ve ölçeklenebilir olmasını sağlar. Bu, uygulamanın daha güvenilir ve esnek olmasına yardımcı olur ve hata durumlarına daha etkili bir şekilde yanıt vermesini sağlar. Ayrıca, izleme ve günlüklemeyi teşvik eder, bu da uygulamanın performansını ve güvenilirliğini artırır.

