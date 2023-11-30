# 1⃣ Codebase

<figure><img src="../.gitbook/assets/Screen Shot 2023-09-18 at 12.34.24 AM.png" alt=""><figcaption></figcaption></figure>

**Codebase (Kod Tabanı)**

12 Factor App metodolojisinin ilk ilkesi, uygulamanızın kodunun nasıl organize edildiğini ve yönetildiğini ele alır. Bu ilke, bir uygulamanın tüm sürümlerinin tek bir kaynak kod deposu içinde takip edilmesini ve yönetilmesini gerektirir. İşte bu ilkenin anahtar unsurları:

**1. Tek Bir Kaynak Kod Deposu:** Uygulamanın tüm sürümleri, bir tek kaynak kod deposunda saklanmalıdır. Bu, uygulamanın farklı sürümlerinin, değişikliklerin ve güncellemelerin bir merkezi depoda izlenmesini kolaylaştırır.

**Örnek:** GitHub veya GitLab gibi bir çevrimiçi kod barındırma hizmeti kullanılarak, uygulamanın tüm sürümleri aynı depoda yönetilir.

**2. Kaynak Kodu Takip Etme:** Tüm kod değişiklikleri, sürüm kontrol sistemi kullanılarak takip edilir. Her değişiklik, bir "commit" ile kaydedilir ve bu, geçmiş değişikliklerin izlenmesini sağlar.

**Örnek:** Git'te, "git commit" komutu ile her değişiklik kaydedilir ve "git log" komutu ile geçmiş değişikliklere erişilir.

**3. Paralel Geliştirme:** Birden fazla geliştirici aynı kaynak kod deposu üzerinde çalışabilir. Her geliştirici, kendi işletim dalında (branch) çalışır ve değişikliklerini birleştirir (merge) ve çatallanmış (fork) kodları bir araya getirir.

**Örnek:** Farklı geliştiriciler, aynı projenin farklı işletim dallarında çalışabilir ve sonra bu dalları ana dal ile birleştirebilir.

**4. Teknolojik Çeşitlilik:** Kod tabanı, farklı teknolojiler ve bileşenler içerebilir. Örneğin, bir web uygulamasının frontend ve backend bileşenleri aynı kod tabanında bulunabilir.

**Örnek:** Bir web uygulamasının kod tabanı, JavaScript (frontend) ve Python (backend) kodlarını içerebilir.

**5. Versiyon Yönetimi:** Her sürüm, bir benzersiz sürüm numarası ile etiketlenmelidir. Bu, her sürümün önceden tanımlanmış bir sürüm numarasına sahip olduğunu ve dağıtıldığını gösterir.

**Örnek:** Sürüm 1.0.0 veya 2.3.1 gibi belirli bir sürüm numarasıyla etiketlenmiş sürümler.

**6. Ortak Paylaşım:** Tüm ekip üyeleri, kod tabanına erişim sağlayabilir ve değişiklikleri takip edebilir. Bu, ekip içi işbirliğini kolaylaştırır.

**Örnek:** Ekip üyeleri, aynı kod deposunda çalışır ve değişiklikleri düzenler.

Bu ilke, kodun düzenli ve yönetilebilir bir şekilde tutulmasını sağlar. Tüm ekibin aynı kaynak koduna erişebilmesi ve değişiklikleri izleyebilmesi, yazılım geliştirme süreçlerini daha verimli hale getirir.

