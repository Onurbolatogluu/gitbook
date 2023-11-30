# 💻 Client Libraries

Prometheus Client Libraries, Prometheus'un desteklediği dillerde yazılı uygulamalarınızı ve hizmetlerinizi Prometheus ile entegre etmenizi sağlayan bir dizi kütüphanedir. Bu kütüphaneler, uygulamalarınızın veya hizmetlerinizin içinden metrikler toplamanıza ve Prometheus sunucusu tarafından alınabilir ve analiz edilebilir bir formatta bu metrikleri sunmanıza olanak tanır.

Prometheus Client Libraries, çeşitli programlama dilleri için kullanılabilir. Bu diller arasında Go, Java, Python, Ruby, .NET, Node.js, Rust ve diğerleri bulunur. Bu kütüphaneleri kullanarak, uygulamanızın veya hizmetinizin performansını ve sağlığını izlemek, hataları ve sorunları tespit etmek ve uygun ölçekleme ve optimizasyon kararları almak için değerli bilgiler elde edebilirsiniz.



Prometheus Client Libraries, kullanıcıların uygulamalarından ve hizmetlerinden metrikler toplamasına yardımcı olmak için çeşitli metrik türleri sunar. Bu metrik türleri, Prometheus'un ölçüm değerlerini toplamak, saklamak ve analiz etmek için kullandığı veri yapılarıdır. Prometheus'un temel metrik türleri şunlardır:

1. Counter (Sayaç): Counter, sadece artabilecek veya sıfırlanabilecek (ör. uygulama yeniden başlatıldığında) bir değeri temsil eder. Genellikle istek sayısı, hata sayısı ve tamamlanan işlemler gibi olayların sayısını ölçmek için kullanılır.
2. Gauge (Gösterge): Gauge, herhangi bir değere sahip olabilen ve artırılabilir veya azaltılabilir bir değeri temsil eder. Göstergeler, örneğin bellek kullanımı, CPU kullanımı veya bağlantı sayısı gibi anlık değerlerin ölçülmesinde kullanılır.
3. Histogram (Histogram): Histogram, gözlemlenen değerlerin belirli aralıklara göre sayısını toplayarak değerlerin dağılımını ölçer. Bu metrik türü, ölçüm süresi, gecikme veya istek boyutu gibi değerlerin dağılımını ölçmek için kullanılır. Histogram, gözlemlenen değerlerin toplam sayısı, toplam değeri ve her bir aralık için değer sayısı içeren üç metriği otomatik olarak sağlar.
4. Summary (Özet): Summary, gözlemlenen değerlerin sayısı, toplam değeri ve isteğe bağlı olarak belirli yüzdelik dilimler (ör. P50, P90, P99) gibi istatistikleri sağlar. Özetler, istek süresi veya işlem süresi gibi ölçümlerin istatistiksel özetlerini elde etmek için kullanılır.

{% embed url="https://prometheus.io/docs/concepts/metric_types/" %}

Bu temel metrik türlerini kullanarak, Prometheus Client Libraries ile entegre olan uygulamalarınız ve hizmetlerinizden anlamlı ve kullanışlı ölçümler toplayabilirsiniz. Bu ölçümler, performansı izlemek, hataları ve sorunları tespit etmek ve uygun ölçekleme ve optimizasyon kararları almak için kullanılabilir.



Prometheus metrik türlerinin kullanımı, metrik türüne bağlı olarak çeşitli yöntemlere sahiptir. İşte temel metrik türleri için kullanılabilir yöntemler:

1. Counter (Sayaç):
   * `inc()`: Sayaç değerini 1 artırır.
   * `add(value)`: Sayaç değerini belirtilen `value` kadar artırır. `value` pozitif olmalıdır.
2. Gauge (Gösterge):
   * `inc()`: Gösterge değerini 1 artırır.
   * `dec()`: Gösterge değerini 1 azaltır.
   * `add(value)`: Gösterge değerini belirtilen `value` kadar artırır. `value` hem pozitif hem de negatif olabilir.
   * `sub(value)`: Gösterge değerini belirtilen `value` kadar azaltır. `value` hem pozitif hem de negatif olabilir.
   * `set(value)`: Gösterge değerini belirtilen `value` olarak ayarlar.
3. Histogram (Histogram):
   * `observe(value)`: Gözlemlenen değeri histograma ekler ve uygun aralık sayacını artırır.
4. Summary (Özet):
   * `observe(value)`: Gözlemlenen değeri özete ekler ve istatistiksel değerlerin hesaplanması için kullanılır.

Bu yöntemler, belirtilen metrik türü için uygun olan işlemleri gerçekleştirir ve metriklerin Prometheus tarafından alınabilmesi için uygun veri yapılarını günceller.



{% hint style="info" %}
Histogram, genellikle süreler veya boyutlar gibi sürekli değerlerin dağılımını ölçmek için kullanılır. Önceden tanımlanmış aralıklar (veya "bucket" denilen) belirlenir ve her gözlem için uygun aralığın sayacı artırılır.

Örneğin, bir web sunucusunda istek sürelerini ölçmek istediğinizi düşünelim. Histograma şu şekilde aralıklar belirleyebiliriz: 0-100 ms, 100-200 ms, 200-300 ms vb. Bir istek 150 ms sürerse, 100-200 ms aralığındaki sayacı artırırız.
{% endhint %}





