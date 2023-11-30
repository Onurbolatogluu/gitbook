# 3⃣ Concurrency

**Concurrency (Eşzamanlılık)**

"Concurrency," bir uygulamanın eşzamanlı olarak çalışma yeteneğini ifade eder. Bu ilke, uygulamanın yüksek trafik veya iş yükleri altında nasıl ölçeklendirileceğini ve verimli bir şekilde çalışacağını ele alır. İşte bu ilkenin önemli unsurları:

**1. Süreç Tabanlı Çalışma:** Uygulamanın çalışma birimi, bir veya daha fazla süreç olarak tasarlanmalıdır. Her bir süreç, bir örneğini temsil eder ve işlemi yürütür.

**Örnek:** Bir web sunucusu, gelen istekleri eşzamanlı olarak işlemek için birden fazla süreci kullanabilir.

**2. Ölçeklendirme:** Uygulama, ihtiyaç duyulduğunda daha fazla süreç ekleyerek ölçeklendirilebilir. Bu, yüksek trafik veya talep durumunda daha fazla kaynak tahsis edilmesini sağlar.

**Örnek:** E-ticaret uygulaması, tatil sezonlarında artan talep için daha fazla sunucu örneği başlatarak ölçeklenebilir.

**3. İş Parçacığı (Thread) ve İşlem (Process) Yönetimi:** Eşzamanlılık, iş parçacıkları (threads) veya işlemler (processes) kullanılarak yönetilir. İş parçacıkları, aynı süreç içinde çalışırken işlemler farklı süreçler arasında çalışır.

**Örnek:** Bir web sunucusu, her gelen isteği bir iş parçacığı veya işlemde işleyebilir.

**4. Durumsuz Tasarım:** Eşzamanlılık, durumsuz (stateless) tasarımı teşvik eder. Bu, her işlem veya isteğin kendi bağımsız durumuyla çalışmasını sağlar ve paylaşılan verilere erişimde sorunları önler.

**Örnek:** Bir mikro hizmet, her isteği bağımsız olarak işler ve veritabanı erişiminde sıkıntılar yaşamaz.

**5. Asenkron İşleme:** Uygulama, uzun sürecek işlemleri eşzamanlı olarak işlemek için asenkron işleme yöntemlerini kullanabilir. Bu, uygulamanın işlemleri beklemek yerine diğer işleri gerçekleştirmesini sağlar.

**Örnek:** Bir e-posta gönderme işlemi, asenkron olarak arka planda işlenebilir.

**6. Hızlı Başlatma ve Kapatma:** Süreçlerin hızlı bir şekilde başlatılması ve kapatılması, uygulamanın ölçeklenmesi ve bakımı için önemlidir.

**Örnek:** Yeni bir sürecin hızlıca başlatılabilmesi, yüksek trafik anlarında daha fazla kaynağın devreye alınmasına yardımcı olur.

Eşzamanlılık ilkesi, uygulamanın taleplere yanıt verme yeteneğini artırır ve ölçeklendirme esnekliği sağlar. Bu sayede uygulama, büyüyen kullanıcı tabanlarına ve iş yüklerine daha iyi uyum sağlayabilir.
