# 🔦 promQL Data Types

**Instant vector**

Instant vector sorguları, zaman serilerinin belirli bir anlık durumunu sorgulamak için kullanılır. Bu sorgular, sorgulanan zaman damgasına en yakın değerlerini döndürür. Örneğin, `up{job="node_exporter"}` ifadesi, "node\_exporter" işinin son anındaki durumunu ifade eder ve bir instant vector olarak sorgulanmış olur.

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

#### Range Vector

PromQL'deki Range Vector, belli bir zaman aralığı boyunca gözlemlenen ölçümler kümesidir. Range Vector'lar, bir zaman serisi veritabanında depolanan zaman serilerinin belirli bir aralığını sorgulamak için kullanılır.

Range Vector'lar, PromQL'de ölçümlerin zaman serilerini nasıl filtrelediğinizi belirlemenize olanak tanır. Örneğin, belirli bir zaman aralığındaki CPU kullanımı veya ağ trafiği gibi ölçümleri sorgulayabilirsiniz. Range Vector'lar, PromQL'de çok yaygın bir kullanıma sahiptir ve çoğu sorguda kullanılan temel bir özelliktir.

<figure><img src="../.gitbook/assets/image (85).png" alt=""><figcaption><p>Bu sorgular <code>[range vector selector][time range]</code> şeklinde oluşturulur. Range Vector Selector, sorgulanacak olan zaman serisini belirlerken, Time Range, sorgulanacak zaman aralığını belirler.</p></figcaption></figure>

`up{job="node_exporter"}[5m]` sorgusu bir Range Vector sorgusuna örnektir. Bu sorgu, "node\_exporter" işindeki son 5 dakika boyunca "up" metrik etiketine sahip tüm zaman serilerini içeren bir Range Vector döndürür. Bu sorgu, son 5 dakika boyunca belirli bir süre içindeki "up" metrik değerleri hakkında bilgi edinmek için kullanılabilir.

`avg_over_time(node_memory_Active_bytes[2m])/1024/1024/1024`

Yukarıdaki sorgu, son 2 dakika içindeki `node_memory_Active_bytes` metriklerinin ortalama değerini(`avg_over_time`) alır ve sonucu gigabayt cinsinden döndürür.

{% hint style="info" %}
`avg_over_time()` fonksiyonu, bu aralık vektörünün ortalama değerini hesaplar. Son olarak, `1024/1024/1024` ifadeleri, byte cinsinden hesaplanan ortalama değeri gigabayt cinsinden ifade etmek için kullanılır.
{% endhint %}

#### Scalar

Yalnızca bir sayısal değer döndüren bir ifadedir. Bu ifadeler, bir aralık veya anlık vektörü yerine, doğrudan bir sayıya karşılık gelirler.

Örneğin, bir hedefteki toplam bellek miktarını gösteren bir skaler ifade şu şekilde olabilir:

`node_memory_MemTotal_bytes`

<figure><img src="../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>



Yukarıdaki ifade, bir skaler ifadesidir çünkü yalnızca tek bir sayısal değer döndürür ve herhangi bir zaman serisi verisine sahip değildir.

`scalar(avg(node_memory_Active_bytes) / 1024 / 1024 / 1024)`

Yukarıdaki sorgu, `node_memory_Active_bytes` metriğinin ortalamasını gigabyte cinsinden scalar olarak döndürür.

#### String

PromQL'de `string` veri tipi, karakter dizilerini temsil etmek için kullanılan bir veri tipidir. Veri tipi olarak, dizi işlemleri veya matematiksel işlemler gibi sayısal işlemler gerçekleştirilemez. Bunun yerine, karakter dizileriyle ilgili özel işlemler gerçekleştirilir. Özetle, `string` ifadeler, metin verileriyle çalışmak için kullanılır ve sayısal işlemler için uygun değildir.&#x20;







