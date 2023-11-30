# 🕚 Logs

<figure><img src="../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

**Logs (Günlükler)**

"Logs," uygulamanın ürettiği günlük (log) verilerinin nasıl ele alınması gerektiğini ve bu verilerin etkili bir şekilde izlenmesi ve yönetilmesini düzenler. Günlük verileri, uygulamanın durumu, hatalar ve işlem aktiviteleri gibi önemli bilgileri kaydetmek için kullanılır.

**1. Günlük Verilerinin Standardize Edilmesi:**

* **Amaç:** Günlük verilerinin belirli bir standart formatı takip etmesi.
* **İşlem:** Günlük verileri, belirli bir standart formatta olmalıdır. Bu, günlükleri kolayca okunabilir ve analiz edilebilir hale getirir.

**2. Günlüklerin Standart Çıktıya Yazılması:**

* **Amaç:** Günlük verilerinin standart çıktıya yazılması.
* **İşlem:** Uygulama günlük verilerini standart çıktıya (stdout veya stderr gibi) yazmalıdır. Bu, günlüklerin merkezi bir konumda toplanmasını kolaylaştırır.

**3. Günlük Verilerinin Dışa Aktarılabilir Olması:**

* **Amaç:** Günlük verilerinin uygulama içinden dışa aktarılabilir olması.
* **İşlem:** Günlük verileri, uygulama içinden dış sistemlere veya depolama hizmetlerine aktarılabilir olmalıdır. Bu, günlüklerin izlenmesini ve analiz edilmesini sağlar.

<figure><img src="../.gitbook/assets/image (177).png" alt="" width="375"><figcaption></figcaption></figure>

**4. Günlüklerin Depolanması ve İzlenmesi:**

* **Amaç:** Günlük verilerinin uzun süreli depolanması ve izlenmesi.
* **İşlem:** Günlük verileri, uzun süreli depolama sistemlerine kaydedilmeli ve izlenmelidir. Bu, hataların tespit edilmesi, izlenmesi ve performans analizlerinin yapılması için önemlidir.

**5. Günlük Seviyeleri ve Kategorileri:**

* **Amaç:** Günlük verilerinin seviyeler ve kategorilere ayrılması.
* **İşlem:** Günlük verileri, farklı seviyelerde (örneğin, hata, uyarı, bilgi) ve kategorilerde (örneğin, veritabanı, ağ, güvenlik) ayrılmalıdır. Bu, günlüklerin daha iyi filtrelenmesini ve analiz edilmesini sağlar.

**6. Günlük Verilerinin Korunması:**

* **Amaç:** Günlük verilerinin güvenli ve gizli tutulması.
* **İşlem:** Hassas bilgilerin (örneğin, şifreler veya kullanıcı verileri) günlüklerde görünmemesi ve günlük verilerinin yetkilendirilmiş kişiler dışında erişilmez olması sağlanmalıdır.

**7. Günlük Verilerinin Düzenli Temizlenmesi:**

* **Amaç:** Günlük verilerinin gereksiz birikimini önlemek.
* **İşlem:** Günlük verileri, belirli bir süre sonra otomatik olarak temizlenmelidir. Bu, depolama kaynaklarının verimli kullanılmasını sağlar.

Günlükler, uygulamanın performansını izlemek, hataları tespit etmek, güvenliği sağlamak ve sorunları gidermek için hayati bir rol oynar. Bu ilke, günlük verilerinin düzenli ve etkili bir şekilde yönetilmesini ve analiz edilmesini sağlar, böylece uygulama sürekli olarak iyileştirilebilir.
