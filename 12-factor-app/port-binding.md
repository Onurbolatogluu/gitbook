# 8⃣ Port binding

<figure><img src="../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

**Port Binding (Bağlantı Noktası Bağlama)**

"Port Binding," bir uygulamanın dış dünyayla iletişim kurabilmek için kullanacağı bağlantı noktalarını nasıl yönetmesi gerektiğini ele alır. Bu ilke, uygulamanın ağ üzerinden erişilebilir hale gelmesini ve dış kaynaklara (örneğin, diğer hizmetlere veya kullanıcılara) nasıl hizmet verdiğini tanımlar. İşte bu ilkenin anahtar unsurları:

**1. Bağlantı Noktalarının Bağlama:**

* **Amaç:** Uygulamanın dış dünyayla etkileşimde bulunabilmesi için belirli bağlantı noktalarına ihtiyaç duyar.
* **İşlem:** Uygulama, dış dünyayla iletişim kurabilmesi için belirli bağlantı noktalarını dinler veya belirler. Örneğin, bir web sunucusu HTTP isteklerini dinlemek için 80 numaralı bağlantı noktasını kullanabilir.

**2. Bağlantı Noktalarının Çevresel Değişkenlerle Ayarlanması:**

* **Amaç:** Bağlantı noktalarının yapılandırılabilir olması ve çevresel değişkenlerle belirlenmesi.
* **İşlem:** Bağlantı noktalarının numaraları, çevresel değişkenler veya yapılandırma ayarları ile belirlenmelidir. Bu, farklı çevrelerde (development, production, staging) farklı bağlantı noktalarının kullanılmasını sağlar.

**3. Bağlantı Noktalarının Dinlenmesi:**

* **Amaç:** Uygulama, belirlenen bağlantı noktalarını dinler ve gelen isteklere cevap verir.
* **İşlem:** Uygulama, belirlediği bağlantı noktalarını sürekli dinler ve bu bağlantı noktalarına gelen isteklere yanıt verir. Örneğin, bir web sunucusu HTTP isteklerini dinler ve web sayfalarını gönderir.

**4. Bağlantı Noktalarının Dinamik Olarak Ayarlanması:**

* **Amaç:** Bağlantı noktalarının dinamik olarak belirlenmesi, ölçeklenebilirliği ve esnekliği artırır.
* **İşlem:** Modern uygulamalar, bulut ortamlarında veya konteyner platformlarında çalıştığında, bağlantı noktaları dinamik olarak atanabilir. Bu, uygulamanın otomatik olarak yeni bağlantı noktaları oluşturmasını veya mevcut bağlantı noktalarını dağıtmasını sağlar.

**5. Bağlantı Noktalarının Diğer Hizmetlere Bağlanması:**

* **Amaç:** Uygulama, diğer hizmetler veya bileşenlerle iletişim kurmak için belirli bağlantı noktalarını kullanabilir.
* **İşlem:** Uygulama, başka bir hizmete veri göndermek veya veri almak için belirli bir bağlantı noktasını kullanabilir. Örneğin, bir veritabanına bağlanmak için belirli bir bağlantı noktası kullanılabilir.

Port Binding ilkesi, uygulamanın dış dünyayla etkileşimde bulunma şeklini ve bu etkileşim için hangi bağlantı noktalarını kullanacağını düzenler. Bu, uygulamanın ölçeklenebilir, esnek ve çevresel farklılıklara uygun bir şekilde çalışmasını sağlar. Ayrıca, farklı hizmetlerle iletişim kurma yeteneği sunar.
