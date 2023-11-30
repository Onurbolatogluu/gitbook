# 💀 ignoring and on

#### ignoring

PromQL’de `ignoring` anahtar kelimesi, iki vektörü birleştirirken belirli etiketleri dikkate almamak için kullanılır. Bu, iki vektörün diğer etiketlerinin tam olarak eşleştiği durumlarda bile sonuçların döndürülmesini sağlar.

`ignoring` anahtar kelimesi, iki vektörü birleştirirken belirli etiketleri dikkate almamak için kullanılır. Bu, iki vektörün diğer etiketlerinin tam olarak eşleştiği durumlarda bile sonuçların döndürülmesini sağlar.

```promql
prometheus_http_requests_total and ignoring(handler) promhttp_metric_handler_requests_total
```

`prometheus_http_requests_total and ignoring(handler) promhttp_metric_handler_requests_total` sorgusu `prometheus_http_requests_total` ve `promhttp_metric_handler_requests_total` metriklerini birleştirir ve `handler` etiketini dikkate almaz. `and` operatörü ile bu iki metriğin tam olarak eşleşen etiket kümelerine sahip olan öğelerini seçer ve diğer öğeleri atar. Sonuçlar her zaman `and` ifadesinin ilk vektöründen gelir.

Bu sorgu, Prometheus tarafından işlenen HTTP isteklerinin sayısını döndürebilir. `ignoring(handler)` anahtar kelimesi ile `handler` etiketini dikkate almamak, iki vektörü birleştirirken `handler` etiketinin değerlerini göz ardı eder. Bu, iki vektörün diğer etiketlerinin tam olarak eşleştiği durumlarda bile sonuçların döndürülmesini sağlar.



<figure><img src="../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

Gördüğünüz gibi, sorgunun boş dönmesi etiketlerin (handler etiketi) eşleşmemesinden kaynaklanıyor.

`prometheus_http_requests_total` ve `promhttp_metric_handler_requests_total` ölçüm setleri farklı Prometheus sunucusu örnekleri tarafından sağlanır ve farklı etiketlere sahip olabilirler. Bu nedenle, sorguda kullanılan etiketlerin ölçüm setleriyle eşleşmesi önemlidir.

Örneğin, `prometheus_http_requests_total` ölçüm setinde `handler`, `code` ve `instance`  `job` gibi etiketler bulunurken, `promhttp_metric_handler_requests_total` ölçüm setinde `code` ve `job`  `instance` etiketleri bulunur. Bu durumda `promhttp_metric_handler_requests_total` ölçüm setinde `handler` etiketi olmadığı için `and` operatörü kullanarak `prometheus_http_requests_total` ölçüm setiyle `handler` etiketi eşleşmesi beklenemez.

Sorgu sonuçlarının ölçüm setiyle eşleşmesini sağlamak için, sorgularda kullanılan etiketlerin ölçüm setiyle uyumlu olduğundan emin olmak önemlidir. Özellikle birden fazla ölçüm setiyle çalışırken, etiketlerin doğru şekilde kullanılması sonuçların doğru şekilde yorumlanmasına ve anlaşılmasına yardımcı olur.



<figure><img src="../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

```promql
 prometheus_http_requests_total and ignoring(handler) promhttp_metric_handler_requests_total
```

`prometheus_http_requests_total and ignoring(handler) promhttp_metric_handler_requests_total` sorgusu `prometheus_http_requests_total` ve `promhttp_metric_handler_requests_total` metriklerini birleştirir ve `handler` etiketini dikkate almaz. `and` operatörü ile bu iki metriğin tam olarak eşleşen etiket kümelerine sahip olan öğelerini seçer ve diğer öğeleri atar. **Sonuçlar her zaman and ifadesinin ilk vektöründen gelir.**

`ignoring(handler)` anahtar kelimesi ile `handler` etiketini dikkate almamak, iki vektörü birleştirirken `handler` etiketinin değerlerini göz ardı eder.

#### On

PromQL’de `on` anahtar kelimesi, iki vektörü birleştirirken sadece belirli etiketleri dikkate almak için kullanılır. Bu, iki vektörün diğer etiketlerinin tam olarak eşleşmediği durumlarda bile sonuçların döndürülmesini sağlar.

```promql
prometheus_http_requests_total and on(code) promhttp_metric_handler_requests_total
```

`prometheus_http_requests_total and on(code) promhttp_metric_handler_requests_total` sorgusu `prometheus_http_requests_total` ve `promhttp_metric_handler_requests_total` metriklerini birleştirir ve sadece `code` etiketini dikkate alır. `and` operatörü ile bu iki metriğin tam olarak eşleşen `code` etiketlerine sahip olan öğelerini seçer ve diğer öğeleri atar. Sonuçlar her zaman `and` ifadesinin ilk vektöründen gelir.

`on(code)` anahtar kelimesi ile sadece `code` etiketini dikkate almak, iki vektörü birleştirirken sadece `code` etiketinin değerlerini göz önünde bulundurur. **Bu, iki vektörün diğer etiketlerinin tam olarak eşleşmediği durumlarda  sonuçların döndürülmesini sağlar.**

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

**Examples - 1**

```promql
node_cpu_guest_seconds_total and on(mode) windows_cpu_time_total
```

Bu sorgu, `node_cpu_guest_seconds_total` ölçüm kümesindeki verileri, `windows_cpu_time_total` ölçüm kümesindeki verilerin `mode` etiketine göre eşleşen verileriyle birleştirir.

`on()` ifadesi, etiket eşleştirmesinde kullanılan anahtarları belirtir. Bu sorguda, `mode` etiketi kullanılarak `node_cpu_guest_seconds_total` ve `windows_cpu_time_total` ölçüm kümesi birleştirilir.
