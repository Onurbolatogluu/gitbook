# 🐐 Examining the data of our GO application

#### Counter Metrics

```go
package main

import (
        "net/http"
        "fmt"
        "log"
        "github.com/gorilla/mux"
        "github.com/prometheus/client_golang/prometheus"
        "github.com/prometheus/client_golang/prometheus/promhttp"
        "github.com/prometheus/client_golang/prometheus/promauto"
)

var
        REQUEST_COUNT = promauto.NewCounter(prometheus.CounterOpts{
                Name: "go_app_requests_count",
                Help: "Total App HTTP Requests count.",
        })

func main() {
        // Start the application
        startMyApp()
}

func startMyApp() {
        router := mux.NewRouter()
        router.HandleFunc("/birthday/{name}", func(rw http.ResponseWriter, r *http.Request) {
                        vars := mux.Vars(r)
                        name := vars["name"]
                        greetings := fmt.Sprintf("Happy Birthday %s :)", name)
                        rw.Write([]byte(greetings))

                        REQUEST_COUNT.Inc()
        }).Methods("GET")

        log.Println("Starting the application server...")
        router.Path("/metrics").Handler(promhttp.Handler())
        http.ListenAndServe(":8000", router)
```

Yukarıdaki Go uygulamasında, uygulamanın alınan HTTP isteklerinin sayısını izlemeyi sağlıyoruz. Bu, uygulamanın ne kadar trafiği işlediğini anlamamıza yardımcı olur.

Uygulamada, "go\_app\_requests\_count" adında bir Prometheus Counter metriği oluşturuyoruz ve bu metriği kullanarak, `/birthday/{name}` yoluna gelen HTTP isteklerinin toplam sayısını izliyoruz.



1. İlk olarak, Prometheus'un client\_golang kütüphanesini kullanarak uyumlu metrikler oluşturmak için gerekli paketleri içe aktarıyoruz:

```go
import (
    ...
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
    "github.com/prometheus/client_golang/prometheus/promauto"
)
```

{% hint style="info" %}
1. `"github.com/prometheus/client_golang/prometheus"`: Bu paket, Prometheus metriklerini tanımlamak ve işlemek için kullanılan temel yapıları içerir. Counter, Gauge, Histogram ve Summary gibi metrik türlerini tanımlamak için bu paketin sağladığı yapıları kullanırız.
2. `"github.com/prometheus/client_golang/prometheus/promhttp"`: Bu paket, Prometheus metriklerini sunan HTTP handler'larını sağlar. Bu paket sayesinde, uygulamanızdaki `/metrics` yolunu tanımlayarak Prometheus'un metrikleri alabilmesi için bir HTTP endpoint oluşturabilirsiniz. Bu paketin en önemli işlevi olan `promhttp.Handler()` işlevi, metrikleri döndüren bir HTTP handler oluşturur.
3. `"github.com/prometheus/client_golang/prometheus/promauto"`: Bu paket, metriklerin otomatik olarak kaydedilmesini sağlar. Bu sayede, uygulamanızdaki metriklerinizi tanımladıktan sonra, Prometheus'un bunları otomatik olarak kaydetmesini ve toplamasını sağlar. Bu paketin `NewCounter`, `NewGauge`, `NewHistogram` ve `NewSummary` gibi işlevleri, metrikleri otomatik olarak kaydeden nesneler oluşturmak için kullanılır.



Bu modüller, Prometheus'un Go uygulamalarında kullanılabilmesi için gerekli temel yapıları ve işlevleri sağlar. İçe aktarılan bu paketler sayesinde, Go uygulamanızda Prometheus metrikleri tanımlayabilir, bu metrikleri bir HTTP endpoint üzerinden sunabilir ve Prometheus tarafından otomatik olarak toplanmalarını sağlayabilirsiniz.
{% endhint %}

2. Daha sonra, uygulamanın HTTP isteklerinin sayısını izlemek için bir Prometheus Counter metriği oluşturuyoruz. Counter, artan sayıları temsil eden ve sadece artırılabilen bir metrik türüdür:

```go
var REQUEST_COUNT = promauto.NewCounter(prometheus.CounterOpts{
    Name: "go_app_requests_count",
    Help: "Total App HTTP Requests count.",
})
```

Burada, "go\_app\_requests\_count" adında bir Counter metriği oluşturuyoruz ve "Total App HTTP Requests count." açıklamasını ekliyoruz.

3. `/birthday/{name}` yolundaki HTTP isteklerini işleyen işlevde, her istek için Counter metriğini 1 artırıyoruz:

```go
func(rw http.ResponseWriter, r *http.Request) {
    ...
    REQUEST_COUNT.Inc()
}
```

`REQUEST_COUNT.Inc()` işlevi, metriği 1 birim artırmak için kullanılır.

4. Son olarak, Prometheus'un uygulama metriklerini alabilmesi için `/metrics` yolunu tanımlıyoruz:

```go
router.Path("/metrics").Handler(promhttp.Handler())
```

`promhttp.Handler()` işlevi, Prometheus metriklerini döndüren bir HTTP handler oluşturur. Bu sayede Prometheus, uygulamadan metrikleri alarak izleme ve uyarılar için kullanabilir.

Özetle, yukarıdaki Go uygulaması, Prometheus ile entegre olarak çalışmak üzere tasarlanmıştır. Uygulamanın HTTP istek sayısını izleyen bir Counter metriği oluşturulmuş ve bu metriği Prometheus'a sunmak için `/metrics` yolunu kullanıyoruz.



* Yukarıdaki uygulamayı çalışır duruma getirdikten sonra, Go uygulamamızı "Prometheus" a dahil ediyoruz. Aşağıdaki parametreleri prometheus yaml dosyamıza tanımladıktan sonra, prometheus servisimizi restart ediyoruz.

```yaml
  - job_name: "prom_go_app"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ["10.90.0.144:8000"]
```

* Gördüğünüz gibi go uygulamamız target olarak eklenmiş durumda.

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Prometheus dashboard kullanarak, go uygulamamıza gelen toplan request miktarını görüntüleyebiliriz.

<figure><img src="../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

.

.

.

#### Gauge Metrics

```go
package main

import (
        "net/http"
        "fmt"
        "log"
        "time"
        "github.com/gorilla/mux"
        "github.com/prometheus/client_golang/prometheus"
        "github.com/prometheus/client_golang/prometheus/promhttp"
        "github.com/prometheus/client_golang/prometheus/promauto"
)

var
        REQUEST_INPROGRESS = promauto.NewGauge(prometheus.GaugeOpts{
                Name: "go_app_requests_inprogress",
                Help: "Number of application requests in progress",
        })

func main() {
        // Start the application
        startMyApp()
}

func startMyApp() {
        router := mux.NewRouter()
        router.HandleFunc("/birthday/{name}", func(rw http.ResponseWriter, r *http.Request) {
                        REQUEST_INPROGRESS.Inc()
                        vars := mux.Vars(r)
                        name := vars["name"]
                        greetings := fmt.Sprintf("Happy Birthday %s :)", name)
                        time.Sleep(5 * time.Second)
                        rw.Write([]byte(greetings))

                        REQUEST_INPROGRESS.Dec()
        }).Methods("GET")

        log.Println("Starting the application server...")
        router.Path("/metrics").Handler(promhttp.Handler())
        http.ListenAndServe(":8000", router)
```

Bu uygulamada, şu anda işlem gören HTTP isteklerinin sayısını izlemeyi sağlıyoruz.

1. İlk olarak, uygulamanın şu anda işlem gören HTTP isteklerinin sayısını izlemek için bir Prometheus Gauge metriği oluşturuyoruz. Gauge, artan ve azalan sayıları temsil eden ve hem artırılabilen hem de azaltılabilen bir metrik türüdür:

```go
var REQUEST_INPROGRESS = promauto.NewGauge(prometheus.GaugeOpts{
    Name: "go_app_requests_inprogress",
    Help: "Number of application requests in progress",
})
```

Burada, "go\_app\_requests\_inprogress" adında bir Gauge metriği oluşturuyoruz ve "Number of application requests in progress" açıklamasını ekliyoruz.

2. `/birthday/{name}` yolundaki HTTP isteklerini işleyen işlevde, her istek için Gauge metriğini 1 artırıyoruz:

```go
func(rw http.ResponseWriter, r *http.Request) {
    REQUEST_INPROGRESS.Inc()
    ...
}
```

`REQUEST_INPROGRESS.Inc()` işlevi, metriği 1 birim artırmak için kullanılır.

3. İstek işlendikten sonra, Gauge metriğini 1 azaltıyoruz:

```go
func(rw http.ResponseWriter, r *http.Request) {
    ...
    REQUEST_INPROGRESS.Dec()
}
```

`REQUEST_INPROGRESS.Dec()` işlevi, metriği 1 birim azaltmak için kullanılır.

4. Son olarak, Prometheus'un uygulama metriklerini alabilmesi için `/metrics` yolunu tanımlıyoruz:

```go
router.Path("/metrics").Handler(promhttp.Handler())
```

`promhttp.Handler()` işlevi, Prometheus metriklerini döndüren bir HTTP handler oluşturur. Bu sayede Prometheus, uygulamadan metrikleri alarak izleme ve uyarılar için kullanabilir.

Özetle, yukarıdaki Go uygulamasında, şu anda işlem gören HTTP isteklerinin sayısını izlemeyi sağlıyoruz. Uygulamada "go\_app\_requests\_inprogress" adında bir Gauge metriği oluşturuyoruz ve bu metriği kullanarak, şu anda işlem gören isteklerin sayısını izliyoruz.

<figure><img src="../.gitbook/assets/image (92).png" alt=""><figcaption><p>1 adet işlenen istek mevcut.</p></figcaption></figure>

.

.

.

#### Summary Metrics

```go
package main

import (
        "net/http"
        "fmt"
        "log"
        "time"
        "github.com/gorilla/mux"
        "github.com/prometheus/client_golang/prometheus"
        "github.com/prometheus/client_golang/prometheus/promhttp"
        "github.com/prometheus/client_golang/prometheus/promauto"
)

var
        REQUEST_RESPOND_TIME = promauto.NewSummaryVec(prometheus.SummaryOpts{
                Name: "go_app_response_latency_seconds",
                Help: "Response latency in seconds.",
        }, []string{"path"})


func routeMiddleware(next http.Handler) http.Handler {

        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                start_time := time.Now()
                route := mux.CurrentRoute(r)
                path, _ := route.GetPathTemplate()

                next.ServeHTTP(w, r)
                time_taken := time.Since(start_time)
                REQUEST_RESPOND_TIME.WithLabelValues(path).Observe(time_taken.Seconds())

        })

}

func main() {
        // Start the application
        startMyApp()
}

func startMyApp() {
        router := mux.NewRouter()
        router.HandleFunc("/birthday/{name}", func(rw http.ResponseWriter, r *http.Request) {
                vars := mux.Vars(r)
                name := vars["name"]
                greetings := fmt.Sprintf("Happy Birthday %s :)", name)

                time.Sleep(5 * time.Second)
                rw.Write([]byte(greetings))

        }).Methods("GET")

        router.Use(routeMiddleware)
        log.Println("Starting the application server...")
        router.Path("/metrics").Handler(promhttp.Handler())
        http.ListenAndServe(":8000", router)
}
```

Bu uygulamada, uygulamanın HTTP isteklerine yanıt verme süresini ölçmeyi sağlıyoruz.

1. İlk olarak, yanıt verme süresini izlemek için bir Prometheus SummaryVec metriği oluşturuyoruz. Summary, veri örneklerinin sayısı, toplamı ve istatistiksel dağılımını temsil eden bir metrik türüdür:

```go
var REQUEST_RESPOND_TIME = promauto.NewSummaryVec(prometheus.SummaryOpts{
    Name: "go_app_response_latency_seconds",
    Help: "Response latency in seconds.",
}, []string{"path"})
```

Burada, "go\_app\_response\_latency\_seconds" adında bir SummaryVec metriği oluşturuyoruz ve "Response latency in seconds." açıklamasını ekliyoruz. Ayrıca, "path" etiketi ile isteklerin hangi yola yapıldığını belirtiyoruz.

2. `routeMiddleware` adında bir middleware işlevi tanımlıyoruz. Bu işlev, bir HTTP isteğinin süresini ölçmeye ve isteğin tamamlanma süresini Summary metriğine kaydetmeye yardımcı olur:

```go
func routeMiddleware(next http.Handler) http.Handler {
    ...
}
```

3. Middleware işlevi içinde, isteğin başlangıç zamanını kaydediyoruz ve isteğin yolunu alıyoruz:

```go
start_time := time.Now()
route := mux.CurrentRoute(r)
path, _ := route.GetPathTemplate()
```

4. İstek işlendikten sonra, geçen süreyi hesaplayarak Summary metriğine kaydediyoruz:

```go
next.ServeHTTP(w, r)
time_taken := time.Since(start_time)
REQUEST_RESPOND_TIME.WithLabelValues(path).Observe(time_taken.Seconds())
```

`REQUEST_RESPOND_TIME.WithLabelValues(path).Observe(time_taken.Seconds())` işlevi, isteğin tamamlanma süresini ölçen metriği kaydetmek için kullanılır.

5. Son olarak, Prometheus'un uygulama metriklerini alabilmesi için `/metrics` yolunu tanımlıyoruz:

```go
router.Path("/metrics").Handler(promhttp.Handler())
```

`promhttp.Handler()` işlevi, Prometheus metriklerini döndüren bir HTTP handler oluşturur. Bu sayede Prometheus, uygulamadan metrikleri alarak izleme ve uyarılar için kullanabilir.

Özetle, yukarıdaki Go uygulamasında, uygulamanın HTTP isteklerine yanıt verme süresini ölçmeyi sağlıyoruz.  Uygulamada "go\_app\_response\_latency\_seconds" adında bir SummaryVec metriği oluşturuyoruz ve bu metriği kullanarak, yanıt verme sürelerini izliyoruz.



<figure><img src="../.gitbook/assets/image (100).png" alt=""><figcaption><p>Yukarıdaki Prometheus sorgusu, uygulamanın ortalama yanıt süresini hesaplamak için kullanılır. İki rate fonksiyonunun bölümü şeklinde yapılan bu sorgu, belirli bir zaman aralığındaki ortalama yanıt süresini elde etmenizi sağlar. Bu örnekte, 5 dakikalık (<code>5m</code>) bir zaman aralığı kullanılmaktadır. Bu sorgu sayesinde, uygulamanın belirli bir zaman dilimindeki ortalama yanıt süresini (saniye .cinsinden) elde edebilirsiniz.</p></figcaption></figure>

`REQUEST_RESPOND_TIME` metriğine veri kaydetme işlemi, `startMyApp()` fonksiyonu içerisinde değil, `routeMiddleware` adlı middleware işlevi içerisinde gerçekleştirilir. `routeMiddleware` işlevi, HTTP isteklerinin sürelerini ölçen ve bu süreleri `REQUEST_RESPOND_TIME` metriğine kaydeden bir işleve sahiptir.

`startMyApp()` fonksiyonu içinde şu kod satırı ile middleware'i kullanıma alıyoruz:

```go
router.Use(routeMiddleware)
```

`router.Use(routeMiddleware)` ile `routeMiddleware` işlevini uygulamanın tüm yollarına uygularız. Bu sayede her bir istek, `routeMiddleware` işlevi içindeki kodla işlenir ve isteğin süresi `REQUEST_RESPOND_TIME` metriğine kaydedilir.

`routeMiddleware` işlevi içinde, isteğin başlangıç zamanını alır, isteğin gerçekleştiği yolun şablonunu alır, ardından isteğin işlenmesini sürdürür ve isteğin tamamlanma süresini ölçer:

```go
start_time := time.Now()
route := mux.CurrentRoute(r)
path, _ := route.GetPathTemplate()

next.ServeHTTP(w, r)
time_taken := time.Since(start_time)
```

Son olarak, geçen süreyi hesapladıktan sonra, `REQUEST_RESPOND_TIME` metriğine kaydediyoruz:

```go
REQUEST_RESPOND_TIME.WithLabelValues(path).Observe(time_taken.Seconds())
```

Bu şekilde, `REQUEST_RESPOND_TIME` metriği, uygulamanın HTTP isteklerine yanıt verme süresini ölçer ve Prometheus bu metrikler üzerinden uygulamanın performansını takip edebilir.



.

.

.



#### Histogram Metrics

```go
package main

import (
        "net/http"
        "fmt"
        "log"
        "time"
        "github.com/gorilla/mux"
        "github.com/prometheus/client_golang/prometheus"
        "github.com/prometheus/client_golang/prometheus/promhttp"
        "github.com/prometheus/client_golang/prometheus/promauto"
)

var
        REQUEST_RESPOND_TIME = promauto.NewHistogramVec(prometheus.HistogramOpts{
                Name: "go_app_response_latency_seconds",
                Help: "Response latency in seconds.",
        }, []string{"path"})


func routeMiddleware(next http.Handler) http.Handler {

        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                start_time := time.Now()
                route := mux.CurrentRoute(r)
                path, _ := route.GetPathTemplate()

                next.ServeHTTP(w, r)
                time_taken := time.Since(start_time)
                REQUEST_RESPOND_TIME.WithLabelValues(path).Observe(time_taken.Seconds())

        })

}

func main() {
        // Start the application
        startMyApp()
}

func startMyApp() {
        router := mux.NewRouter()
        router.HandleFunc("/birthday/{name}", func(rw http.ResponseWriter, r *http.Request) {
                vars := mux.Vars(r)
                name := vars["name"]
                greetings := fmt.Sprintf("Happy Birthday %s :)", name)

                time.Sleep(2 * time.Second)
                rw.Write([]byte(greetings))

        }).Methods("GET")

        router.Use(routeMiddleware)
        log.Println("Starting the application server...")
        router.Path("/metrics").Handler(promhttp.Handler())
        http.ListenAndServe(":8000", router)
}
```

Yukarıdaki Go uygulaması Prometheus için metrikler toplar ve histogram metriği kullanır. Histogram, istek sürelerinin dağılımını temsil eder ve belirli bir süre içinde ne kadar süre harcandığını gösterir. Şimdi Prometheus ile ilgili kısımları açıklayalım.

1. Histogram metriği tanımlama:

```go
var REQUEST_RESPOND_TIME = promauto.NewHistogramVec(prometheus.HistogramOpts{
        Name: "go_app_response_latency_seconds",
        Help: "Response latency in seconds.",
}, []string{"path"})
```

Bu kod parçası, `go_app_response_latency_seconds` adlı bir histogram metriği oluşturur. Bu metrik, yanıt gecikme sürelerini saniye cinsinden gösterir ve etiket olarak "path" kullanır.

2. Middleware işlevi:

```go
func routeMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start_time := time.Now()
        route := mux.CurrentRoute(r)
        path, _ := route.GetPathTemplate()

        next.ServeHTTP(w, r)
        time_taken := time.Since(start_time)
        REQUEST_RESPOND_TIME.WithLabelValues(path).Observe(time_taken.Seconds())
    })
}
```

`routeMiddleware` işlevi, her HTTP isteğinin başlangıç ve bitiş zamanını ölçer ve bu süreyi `REQUEST_RESPOND_TIME` histogram metriğine kaydeder. İşlev ayrıca, hangi yolun süreyi aldığını belirtmek için "path" etiketini kullanır.

3. Middleware'i kullanma:

```go
router.Use(routeMiddleware)
```

Bu kod parçası, `routeMiddleware` işlevini uygulamamız için kullanılabilir hale getirir. Böylece, her bir istek için yanıt süreleri ölçülür ve metriğe kaydedilir.

4. Prometheus metriklerini sunma:

```go
router.Path("/metrics").Handler(promhttp.Handler())
```

Bu satır, uygulamanın `/metrics` yolunda Prometheus metriklerini sunar. Bu sayede, Prometheus bu yolu tarayarak uygulamanın performans metriklerini toplayabilir.

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

...

...

...<br>



