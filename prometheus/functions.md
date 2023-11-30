# 🧠 Functions

PromQL, veri modelinde ölçümleri işlemek için birçok farklı işlev (function) sunar. PromQL işlevleri, ölçümleri filtrelemek, birleştirmek, gruplandırmak, işlemek ve daha pek çok işlevi yerine getirmek için kullanılır.

* `sum()`: Belirli bir etiket kümesine göre ölçümleri toplar.
* `avg()`: Belirli bir etiket kümesine göre ölçümlerin ortalamasını hesaplar.
* `max()`: Belirli bir etiket kümesine göre ölçümlerin maksimum değerini hesaplar.
* `min()`: Belirli bir etiket kümesine göre ölçümlerin minimum değerini hesaplar.
* `count()`: Belirli bir etiket kümesine göre ölçümlerin sayısını hesaplar.
* `rate()`: Bir ölçüm serisindeki artış oranını hesaplar.
* `irate()`: Ölçüm serisindeki anlık artış hızını hesaplar.
* `delta()`: Bir ölçüm serisindeki değişim miktarını hesaplar.
* `absent()`: Belirli bir etiket kümesine göre ölçüm serisinde herhangi bir verinin eksik olup olmadığını kontrol eder.

#### Kullanımları,

* Rate

```promql
rate(prometheus_http_requests_total[1m])
```

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Yukarıdaki PromQL sorgusu, belirli bir ölçüm serisi için son 1 dakikadaki artış oranını hesaplamak için kullanılır. `prometheus_http_requests_total` ölçüm serisindeki ölçümlerin artış hızını hesaplamak için `rate()` işlevi kullanılır.

`[1m]` ifadesi, sorgu sonucunun son 1 dakikadaki artış hızını göstermesini sağlar. Bu ifade, `rate()` işlevine iletilen ölçüm serisindeki ölçümlerin aralığını belirler. Bu örnekte, son 1 dakikadaki ölçümler hesaplanacaktır.&#x20;

Sonuç olarak, bu sorgu, son 1 dakika boyunca `prometheus_http_requests_total` ölçüm serisindeki ölçümlerin artış hızını hesaplar ve sorgu sonucunda artış hızı olarak ifade eder.&#x20;



* irate

```promql
irate(prometheus_http_requests_total[1m])
```

Bu sorgu, `prometheus_http_requests_total` metriğinin son 1 dakikadaki anlık artış hızını hesaplar. `irate()` fonksiyonu, bir aralık vektöründeki zaman serisi verilerinin anlık artış hızını hesaplar. Bu hesaplama son iki veri noktasına dayanır.

`irate()` fonksiyonu, bir aralık vektöründeki zaman serisi verilerinin anlık artış hızını hesaplar. Bu hesaplama son iki veri noktasına dayanır. Yani, `irate()` fonksiyonu aralık vektöründeki son iki veri noktasını alır ve bu iki nokta arasındaki artış hızını hesaplar.

Örneğin, `irate(prometheus_http_requests_total[1m])` sorgusu, `prometheus_http_requests_total` metriğinin son 1 dakikadaki verilerini alır ve bu verilerin son iki noktası arasındaki artış hızını hesaplar.

{% hint style="info" %}
"rate" fonksiyonunun hesaplamasında belirlenen zaman aralığına göre ("range" değeri) hesaplama yapılır. Bu nedenle, "rate" fonksiyonu sonucu, belirli bir zaman aralığındaki artış oranını hesaplar. "Irate" fonksiyonu ise, son örneklemeden itibaren belirli bir süreyi hesaba katarak sabit bir oran hesaplar. Bu nedenle, "irate" fonksiyonu daha anlık bir artış oranını verirken, "rate" fonksiyonu daha belirli bir zaman aralığındaki artış oranını verir.
{% endhint %}





* changes

```promql
changes(process_start_time_seconds{job="node_exporter"}[1h])
```

<figure><img src="../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

`changes()` fonksiyonu, belirli bir zaman aralığı içindeki ölçüm serisindeki değişiklik sayısını hesaplar. Bu fonksiyon, zaman aralığındaki ölçüm serisindeki herhangi bir artış veya azalışı sayar ve sonucu döndürür.

`process_start_time_seconds{job="node_exporter"}` ifadesi, `node_exporter` işinin başlangıç süresi ölçüm serisini ifade eder. `[1h]` ifadesi, son bir saat içindeki ölçümleri filtreler.

Dolayısıyla `changes(process_start_time_seconds{job="node_exporter"}[1h])` sorgusu, son bir saat içinde `node_exporter` işinin başlangıç süresi ölçüm serisindeki değişiklik sayısını hesaplar ve sonucu döndürür. Bu, `node_exporter` işinin son bir saat içinde kaç kez başladığını veya durdurulduğunu hesaplamak için kullanılır.



* deriv

```promql
deriv(node_network_receive_bytes_total{device="ens160"}[1h])
```

`deriv(node_network_receive_bytes_total{device="ens160"}[1h])` sorgusu, `node_network_receive_bytes_total` metriğini `device="ens160"` etiketi ile filtreler ve son bir saat içindeki değerlerini seçer. Daha sonra `deriv` işlevi bu metrikteki değişiklik oranını hesaplar. Bu sorgu, belirtilen süre boyunca `ens160` interface'in ağ alım bayt sayısındaki değişiklik oranını saniye başına bayt cinsinden gösterir.

Özetle, Örneğin, son bir saat içinde `ens160` interface gelen ağ trafiği byte miktarı 10 MB arttıysa, `deriv(node_network_receive_bytes_total{device="ens160"}[1h])` sorgusu sonucu 10 MB/saniye verir.





* predict\_linear

```promql
predict_linear(node_memory_MemFree_bytes{job="node_exporter"}[1h], 2*60*60)/1024/1024/1024 
```

`predict_linear(node_memory_MemFree_bytes{job="node_exporter"}[1h], 2*60*60)/1024/1024/1024` sorgusu, `node_memory_MemFree_bytes` metriğini `job="node_exporter"` etiketi ile filtreler ve son bir saat içindeki değerlerini seçer. Daha sonra `predict_linear` işlevi bu metrikteki değişiklik oranını kullanarak gelecekteki bir değeri tahmin eder. Bu sorgu, 2 saat sonra boş bellek miktarını gigabayt cinsinden tahmin eder.

`2*60*60` ifadesi 2 saat sonrasını saniye cinsinden ifade eder. `predict_linear` işlevi ikinci parametre olarak gelecekteki bir zamanı saniye cinsinden alır ve bu zaman için bir tahmin yapar. Bu durumda, `2*60*60` ifadesi 2 saat sonrasını saniye cinsinden ifade eder ve `predict_linear` işlevi 2 saat sonra boş bellek miktarını tahmin eder.

{% hint style="info" %}
`2*60*60` ifadesi 2 saat sonrasını saniye cinsinden ifade eder. Bir saat 60 dakikadır ve bir dakika 60 saniyedir. Bu nedenle, bir saat 60 \* 60 = 3600 saniyedir. 2 saat ise 2 \* 3600 = 7200 saniyedir. Dolayısıyla, `2*60*60` ifadesi 2 saat sonrasını saniye cinsinden ifade eder.
{% endhint %}





* max\_over\_time

`max_over_time` işlevi, bir zaman serisi metriğinin belirtilen bir süre boyunca en yüksek değerini döndürür.&#x20;

```promql
max_over_time(node_cpu_seconds_total[1h])
```

`max_over_time(node_cpu_seconds_total[1h])` sorgusu, `node_cpu_seconds_total` metriğinin son bir saat içindeki en yüksek değerini döndürür. Bu sorgu, belirtilen süre boyunca bir sunucunun CPU kullanımının en yüksek değerini gösterir.



* min\_over\_time

```promql
min_over_time(node_cpu_seconds_total[1h]) 
```

`min_over_time(node_cpu_seconds_total[1h])` sorgusu, `node_cpu_seconds_total` metriğinin son bir saat içindeki en düşük değerini döndürür. Bu sorgu, belirtilen süre boyunca bir sunucunun CPU kullanımının en düşük değerini gösterir.



* avg\_over\_time

```promql
avg_over_time(node_cpu_seconds_total[1h])
```

Bu sorgu, 1 saatlik bir zaman dilimi boyunca bir dizi işlemci metriklerinin ortalama değerini hesaplar. "node\_cpu\_seconds\_total" işlemci metrikleri için varsayılan bir Prometheus metrik adıdır ve toplam işlemci süresini saniye cinsinden ölçer. "avg\_over\_time" işlevi, belirtilen bir zaman aralığı boyunca verilerin ortalamasını hesaplar.



* sort

```promql
sort(node_cpu_seconds_total)
```

`sort(node_cpu_seconds_total)` sorgusu, `node_cpu_seconds_total` metriğinin değerlerini artan sıraya göre sıralar.&#x20;





* sort\_desc

```promql
sort_desc(node_cpu_seconds_total) 
```

`sort_desc(node_cpu_seconds_total)` sorgusu, `node_cpu_seconds_total` metriğinin değerlerini azalan sıraya göre sıralar.&#x20;



* time

```promql
time()
```

`time()` sorgusu, PromQL'deki işlevlerden biridir ve sorgunun çalıştırıldığı zamanın tam zaman damgasını (unix timestamp) döndürür.

```promql
time() - process_start_time_seconds{job="node_exporter"}
```

`time() - process_start_time_seconds{job="node_exporter"}` sorgusu, `process_start_time_seconds` metriğindeki bir işlemin ne kadar süredir çalıştığını hesaplamak için kullanılır.

Bu sorgu, `time()` işlevi ile `process_start_time_seconds` işlevi arasındaki zaman farkını hesaplayarak, bir işlemin ne kadar süredir çalıştığını hesaplar. Sonuç olarak, bu sorgu `node_exporter` işleminin çalışma süresini saniye cinsinden döndürür.





* day\_of\_week

```promql
day_of_week()
```

`day_of_week()` PromQL işlevi, bir zaman damgası (timestamp) değerinin haftanın hangi gününe denk geldiğini döndürür.&#x20;

`day_of_week()` işlevi, bir zaman damgasının haftanın kaçıncı gününe denk geldiğini 0'dan 6'ya kadar olan bir sayı olarak ifade eder. 0 Pazar'ı temsil ederken, 1 Pazartesi'yi, 2 Salı'yı vb. temsil eder.



* day\_of\_month

```promql
day_of_month()
```

`day_of_month()` PromQL işlevi, bir zaman damgasının ayın kaçıncı gününe denk geldiğini döndürür.

`day_of_month()` işlevi, bir zaman damgasının ayın kaçıncı gününe denk geldiğini bir tam sayı olarak ifade eder. Örneğin, 1 Ocak tarihi 1, 2 Ocak tarihi 2, vb. şeklinde ifade edilir.

