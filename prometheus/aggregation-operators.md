# ✍ Aggregation Operators

Prometheus'ta, verileri işlemek ve analiz etmek için kullanılan bir dizi operatör vardır. Bu operatörlerden biri de aggregation (toplama) operatörleridir. Toplama operatörleri, birden fazla zaman serisi verisi içinden toplama, ortalama alma, maksimum veya minimum değeri bulma gibi işlemleri gerçekleştirmek için kullanılır.

1. sum(): Belirtilen zaman aralığı boyunca belirtilen metriklerin toplamını hesaplar.
2. avg(): Belirtilen zaman aralığı boyunca belirtilen metriklerin ortalamasını hesaplar.
3. min(): Belirtilen zaman aralığı boyunca belirtilen metriklerin en küçük değerini hesaplar.
4. max(): Belirtilen zaman aralığı boyunca belirtilen metriklerin en büyük değerini hesaplar.
5. count(): Belirtilen zaman aralığı boyunca belirtilen metriklerin sayısını hesaplar.
6. stdvar(): Belirtilen zaman aralığı boyunca belirtilen metriklerin standart sapmasını hesaplar.
7. stdvarp(): Belirtilen zaman aralığı boyunca belirtilen metriklerin örneklem standart sapmasını hesaplar.
8. topk(): Belirtilen zaman aralığı boyunca belirtilen metriklerin en yüksek k değerini döndürür.
9. bottomk(): Belirtilen zaman aralığı boyunca belirtilen metriklerin en düşük k değerini döndürür.

Bu aggregation operatörleri, Prometheus kullanıcıları tarafından sık sık kullanılır ve verilerin analizi ve görselleştirilmesinde önemli bir rol oynarlar.

#### Kullanımları,

```promql
sum(prometheus_http_requests_total)
```

<figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

Yukarıdaki sorgu, Prometheus'ta "prometheus\_http\_requests\_total" adında bir metriğin toplam sayısını hesaplar. Bu metrik, Prometheus sunucusunun kendisi tarafından toplanır ve HTTP isteklerinin toplam sayısını tutar. Bu sorgu, bir zaman aralığı belirtilmediği sürece tüm zaman serisi verileri kapsar ve bu nedenle sunucunun başlatıldığından bu yana geçen tüm süre boyunca toplam HTTP istek sayısını döndürür.



```promql
sum(prometheus_http_requests_total) by (code)
```

<figure><img src="../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

Yukarıdaki sorgu, Prometheus'ta "prometheus\_http\_requests\_total" adlı bir metriğin farklı HTTP yanıt kodlarına göre gruplandırılmış toplam sayısını hesaplar. Bu sorguda `by` kelimesi kullanılarak, `code` etiketi ile gruplama yapılır. Bu sorgu, her bir kod için ayrı ayrı HTTP istek sayısını hesaplayacaktır.



```promql
topk(3, sum(node_cpu_seconds_total) by (mode))
```



<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Yukarıdaki sorgu, Prometheus veritabanında toplanan `node_cpu_seconds_total` metrik verilerini kullanarak, CPU kullanımını modlara (user, system, idle, iowait, irq, softirq) göre gruplayarak, toplam CPU zamanlarının en yüksek üç değerini getirmek için kullanılır. `topk()` fonksiyonu, sıralama işlevi sağlar. İlk parametre olarak, sorguda kaç sonuç döndürüleceği belirtilir. Bu sorguda `topk(3)` ifadesi kullanılmıştır. Bu, en yüksek 3 CPU zaman değerini getirecektir.





```promql
bottomk(3, sum(node_cpu_seconds_total) by (mode))
```

<figure><img src="../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

Yukarıdaki sorgu, Prometheus veritabanındaki `node_cpu_seconds_total` metrik verilerini kullanarak, CPU kullanımını modlara (user, system, idle, iowait, irq, softirq) göre gruplandırarak, toplam CPU zamanlarının en düşük üç değerini getirmek için kullanılır. `bottomk()` fonksiyonu, sıralama işlevi sağlar. İlk parametre olarak, sorguda kaç sonuç döndürüleceği belirtilir. Bu sorguda `bottomk(3)` ifadesi kullanılmıştır. Bu, en düşük 3 CPU zaman değerini getirecektir.





```promql
max(node_cpu_seconds_total)
```

<figure><img src="../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

Yukarıdaki sorgu, Prometheus veritabanındaki `node_cpu_seconds_total` metrik verilerini kullanarak, tüm sunuculara (instance) ait **maksimum** CPU kullanım süresini getirmek için kullanılır.

`max()` fonksiyonu, tüm etiket değerleri aynı olan ölçümlerden en yüksek değeri getirir. Bu sorguda `max(node_cpu_seconds_total)` ifadesi kullanılarak, tüm CPU zamanlarının en yüksek değeri hesaplanır.





```promql
min(node_cpu_seconds_total)
```

Yukarıdaki sorgu, Prometheus veritabanındaki `node_cpu_seconds_total` metrik verilerini kullanarak, tüm sunuculara (instance) ait **minimum** CPU kullanım süresini getirmek için kullanılır.

`min()` fonksiyonu, tüm etiket değerleri aynı olan ölçümlerden en düşük değeri getirir. Bu sorguda `min(node_cpu_seconds_total)` ifadesi kullanılarak, tüm CPU zamanlarının en düşük değeri hesaplanır.



```promql
count(node_cpu_seconds_total)
```

Yukarıdaki sorgu, Prometheus veritabanındaki `node_cpu_seconds_total` metrik verilerini kullanarak, tüm sunuculara (instance) ait CPU kullanım süresi ölçümlerinin sayısını hesaplamak için kullanılır. `count()` fonksiyonu, ölçümlerin sayısını hesaplar.&#x20;

Bu sorguda `count(node_cpu_seconds_total)` ifadesi kullanılarak, ölçüm serilerindeki ölçümlerin sayısı hesaplanır.

