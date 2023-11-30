# 🔢 Binary Operators

Prometheus Query Language'deki iki anahtar kelime arasındaki işlemler, PromQL binary operators olarak bilinir. Binary operators, iki değer arasında işlem yapmak için kullanılır ve sonucu bir skaler değere dönüştürür.

Binary operators, birden fazla metrik arasında işlem yapmak veya verileri karşılaştırmak için kullanılır ve PromQL'de sıklıkla kullanılır.

#### Binary Operators;

* Arithmetic binary operator
* Comparison binary operators
* Logical/set binary operators

#### Arithmetic binary operator

Arithmetic binary operators, iki sayı arasında yapılan matematiksel işlemleri ifade eden binary operatörlerdir. Bu operatörler, iki sayıyı toplama, çıkarma, çarpma veya bölme işlemlerinde kullanılarak sonucu bir skaler değere dönüştürür.

Aşağıda arithmetic binary operatorlar hakkında maddeler halinde açıklamalar bulabilirsiniz:

<table><thead><tr><th width="152">Operatör</th><th>Açıklama</th></tr></thead><tbody><tr><td>+</td><td>İki sayıyı veya vektörleri toplar ve sonucu bir skaler değer olarak döndürür.</td></tr><tr><td>-</td><td>İki sayıyı veya vektörlerden birini diğerinden çıkarır ve sonucu bir skaler değer olarak döndürür.</td></tr><tr><td>*</td><td>İki sayıyı veya vektörleri çarpar ve sonucu bir skaler değer olarak döndürür.</td></tr><tr><td>/</td><td>İki sayıyı veya bir vektörü diğerine bölerek oranını hesaplar ve sonucu bir skaler değer olarak döndürür.</td></tr><tr><td>%</td><td>İki sayının modülünü hesaplar ve sonucu bir skaler değer olarak döndürür.</td></tr><tr><td>**</td><td>İki sayının üstünü hesaplar ve sonucu bir skaler değer olarak döndürür.</td></tr></tbody></table>

Arithmetic binary operatorlar, matematiksel hesaplamaların yanı sıra veri analizi, görselleştirme ve programlama dillerinde kullanılır. PromQL'de kullanıldıklarında, metric verileri işlemek için kullanılırlar ve sonucu skaler bir değer olarak döndürürler.

Bu operatörler arasında öncelik sıralaması bulunur, öncelik sıralaması şu şekildedir:

1. `**`
2. `*`, `/`, `%`
3. `+`, `-`

Yani, öncelik sıralamasına uygun olarak işlemler yapılır. Örneğin, 2 + 3 \* 4 işleminde çarpma işlemi önce yapılır ve sonuç olarak 14 elde edilir.

#### Example;

```promql
node_memory_Active_bytes/8
```

Bu sorgu, bir sistemdeki "aktif bellek" miktarını bayt cinsinden ifade eden `node_memory_Active_bytes` metriğinin 8'e bölünmesi işleminden oluşur.



#### Comparison binary operators

PromQL'deki Comparison Binary Operatorlar, karşılaştırma işlemleri yapmak için kullanılan operatörlerdir. Bu operatörler, iki değeri karşılaştırır ve sonucu true (doğru) veya false (yanlış) olarak döndürür.&#x20;

<table><thead><tr><th width="154">Operatör</th><th>Açıklama</th></tr></thead><tbody><tr><td>==</td><td>İki değerin eşit olup olmadığını kontrol eder. Eşitse true, değilse false döndürür.</td></tr><tr><td>!=</td><td>İki değerin eşit olmadığını kontrol eder. Eşit değilse true, eşitse false döndürür.</td></tr><tr><td>></td><td>Sol taraftaki değerin sağ taraftakinden büyük olup olmadığını kontrol eder. Büyükse true, değilse false döndürür.</td></tr><tr><td>>=</td><td>Sol taraftaki değerin sağ taraftakinden büyük veya eşit olup olmadığını kontrol eder. Büyük veya eşitse true, değilse false döndürür.</td></tr><tr><td>&#x3C;</td><td>Sol taraftaki değerin sağ taraftakinden küçük olup olmadığını kontrol eder. Küçükse true, değilse false döndürür.</td></tr><tr><td>&#x3C;=</td><td>Sol taraftaki değerin sağ taraftakinden küçük veya eşit olup olmadığını kontrol eder. Küçük veya eşitse true, değilse false döndürür.</td></tr></tbody></table>

Bu operatörler, özellikle metrik verileri sorgularken sıklıkla kullanılır. Örneğin, CPU kullanımı, bellek kullanımı, ağ trafiği vb. gibi metriklerin belirli bir eşik değerinin üzerine çıkıp çıkmadığını kontrol etmek için bu operatörler kullanılabilir.

#### Example;

```promql
node_procs_running>6
```

Bu sorgu, bir düğümde (node) çalışan işlemlerin sayısının 6'dan büyük olup olmadığını kontrol eder.

Sorguda yer alan `>` operatörü, sağ tarafındaki değerin sol tarafındakinden büyük olup olmadığını kontrol eder. Bu durumda, sorgu, bir düğümde (node) çalışan işlemlerin sayısının 6'dan büyük olup olmadığını kontrol eder. Sonuç olarak, eğer çalışan işlemlerin sayısı 6'dan büyükse, sorgu true (doğru) değerini döndürür. Aksi halde, sorgu false (yanlış) değerini döndürür.



#### Logical/set binary operators

PromQL'de, mantıksal/set ikili operatörleri, iki veya daha fazla ifadeyi karşılaştırmak ve sonuçları birleştirmek için kullanılan operatörlerdir. PromQL'de yaygın olarak kullanılan bazı mantıksal/set ikili operatörler aşağıda açıklanmaktadır:

1. `and`: İki veya daha fazla ifadenin "ve" mantıksal işlemi yapılır. Sonuç, tüm ifadelerin doğru (true) olması durumunda true (doğru) değerini döndürür.
2. `or`: İki veya daha fazla ifadenin "veya" mantıksal işlemi yapılır. Sonuç, en az bir ifade doğru (true) olduğunda true (doğru) değerini döndürür.
3. `unless`: İlk ifade ikinci ifadeye eşit veya küçük olduğunda true (doğru) değerini döndürür. Aksi halde, false (yanlış) değerini döndürür.
4. `group_left`, `group_right`: Bu operatörler, etiket çiftleri üzerindeki işlemlerde kullanılır ve birleştirme işlemi için kullanılır. `group_left` operatörü sol etiket çiftine, `group_right` operatörü ise sağ etiket çiftine uygulanır.

#### Example;

```promql
prometheus_http_requests_total or promhttp_metric_handler_requests_total
```

Bu sorgu, `prometheus_http_requests_total` adlı metriği ve `promhttp_metric_handler_requests_total` adlı metriği birleştirir. Her iki metrik de HTTP isteklerini sayar ve prometheus server'ı üzerinde gerçekleşen farklı isteklerin sayısını hesaplar. `or` operatörü, her iki metrikten elde edilen sonuçları birleştirir ve sonuç olarak iki metriğin ölçtüğü toplam HTTP istek sayısını gösterir.

```promql
node_filesystem_avail_bytes{fstype="ext4"} and node_filesystem_size_bytes{fstype="ext4"}

```

<figure><img src="../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

Bu "and" sorgusu olarak bilinen iki farklı metrik etiketi arasında bir kesişim bulmaya çalışır. Sonuç olarak, bu sorgu "ext4" dosya sistemi türüne sahip disk bölümlerinin mevcut ve toplam bayt sayılarının kesişimini verir ve <mark style="color:red;">**sonuçlar her zaman**</mark><mark style="color:red;">** **</mark><mark style="color:red;">**`and`**</mark><mark style="color:red;">** **</mark><mark style="color:red;">**ifadesinin ilk vektöründen gelir.**</mark>

Özetle,

```promql
node_filesystem_size_bytes{fstype="ext4"} and node_filesystem_avail_bytes{fstype="ext4"}

```

Yukarıdaki sorgunun çıktısı, ext4 dosya sistem türüne sahip olan bir cihazın toplam dosya sistemi boyutunu vermektedir.

```promql
node_filesystem_avail_bytes{fstype="ext4"} and node_filesystem_size_bytes{fstype="ext4"}
```

Yukarıdaki sorgunun çıktısı, ext4 dosya sistem türüne sahip olan bir cihazın kullanılabilir dosya sistemi alanını (byte cinsinden) göstermektedir.



```promql
node_filesystem_avail_bytes{mountpoint="/"} and node_filesystem_size_bytes{mountpoint="/"}
```

Bu sorgu, `node_filesystem_avail_bytes` ve `node_filesystem_size_bytes` metriklerini `mountpoint="/"`, yani kök dizin için filtreler. `and` operatörü ile bu iki metriğin tam olarak eşleşen etiket kümelerine sahip olan öğelerini seçer ve diğer öğeleri atar. <mark style="color:red;">**Sonuçlar her zaman**</mark><mark style="color:red;">** **</mark><mark style="color:red;">**`and`**</mark><mark style="color:red;">** **</mark><mark style="color:red;">**ifadesinin ilk vektöründen gelir.**</mark>



### More Examples;

```promql
prometheus_http_requests_total*2
```

`prometheus_http_requests_total` adlı metriğin her bir örneğindeki değerleri 2 ile çarpar.



```promql
node_cpu_seconds_total>500
```

Bu sorgu, `node_cpu_seconds_total` adlı metrikte toplam CPU kullanım süresini saniye cinsinden tutan değerlerin 500'den büyük olan örneklerini seçer. Bu sorgu, özellikle yüksek CPU kullanımı olan işlemleri izlemek için kullanılabilir. `node_cpu_seconds_total` metriği, her bir işlem için kullanım süresi miktarını kaydeder. `>500` ifadesi, sorgunun sadece 500 saniyeden fazla CPU kullanımı olan işlemleri seçmesini sağlar.



