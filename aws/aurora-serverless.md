# 🎒 Aurora Serverless

AWS artık veri tabanı oluştururken, "Basic Create" seçeneği ile çok daha hızlı ve basit veri tabanı kurmamızı sağlayan bir seçenek (sekme) ekledi.

{% hint style="info" %}
Aurora, AWS DB servisi hizmeti.
{% endhint %}

Aurora Serverless, Önceden DB oluştururken kaynağı ilk başta seçip, 2. defa değiştiremiyorduk. Sunucu çok fazla kaynak kullansa dahi işlem yapılamıyordu kaynak üzerinde.  Aurora serverless ile DB oluştururken, minimum kaynak ve maksimum kaynak belirliyoruz. Örnek 2 CPU 4 GB Memory Min. 16 CPU 64 GB Memory Max. Sunucu ihtiyacına göre, yani DB ihtiyacına göre burada belirlediğimiz değer arasında kullanım yapılabiliyor. Böylece autoscaling yapabilecek. Servise (DB) hiç istek gelmiyorsa, istek gelene kadar servisi pause duruma getirebilir.&#x20;

*   Pause compute capacity after consecutive minutes of inactivity: 1 saat yazarsak, 1 saat boyunca istek gelmezse servisi pause edecek.


*   Force scaling the capacity to the specified values when the timeout is reached:  Aurora Serverless, kaynak dalgalanmaları durumunda güvenli bir şekilde ölçeklendirmek için **ölçeklendirme noktaları** arıyor. Ancak aşağıdaki durumlarda ölçekleme noktası belirlenemez ve bir zaman aşımı meydana gelebilir.

    * **Devam eden uzun vadeli sorgularınız veya işlemleriniz varsa vb.**
    * Geçici bir tablo veya tablo kilidi kullanıyorsanız.

    <mark style="color:orange;">Bu güncelleme ile, ölçekleme noktası belirlenemediğinde ve yukarıdaki kullanım koşullarından dolayı zaman aşımı oluştuğunda, ölçeklendirmenin</mark> <mark style="color:orange;"></mark><mark style="color:orange;">**zorla yapılıp yapılmayacağını seçmek artık mümkün**</mark><mark style="color:orange;">.</mark>
