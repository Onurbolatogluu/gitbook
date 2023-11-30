# 💡 rate functions

`rate()` işlevi, bir zaman serisindeki artış oranını hesaplamak için kullanılır. İşte basit bir örnek:

Diyelim ki, bir uygulamanızda gerçekleşen HTTP isteklerini sayan bir `http_requests_total` adında bir sayaç (counter) metriği var. Bu metrik, zaman içinde artarak, uygulamanıza yapılan toplam istek sayısını temsil eder.

Örnek zaman serisi verileri şu şekilde olsun:

```yaml
http_requests_total = [
  {time: 10:00, value: 100},
  {time: 10:01, value: 120},
  {time: 10:02, value: 140},
  {time: 10:03, value: 160},
  {time: 10:04, value: 180},
  {time: 10:05, value: 200}
]
```

Bu örnekte, `http_requests_total` metriği, her dakikada 20 istek artmaktadır. Şimdi, `rate()` işlevini kullanarak, bu metrikteki artış oranını hesaplayalım.

PromQL sorgusu:

```scss
rate(http_requests_total[1m])
```

Bu sorgu, `http_requests_total` metriğindeki artış oranını 1 dakikalık süre boyunca (`[1m]`) hesaplar. Bu örnek için, her dakikada 20 istek artışı olduğu için, `rate()` işlevinin sonucu 20'dir.

`rate()` işlevi, özellikle sayaç (counter) metriklerinde kullanışlıdır, çünkü zaman içinde artan değerlerin ne kadar hızlı değiştiğini anlamaya yardımcı olur. Bu bilgi, uygulama performansını izlemek ve analiz etmek için önemlidir.
