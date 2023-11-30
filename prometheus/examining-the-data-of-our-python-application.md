# 🐍 Examining the data of our Python application

```python
import http.server
from prometheus_client import start_http_server # For Prometheus

APP_PORT = 8000
METRICS_PORT = 8001 # For Prometheus

class HandleRequests(http.server.BaseHTTPRequestHandler):

    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(bytes("<html><head><title>First Application</title></head><body style='color: #333; margin-top: 30px;'><center><h2>Welcome to our first Prometheus-Python application.</center></h2></body></html>", "utf-8"))
        self.wfile.close()

if __name__ == "__main__":
    start_http_server(METRICS_PORT) # For Prometheus
    server = http.server.HTTPServer(('10.90.0.144', APP_PORT), HandleRequests)
    server.serve_forever()

```

Yukarıdaki uygulama, Prometheus metriklerini kullanarak bir HTTP sunucusu başlatan basit bir uygulamadır. Uygulamanın detaylarına inecek olursak;



1. İlk olarak, gerekli modüller ve kütüphaneler içe aktarılır:
   * `http.server`: Bu modül, temel bir HTTP sunucusu oluşturmak için kullanılır.
   * `prometheus_client`: Prometheus metriklerini yayınlamak için kullanılan bir kütüphanedir.
2. Ardından, uygulama ve metrikler için iki farklı port numarası belirlenir:
   * `APP_PORT`: Uygulamanın dinleyeceği port numarası (8000).
   * `METRICS_PORT`: Prometheus metriklerinin yayınlanacağı port numarası (8001).
3. `HandleRequests` adında bir sınıf tanımlanır. Bu sınıf, `http.server.BaseHTTPRequestHandler` sınıfından türetilmiştir ve gelen HTTP isteklerini işlemek için kullanılır. Şu anda sadece `GET` isteklerini ele alacak şekilde tanımlanmıştır.
4. `do_GET` metodunda, sunucu, başarılı bir yanıt olarak `200` durum kodunu gönderir. Daha sonra yanıtın içeriğine HTML metni ekler ve bağlantıyı kapatır. Bu HTML metni, kullanıcıya "Welcome to our first Prometheus-Python application." mesajını gösterir.
5. `__main__` içinde, metrikleri toplamak ve yayınlamak için Prometheus istemcisini başlatırız: `start_http_server(METRICS_PORT)`.
6. Daha sonra, HTTP sunucusunu başlatırız. Sunucunun IP adresi (`10.90.0.144`) ve uygulama portu (`APP_PORT`) ile `http.server.HTTPServer` sınıfından bir nesne oluştururuz. Bu nesneye, istekleri işlemek için `HandleRequests` sınıfını sağlarız.
7. Son olarak, `server.serve_forever()` ile sunucuyu sürekli çalışır durumda tutarız. Bu sayede sunucu, gelen isteklere sürekli olarak yanıt verebilir.

Özetle, bu uygulama, temel bir HTTP sunucusu başlatır ve Prometheus istemcisi kullanarak metrikleri toplar ve yayınlar.

```bash
/usr/bin/python3 app.py
```

Uygulamamızı çalıştırıyoruz.

{% hint style="info" %}
`prometheus_client` paketi, uygulamalarımızda Prometheus metriklerini toplamamızı ve sunmamızı sağlayan bir kütüphanedir.
{% endhint %}

.

.

.

Ardından python uygulamamızı, prometheus arayüzünden metriklerini incelemek ve olası problemleri tespit etmek için, Uygulamamızın kodunda expose ettiğimiz metrik portunu prometheus.yml dosyasına target olarak ekliyoruz.

```yaml
  - job_name: "prom_python_app"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ["10.90.0.144:8001"]
```

Ardından prometheus servisini yeniden başlatıp, arayüzden verinin kontrolünü sağlıyoruz.

<figure><img src="../.gitbook/assets/WhatsApp Image 2023-04-05 at 15.01.39.jpeg" alt=""><figcaption></figcaption></figure>



#### Counter Metrics

```python
import http.server
import random
from prometheus_client import start_http_server, Counter # For prometheus and counter metrics

REQUEST_COUNT = Counter('app_requests_count_istekler', 'total app http request count',['app_name', 'endpoint']) # Prometheus for counter metrics
RANDOM_COUNT = Counter('app_rastgele_count','increment counter by random value') # Prometheus for counter metrics

APP_PORT = 8000
METRICS_PORT = 8001 # for Prometheus

class HandleRequests(http.server.BaseHTTPRequestHandler):

    def do_GET(self):
        REQUEST_COUNT.labels('prom_python_app', self.path).inc() # Prometheus for counter metrics
        random_val = random.random()*10 # Prometheus for counter metrics
        RANDOM_COUNT.inc(random_val) # Prometheus for counter metrics

        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(bytes("<html><head><title>First Application</title></head><body style='color: #333; margin-top: 30px;'><center><h2>Welcome to our first Prometheus-Python application.</center></h2></body></html>", "utf-8"))
        self.wfile.close()

if __name__ == "__main__":
    start_http_server(METRICS_PORT)  # for Prometheus
    server = http.server.HTTPServer(('10.90.0.144', APP_PORT), HandleRequests)
    server.serve_forever()
```

Kodumuza, gelen istekleri inceleyebilmek için, bazı satırları ekliyoruz. Bu satırların açıklamaları şu şekilde,&#x20;

1. `random` modülü içe aktarıldı. Bu modül, rastgele sayılar üretmek için kullanılır.
2. Prometheus istemcisinden `Counter` metriği içe aktarıldı. Counter, artan bir değeri temsil eden bir metrik türüdür.
3. İki adet Counter metriği tanımlandı:
   * `REQUEST_COUNT`: Uygulamanın aldığı HTTP isteklerinin sayısını takip eder. Bu metrik, 'app\_name' ve 'endpoint' etiketleri ile etiketlendi.
   * `RANDOM_COUNT`: Rastgele bir değerle artırılan bir sayaçtır.
4. `do_GET` metodunda, iki metrik güncellenir:
   * İlk olarak, `REQUEST_COUNT` metriği etiketlerle birlikte artırılır (`inc()` fonksiyonu ile). Bu, uygulamanın adı ve istek yolu ile etiketlenmiş her HTTP isteği için sayaç değerini bir artırır.
   * Daha sonra, `random.random()` fonksiyonu kullanılarak 0 ile 1 arasında rastgele bir sayı üretiliyor. Bu sayıyı 10 ile çarparak 0 ile 10 arasında rastgele bir sayı elde ediyoruz. Bu değer `random_val` adlı değişkene atanıyor.
   * Ardından, `RANDOM_COUNT.inc(random_val)` satırında, `RANDOM_COUNT` adında bir Prometheus Counter nesnesi bulunuyor. Counter, artan sayıları tutmak için kullanılır ve genellikle bir uygulamada gerçekleşen belirli olayların sayısını izlemek için kullanılır. Bu örnekte, `RANDOM_COUNT` adlı Counter nesnesine `inc()` fonksiyonu ile `random_val` değişkeninde tutulan rastgele değeri ekliyoruz. Bu, Counter nesnesinin değerini `random_val` kadar artırır.

<figure><img src="../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

Yeni eklenen kod parçaları sayesinde, uygulamanın aldığı HTTP istek sayısını ve rastgele değerlerle artan bir sayacı takip etmek mümkün hale gelmiştir. Bu metrikler, Prometheus tarafından toplanarak uygulamanın performansı ve işlem sayısı hakkında daha fazla bilgi sağlar.

Uygulamamıza ilgili satırları ekledikten sonra tekrar start ediyoruz ve prometheus ara yüzünde yeni verileri inceliyoruz.

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

Böylelikle uygulamamıza yapılan isteklerin miktarını görebiliyoruz.

.

.

.

#### Gauge Metrics

<pre class="language-python"><code class="lang-python">import http.server
import random
import time
from prometheus_client import start_http_server, Gauge

REQUEST_INPROGRESS = Gauge('app_requests_inprogress','number of application requests in progress')
REQUEST_LAST_SERVED = Gauge('app_last_served', 'Time the application was last served.')

APP_PORT = 8000
METRICS_PORT = 8001

class HandleRequests(http.server.BaseHTTPRequestHandler):

    @REQUEST_INPROGRESS.track_inprogress()
    # track_inprogress() yöntemi, Prometheus'un Gauge sınıfının bir yöntemidir.
<strong>    # Bu, her seferinde bir istek alındığında ve do_GET fonksiyonu çalıştırıldığında, REQUEST_INPROGRESS Gauge metriğinin değeri otomatik olarak artırılır. 
</strong><strong>    # İşlem tamamlandığında ve fonksiyon sona erdiğinde ise değer otomatik olarak azaltılır. 
</strong><strong>    # Bu, şu anda işlenmekte olan isteklerin sayısını takip etmeye ve uygulamanın ne kadar yük altında olduğunu anlamaya yardımcı olur.
</strong>    def do_GET(self):
        time.sleep(5)
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(bytes("&#x3C;html>&#x3C;head>&#x3C;title>First Application&#x3C;/title>&#x3C;/head>&#x3C;body style='color: #333; margin-top: 30px;'>&#x3C;center>&#x3C;h2>Welcome to our first Prometheus-Python application.&#x3C;/center>&#x3C;/h2>&#x3C;/body>&#x3C;/html>", "utf-8"))
        self.wfile.close()
        REQUEST_LAST_SERVED.set_to_current_time()
        # set_to_current_time() fonksiyonu, Prometheus'un Gauge sınıfının bir yöntemidir. 
        # Bu yöntem, Gauge metriğini otomatik olarak mevcut zamanla günceller (Unix epoch formatında, saniye cinsinden). 
        # Bu sayede, uygulama süresince en son işlenen isteğin zamanını takip edebilir ve uygulamanın ne kadar süreyle aktif olduğunu görebilirsiniz.

if __name__ == "__main__":
    start_http_server(METRICS_PORT)
    server = http.server.HTTPServer(('10.90.0.144', APP_PORT), HandleRequests)
    server.serve_forever()
</code></pre>

Gauge metrikleri, bir değeri artırabilen, azaltabilen veya belirli bir değere ayarlayabilen metriklerdir. Bu kod parçasında iki Gauge metrik tanımlanmıştır:



1. REQUEST\_INPROGRESS: Bu metrik, şu anda işlenmekte olan uygulama isteklerinin sayısını temsil eder. 'app\_requests\_inprogress' adıyla ve 'number of application requests in progress' açıklamasıyla tanımlanmıştır.

```python
REQUEST_INPROGRESS = Gauge('app_requests_inprogress','number of application requests in progress')
```

2. REQUEST\_LAST\_SERVED: Bu metrik, uygulamanın en son ne zaman hizmet verdiğini gösterir. 'app\_last\_served' adıyla ve 'Time the application was last served.' açıklamasıyla tanımlanmıştır.

```python
REQUEST_LAST_SERVED = Gauge('app_last_served', 'Time the application was last served.')
```

İlk Gauge metriği, `REQUEST_INPROGRESS`, `do_GET` fonksiyonu için bir dekoratör olarak kullanılır. Bu sayede, fonksiyonun başında ve sonunda otomatik olarak Gauge değeri güncellenir. Bu, işlem süresi boyunca kaç isteğin olduğunu takip etmeye yardımcı olur.

```python
@REQUEST_INPROGRESS.track_inprogress()
def do_GET(self):
    ...
```

İkinci Gauge metriği, `REQUEST_LAST_SERVED`, `do_GET` fonksiyonunun sonunda kullanılır. Uygulamanın en son ne zaman hizmet verdiğini belirlemek için şu anki zamanı bu metriğe ayarlar.

```python
REQUEST_LAST_SERVED.set_to_current_time()
```

Bu ölçümler sayesinde, uygulamanın performansını ve kullanımını daha iyi anlayabilir ve gerektiğinde iyileştirmeler yapabilirsiniz.



<figure><img src="../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption><p><code>time() - app_last_served</code> PromQL sorgusu, şu anki zaman ile <code>app_last_served</code> adlı Gauge metriğinin değeri arasındaki farkı hesaplamak için kullanılır. Bu sorgu, uygulamanın en son isteği işleyip hizmet verdiği zamanın ne kadar önce olduğunu belirlemeye yardımcı olur.</p></figcaption></figure>

.

.

.

#### Summary Metrics

```python
import http.server
import time
from prometheus_client import start_http_server, Summary

REQUEST_RESPOND_TIME = Summary('app_response_latency_seconds', 'Response latency in seconds')

APP_PORT = 8000
METRICS_PORT = 8001

class HandleRequests(http.server.BaseHTTPRequestHandler):

    @REQUEST_RESPOND_TIME.time()
    def do_GET(self):
        time.sleep(6)
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(bytes("<html><head><title>First Application</title></head><body style='color: #333; margin-top: 30px;'><center><h2>Welcome to our first Prometheus-Python application.</center></h2></body></html>", "utf-8"))
        self.wfile.close()


if __name__ == "__main__":
    start_http_server(METRICS_PORT)
    server = http.server.HTTPServer(('10.90.0.144', APP_PORT), HandleRequests)
    server.serve_forever()
```

Yukarıda summary metriğini tanımladığımız kod bloğu mevcut. Uygulama kodumuza eklediğimiz 2 farklı satır mevcut. Bu satırları açıklamak gerekirse;

1. `Summary` metriği tanımlama:

```python
REQUEST_RESPOND_TIME = Summary('app_response_latency_seconds', 'Response latency in seconds')
```

Bu satırda, `Summary` tipinde bir metrik nesnesi oluşturuyoruz. `Summary` metriği, gözlem süresi boyunca olayların sürelerinin veya boyutlarının istatistiksel özetini sağlar. Burada, `app_response_latency_seconds` adında bir metrik tanımlıyoruz ve bu metriğin açıklaması olarak "Response latency in seconds" kullanıyoruz.

2. `time()` dekoratörünü kullanma:

```python
@REQUEST_RESPOND_TIME.time()
```

`time()` dekoratörü, Python'daki dekoratörlerle ilgilidir. Dekoratörler, fonksiyonların veya sınıf metotlarının davranışını değiştirmek veya genişletmek için kullanılır. Bu örnekte, `time()` dekoratörü `do_GET` metodu üzerinde kullanılarak, metot çalıştığında süre ölçümü yapılır ve sonuçlar `REQUEST_RESPOND_TIME` metriğine kaydedilir.

Dekoratörler, fonksiyonun veya metotun üzerinde `@` işareti ile belirtilir. Bu sayede, dekoratörün arkasındaki fonksiyon veya metot, dekoratörün sağladığı ek özelliklerle çalışır. Bu durumda, `time()` dekoratörü `do_GET` metodunun çalışma süresini ölçer ve bu süreyi `REQUEST_RESPOND_TIME` metriğine ekler.

```python
    @REQUEST_RESPOND_TIME.time()
    def do_GET(self):
        ...
```

Özetle, `REQUEST_RESPOND_TIME = Summary(...)` satırıyla, süre ölçümlerini tutmak için bir `Summary` metriği tanımlıyoruz. `@REQUEST_RESPOND_TIME.time()` satırıyla ise, bu metriği kullanarak `do_GET` metodunun çalışma süresini ölçüyoruz. Bu sayede, uygulamanın yanıt süresi hakkında istatistiksel özetler elde ederiz.

{% hint style="info" %}
`@REQUEST_RESPOND_TIME.time()` kod parçacığı, asıl olarak süre ölçümünü yapmak ve bu süreyi `REQUEST_RESPOND_TIME` metriğine kaydetmekle ilgilidir. Bu dekoratör, kodun üzerine yerleştirildiği `do_GET` metodu çalıştığında, metodu çalıştıran işlemi otomatik olarak ölçer ve bu süreyi `REQUEST_RESPOND_TIME` metriğine ekler.

Sonuç olarak, bu dekoratör sayesinde uygulamanın yanıt süresi ölçümleri, Prometheus tarafından toplanarak analiz edilebilir ve izlenebilir hale gelir
{% endhint %}

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Kodumuza 6 saniye gecikmesi için "sleep" eklediğimiz için, ilk isteğin response süresi "6 sn" olarak gözüküyor. Fakat bu değişken sürekli toplanarak ilerlediği için, ortalama response süresini incelememiz için bize yardımcı olmaz.

Ortalama yanıt süresini hesaplamak için,&#x20;

```promql
rate(app_response_latency_seconds_sum[5m]) / rate(app_response_latency_seconds_count[5m])

```

1. `rate(app_response_latency_seconds_sum[5m])`: Son 5 dakika içindeki `app_response_latency_seconds_sum` metriğinin değerinin artış hızını hesaplar. Bu, son 5 dakika boyunca toplam sürelerin ne kadar hızla arttığını gösterir.
2. `rate(app_response_latency_seconds_count[5m])`: Son 5 dakika içindeki `app_response_latency_seconds_count` metriğinin değerinin artış hızını hesaplar. Bu, son 5 dakika boyunca toplam istek sayısının ne kadar hızla arttığını gösterir.
3. Son olarak, bu iki değeri böleriz.

<figure><img src="../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

Bu bölme işlemi, son 5 dakika içindeki ortalama yanıt süresini verir. Ortalama yanıt süresi, toplam sürelerin artış hızının, istek sayısının artış hızına oranı olarak hesaplanır. Bu sayede, son 5 dakikada gerçekleşen isteklerin ortalama süresini görebilirsiniz.



{% hint style="info" %}
`rate` fonksiyonu şu şekilde çalışır:

1. `rate` fonksiyonu, belirtilen zaman aralığı (örneğin, son 5 dakika) boyunca metrik değerinin zaman serisindeki veri noktalarını alır.
2. İki ardışık veri noktası arasındaki farkı alarak, her bir veri noktası arasındaki değişimi hesaplar.
3. Bu değişimlerin toplamını alır ve belirtilen zaman aralığına böler (örneğin, 5 dakika). Bu işlem, zaman aralığı boyunca metrik değerindeki ortalama değişimi hesaplar.



Örneğin, `rate(app_response_latency_seconds_count[5m])` sorgusu için:

1. Son 5 dakikadaki `app_response_latency_seconds_count` metriği için veri noktalarını alır.
2. İki ardışık veri noktası arasındaki farkı alarak, her bir veri noktası arasındaki değişimi hesaplar.
3. Bu değişimlerin toplamını alır ve 5 dakikaya böler. Bu işlem, son 5 dakika içinde işlenen istek sayısının ortalama artış hızını hesaplar.



Sonuç olarak, `rate(app_response_latency_seconds_count[5m])` sorgusu, son 5 dakika içinde işlenen istek sayısının ortalama artış hızını döndürür.&#x20;
{% endhint %}

.

.

.

#### Histogram Metrics

```promql
import http.server
import time
from prometheus_client import start_http_server, Histogram

REQUEST_RESPOND_TIME = Histogram('app_response_latency_seconds', 'Response latency in seconds', buckets=[0.1,0.5,1,2,3,4,5,10])

APP_PORT = 8000
METRICS_PORT = 8001

class HandleRequests(http.server.BaseHTTPRequestHandler):

    @REQUEST_RESPOND_TIME.time()
    def do_GET(self):
        time.sleep(1)
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(bytes("<html><head><title>First Application</title></head><body style='color: #333; margin-top: 30px;'><center><h2>Welcome to our first Prometheus-Python application.</center></h2></body></html>", "utf-8"))
        self.wfile.close()
        REQUEST_RESPOND_TIME.observe(time_taken)


if __name__ == "__main__":
    start_http_server(METRICS_PORT)
    server = http.server.HTTPServer(('10.90.0.144', APP_PORT), HandleRequests)
    server.serve_forever()
```

Yukarıdaki kod parçacığı, Prometheus için Histogram metrik türü kullanarak uygulamanın HTTP isteklerine verdiği yanıt sürelerini ölçmektedir. Histogram metrikleri, belirli bir değer aralığında meydana gelen olayların sayısını ölçen ve bu olayları belirli "kova" (bucket) adı verilen aralıklara göre gruplandıran metriklerdir.

Kod parçacığında, Histogram metriği şu şekilde tanımlanmaktadır:

```python
REQUEST_RESPOND_TIME = Histogram('app_response_latency_seconds', 'Response latency in seconds', buckets=[0.1,0.5,1,2,3,4,5,10])
```

* `app_response_latency_seconds`: Histogram metriğinin adıdır.
* `'Response latency in seconds'`: Histogram metriğinin açıklamasıdır.
* `buckets=[0.1,0.5,1,2,3,4,5,10]`: Histogram için tanımlanan kova sınırlarıdır. Bu, yanıt sürelerini şu aralıklara göre gruplandıracaktır: <= 0.1s, <= 0.5s, <= 1s, <= 2s, <= 3s, <= 4s, <= 5s, <= 10s ve > 10s.

Histogram metriği kullanarak yanıt sürelerini ölçmek için, `HandleRequests` sınıfındaki `do_GET` metodunda histogram metriğini şu şekilde kullanılıyor:

```python
@REQUEST_RESPOND_TIME.time()
```

Bu dekoratör, `do_GET` metodunun çalışma süresini ölçer ve bu süreyi `REQUEST_RESPOND_TIME` histogramına göre ilgili kovalara dağıtır.

Sonuç olarak, bu kod parçacığı, bir web sunucusu başlatır ve gelen HTTP isteklerine yanıt verir. Aynı zamanda, Prometheus için bir Histogram metrik türü kullanarak yanıt sürelerini ölçer ve bu süreleri belirli kova aralıklarına göre gruplandırır. Bu sayede, uygulamanın yanıt sürelerini daha iyi anlayabilir ve performansını izleyebilirsiniz.

{% hint style="info" %}
`app_response_latency_seconds_bucket{le="+Inf"}` ifadesi, Prometheus tarafından histogram metriği için otomatik olarak oluşturulan özel bir kova (bucket) etiketini temsil eder. Bu özel kova, belirtilen kova sınırlarının üstünde olan tüm ölçüm değerlerini toplar.

`le` etiketi "less than or equal to" (eşit veya küçük) anlamına gelir ve `+Inf` (artı sonsuz) değeri, bu kovanın belirtilen tüm diğer kova sınırlarının üstünde olan değerleri toplayacağını gösterir.

Örneğin, histogram metriği için kova sınırları şu şekilde tanımlanmış olsaydı: `buckets=[0.1,0.5,1,2,3,4,5,10]`, `app_response_latency_seconds_bucket{le="+Inf"}` kovası, 10 saniyeden fazla süren tüm yanıt sürelerini toplayacak.
{% endhint %}

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

Soru 1 => bütün bucket değerleri eşit ilerliyor. Tüm istekler nasıl aynı anda bütün bucketlara eşit oluyor?

Cevap => Bu durum, örnek uygulamanızdaki tüm isteklerin 2.0 saniyeden daha kısa sürede tamamlandığını göstermektedir. Histogram metriğinin çalışma şekli nedeniyle, herhangi bir istek süresi, o süreden daha büyük veya eşit olan tüm `le` değerlerine sahip bucket'lara dahil edilir.

Örneğin, bir isteğin süresi 1.5 saniye olsun. Bu durumda, histogram metriği şu şekilde güncellenir:

* 2.0 saniyeden daha kısa olduğu için `app_response_latency_seconds_bucket{le="2.0"}` değerini artırır.
* 5.0 saniyeden daha kısa olduğu için `app_response_latency_seconds_bucket{le="5.0"}` değerini artırır.
* 10.0 saniyeden daha kısa olduğu için `app_response_latency_seconds_bucket{le="10.0"}` değerini artırır.
* Sonsuz saniyeden daha kısa olduğu için `app_response_latency_seconds_bucket{le="+Inf"}` değerini artırır.

Eğer tüm istekler 2.0 saniyeden daha kısa sürede tamamlanırsa, bu durumda tüm bu bucket'lar eşit değerlerle artacaktır. Bu, gözlemlediğiniz durumdur. İstek sürelerinin dağılımını daha iyi görmek için, uygulamanızdaki `time.sleep()` fonksiyonuyla rastgele bekleme sürelerini değiştirebilirsiniz. Örneğin, süreleri 0.1 ile 10 arasında rastgele seçerseniz, farklı bucket'larda farklı sayılar görmeye başlarsınız.



