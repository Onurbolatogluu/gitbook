# 🖊 Alıştırma

#### Check if node memory is less than 20% or not.

```promql
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 20

```

Bu PromQL sorgusu, bir düğümün bellek kullanımını yüzde olarak hesaplar.&#x20;

`node_memory_MemAvailable_bytes` metriği düğümde kullanılabilir bellek miktarını gösterirken, `node_memory_MemTotal_bytes` metriği düğümün toplam bellek miktarını gösterir. Bu iki değer birbirine bölünerek bellek kullanımının yüzdesi hesaplanır.&#x20;

Daha sonra bu yüzde değeri 100 ile çarpılır. Son olarak, elde edilen değer 20’den küçükse sorgu doğru döner.



#### Graph the disk read rate.

```promql
rate(node_disk_read_time_seconds_total[1m]) / rate(node_disk_reads_completed_total[1m])
```

Bu PromQL sorgusu, bir düğümün disk okuma işlemlerinin ortalama süresini hesaplar.

`rate` işlevi, belirli bir zaman aralığındaki bir sayaç metriğinin artış hızını hesaplar. Bu sorguda, son 1 dakikadaki `node_disk_read_time_seconds_total` ve `node_disk_reads_completed_total` metriklerinin artış hızı hesaplanır. Daha sonra bu iki değer birbirine bölünerek disk okuma işlemlerinin ortalama süresi hesaplanır.



#### Based on the past 2 hours of data, find out how much disk will fill in next 6 hours.

```promql
predict_linear(node_filesystem_free_bytes{fstype="ext4"}[2h], 6*60*60)/1024/1024/1024
```

Bu sorgu, ext4 dosya sistemi tipine sahip bir disk bölümündeki boş alanın son 2 saat boyunca nasıl değiştiğini hesaplar ve lineer regresyon yöntemini kullanarak bu değişimin 6 saat boyunca devam etmesi durumunda disk bölümünün doluluk oranını tahmin eder.

<figure><img src="../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

Sorgunun ayrıntılı açıklaması şöyledir:

* `node_filesystem_free_bytes{fstype="ext4"}`: ext4 dosya sistemi tipine sahip disk bölümlerinin boş alanını temsil eden time series'leri seçer.
* `[2h]`: Sadece son 2 saat içindeki verileri seçer.
* `predict_linear(node_filesystem_free_bytes{fstype="ext4"}[2h], 6*60*60)`: Son 2 saat içindeki verilere dayanarak, 6 saat boyunca devam etmesi durumunda disk bölümündeki boş alanın azalacağı tahmin edilen oranı hesaplar.
* `/1024/1024/1024`: Byte cinsinden tahmini doluluk oranını GB cinsine dönüştürür.



#### Find out the CPU usage percentage.

```promql
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[10m])) * 100)
```

Bu sorgu, Prometheus gibi bir zaman serisi veritabanında, "node\_cpu\_seconds\_total" metriğinin "idle" modu altında belirtilen süre boyunca **tüm CPU çekirdeklerinde ortalama boşta kalma yüzdesini hesaplar ve 100'den çıkararak işlemci kullanım yüzdesini hesaplar.**&#x20;

&#x20;"idle" modu, CPU'nun boşta kalma süresini temsil eder.&#x20;

"irate(node\_cpu\_seconds\_total{mode="idle"}\[10m])" ifadesi, son 10 dakika boyunca CPU boşta kalma süresindeki değişim oranını hesaplar.

{% hint style="info" %}
&#x20;`irate()` fonksiyonu aralık vektöründeki son iki veri noktasını alır ve bu iki nokta arasındaki artış hızını hesaplar.
{% endhint %}

"avg by(instance)" ifadesi, tüm CPU çekirdeklerinin ortalamasını alır ve her bir çekirdek üzerinde ayrı ayrı hesaplanır.



&#x20;<mark style="color:green;">CPU kullanımının yüzde cinsinden ifade edildiği bir metrik elde etmek için</mark> <mark style="color:green;"></mark><mark style="color:green;">`irate(node_cpu_seconds_total{mode="idle"}[10m])`</mark> <mark style="color:green;"></mark><mark style="color:green;">ifadesi kullanılır. Bu ifade, son 10 dakika boyunca ölçülen CPU kullanımını yüzde cinsinden ifade eder.</mark>

<mark style="color:green;">Ancak, bu ifade sonucunda elde edilen değer doğrudan kullanım yüzdesi değil, kullanım yüzdesinin tamamlayıcısıdır (yani, boşta geçirilen zamanın yüzdesidir). Bu nedenle, hesaplanan değeri doğrudan kullanım yüzdesi olarak kullanmak için, sonuç 100'den çıkarılır. Örneğin, eğer</mark> <mark style="color:green;"></mark><mark style="color:green;">`irate(node_cpu_seconds_total{mode="idle"}[10m])`</mark> <mark style="color:green;"></mark><mark style="color:green;">ifadesi ile elde edilen sonuç %20 ise,</mark> <mark style="color:green;"></mark><mark style="color:green;">`100 - 20 = 80`</mark> <mark style="color:green;"></mark><mark style="color:green;">ile CPU kullanım yüzdesi %80 olarak hesaplanır.</mark>

\
