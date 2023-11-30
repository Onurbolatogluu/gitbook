# 🛠 Liveness Probe & Readiness Probe

### Liveness Probe

Bazı durumlarda, container içerisinde başlatılan uygulama process çalışıyor olsa da, aslında yapması gereken işi yapmıyor olabilir.&#x20;

Örneğin, bir web sayfası yayınlayan podumuz var. İçerisinde httpd tabanlı bir container çalışıyor. Bu pod başlayınca, container içinde ki, httpd deamon ayağa kalkıyor ve web sitemizi sunmaya başlıyor. Fakat bir zaman sonra bu servis takılmaya başlıyor. Bir hatadan dolayı sayfaları sunmaya devam edemiyor. Ama httpd uygulaması, yani servis ayakta. Servis kapanmadı yada çökmedi. Ama esas yapması gereken iş olan, web sitemizi sunma işini yapmıyor.

Bu ve benzeri durumları yaşadığımız zaman ortamda şöyle bir sıkıntı oluşuyor. Eğer uygulama çalışmıyor olsa, yani uygulama kapansa, kubelet bunu tespit ederek, containerı yeniden başlatarak sorunu çözüyor.

Fakat uygulama ayakta olmasına rağmen, yapması gereken işi yapmıyorsa, kubelet bunu, tespit edemiyor. Dolayısıyla containerı yeniden başlatarak, sorunu çözme mekanizması çalışmıyor. Bu problemi Liveness proble ile çözebiliriz.

* Kubelet bir containerın, ne zaman yeniden başlatılacağını bilmek için, Liveness probe kullanır.

Örneğin, Liveness Probe bir uygulamanın çalıştığı ancak ilerleme sağlayamadığı, bir kitlenmeyi yakalayabilir. Böyle bir durumda bir containerı yeniden başlatmak, hatalara rağmen uygulamayı, daha kullanılabilir hale getirmeye yardımcı olabilir.

Liveness probe ile biz container içerisinde, bir komut çalıştırarak, ya da http endpointe istek göndererek ya da bir porta tcp connection açarak uygulamanın doğru çalışıp, çalışmadığını kontrol etme imkanına sahip oluyoruz.&#x20;

Bu sorgulamaların sonucunda, beklediğimiz sonucu alırsak, containerın sağlıklı olduğunu, Eğer cevap beklediğimiz gibi değilse containerın problemli olduğunu tespit edebiliyoruz. Kubelet bu sorgu sonucuna göre aksiyon alabiliyor. Liveness probe tanımlamaları yaml içerisinde LivenessProbe:  parametresi altında tanımlanıyor.

Kullanabileceğimiz 3 Tip Probe mevcut;

1 - httpGet

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

Bu probe seçtiğimizde, şunu diyoruz. Localhost 'a "path" parametresi altında belirlediğimiz path'e, "Port" Parametresinde belirlediğimiz porta httpGet isteği gönder. Cevap olarak,  200-400 arasında bir cevap kodu dönerse, yani cevap alabilirse işler yolundadır. Aksi bir cevap dönerse, hata olduğunu anlayıp, containerı restart et. İsteğimize custom bir http headerı ekleyebiliriz.

InıtıalDelaySeconds ve PeriodSeconds parametreleri, şu işe yarıyor; Bir pod oluşturduğumuzda, container çalışır fakat container içerisindeki, uygulama hemen ayağa kalkmayabilir. Ön hazırlıkları yapması gerekebilir. Bir yerden dosya çekmesi gerekebilir. kısacası uygulama container çalıştıktan sonra, hemen hizmet vermeyebilir. Eğer Liveness probe hemen başlatırsak, daha henüz uygulama hazır olmadığı için hata alacaktır. Ve hemen container restart edilecektir. Dolayısıyla daha tam anlamıyla hizmet sunmadan fail edecek ve sürekli restart olacaktır.

Bunu önlemek adına, initialDelaySeconds parametresini kullanıyoruz. Ve liveness probe 'a şunu diyoruz. Container başladıktan sonra, burada belirttiğimiz süre sonuna kadar bekle ardından sağlık kontrolüne başla.&#x20;

PeriodSeconds ile, bu sağlık kontrolünün kaç saniye aralıklarla yapılacağını belirtiyoruz. Yani container başladı, initialdelaySeconds süresi boyunca bekledi, ardından httpget ile liveness check yapmaya başladı. 1. sorgusunu yaptı cevap olumlu, 2. sorguyu göndermeden Periodseconds da belirlediğimiz süre boyunca bekledi, sonrasında tekrar sorgu yaptı. Bu döngü bu şekilde devam edecek. Sorgular arası kaçar saniye beklemesini istiyorsak, periodSeconds parametresi ile belirtiyoruz.

2 - Exec (Komut yürütme)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

HTTP uygulamalarımızı httpget ile kontrol edebiliyoruz. Ancak uygulamalarımız http uygulaması değilse, bir konsol uygulaması vb. ise. Bunu httpget ile sorgulamamız mümkün değil.&#x20;

Bunun yerine, bir script yazar, o script içerisinde uygulamamıza uygu sorgulama neyse, misal bir dosyanın olup, olmadığını kontrol edebiliriz. Ya da bir başka process 'in çalışıp, çalışmadığını kontrol edebiliriz. Uygulamamıza uygun sağlık kontrolünü seçeriz.

Liveness Probe altında httpGet dışında kullanabileceğimiz probe olan EXEC bize bu imkanı verir. Exec ile shell 'de, komut yada uygulama çalıştırırız. Bu uygulama(komut) bize 0 kodu dönerse, yani olumlu bir cevap verirse, sağlıklı olduğunu anlar. Bunun dışında bir cevap dönerse, problemli olarak işaretler ve container 'ı restart eder. Yukarıda ki örnekte exec probe ile bir komut çalıştırıyoruz. cat komutu ile /tmp/healty dosyasını kontrol ediyoruz. Eğer, tmp dizininde böyle bir dosya varsa, bize bu komutun sonucu 0 dönecektir ve olumlu olacaktır. Böyle bir dosya yoksa, exit code 1 olacak. Yani burada bir dosyanın olup, olmadığını kontrol ederek, uygulamanın sağlıklı çalışıp, çalışmadığını kontrol ediyoruz.

3 - TCPSocket

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

Bazen httpget isteği ve komut çalıştırmak, bir şeyin sağlıklı olup, olmadığını anlamamıza yardımcı olmaz.

Misal bir mySQL veritabanı container çalıştırıyoruz. mysql process 3306 portundan erişilebilir. Biz de TCPSocket ile bu porta bir istek gönderip, eğer cevap alırsak uygulamanın düzgün çalıştığını varsayabiliriz. Olumsuz cevap alırsak, porta erişemezsek container restart edilecek.



### Readiness Probe

Senaryo ;

3 frontend podumuzu oluşturacak bir deploymentımız, bu podlara dış dünyadan erişilebilmesini sağlayacak Load Balancer tipi bir service mevcut. Biz tüm bunları yaml dosyası arayıcılığıyla oluşturduk ve apply ettik. Ve ortamımız çalışmaya başladı. Load Balancer IP adresi üzerinden kullanıcılar bu frontend podlarımıza erişip, web sitemize gelebiliyorlar.

Şimdi gelin deployment objesinde güncelleme yapalım. Diyelim ki, web sitemizin bulunduğu imajı güncelledik ve deploymentımızı yeniden apply ettik. rolling-update stratejisi ile, podlar sırayla devreden çıkarlıp, yeni podlar devreye alındı.

Burada olanları yakından inceleyelim;

Deployment da yeni bir imaj atadığımız zaman kubernetes hemen bu güncel imaj ile pod oluşturarak, bu yeni podları Load Balancer servisine dahil edecek.

Yani oluşturulup, başlatıldığı anda dış dünyadan bu podlara erişilebilinir olacak. Ya bizim uygulamamız ilk başladığı anda, bir yerlere bağlanıp, bazı dosyaları çekiyor ve sonrasında web sitemizi sunmaya başlıyorsa, Yada başka bir nedenden dolayı container başlatılır, başlatılmaz bu uygulama hizmet veremiyorsa,  aradan belirli bir zaman geçip, bazı işlemler yapması gerekiyorsa, Eğer durum bu ise, bu podları oluşturulduğumuz anda, bu podlar LoadBalancer servisinden trafik almaya başladığı için problem çıkacaktır.

Yani pod ayakta içindeki uygulama çalışıyor ancak, web sitemizi sunmaya hazır değil. Dolayısıyla bu pod, LoadBalancer servisine eklenmek için hazır değil. Peki bu pod 'un Load Balancer servisinden trafik almaya hazır olduğunu nasıl anlayacağız? Burada da, yardımımıza Readiness probe yetişiyor.

Kubelet, bir containerın trafiği kabul etmeye hazır olduğunu bilmek için, readiness probeları kullanır. Bir pod, tüm containerlar hazır olduğunda, hazır kabul edilir. Readiness probe sayesinde, bir pod hazır olana kadar, service arkasına eklenmez.

Readiness probelar, aynen liveness probelar gibi oluşturulur. httpGet, Exec, TCPSocket olarak kurgulanabilir.

Biz bir readiness check tanımlarız. Eğer bu readiness check olumlu sonuç dönüyorsa, Pod Service altına eklenir ve trafik alır. Eğer readiness check fail ediyorsa, bu pod service 'den çıkarılır.

Örneğimize dönersek, yeni imajı tanımladık ve yeni podlar oluşturuldu. Podlar çalışmaya başladı. Ama bu noktada load balancer service altına hemen eklenmedi. Podlar içerisinde readiness check mekanizması çalışmaya başladı initialdelayseconds süresi kadar  bekledi, sonra tanımladığımız probe ile kontrolünü yaptı. Olumlu cevap aldığı anda, bu pod service arkasına eklenir ve trafik almaya başlar.

Readiness Check periodSeconds ile belirlenen süre kadar, sürede bir bu kontrolünü yapmaya devam eder. Fail edene kadar service arkasında trafik alır. Eğer fail ederse, pod service endpoint listesinden çıkarılır.&#x20;

Eski podları direkt kapatmak yerine, kullanıcılara hata göstermemek için. Bir pod terminate edileceği zaman. Bu poda yeni trafik gelmemesi için pod service arkasından çıkarılır. Service ile bağlantısı kesilir. Böylece trafik almaz. Fakat kubernetes hemen bu podu kapatmaz. Çünkü yeni trafik gelmese bile, bu pod halen eski istekleri işlemeye devam ediyor olabilir. Bu nedenle, bu işlemleri beklemesi gerekebilir.&#x20;

Pod tanımlarında, terminationGracePeriodSeconds şeklinde bir parametre bulunur. Ve varsayılan olarak değeri 30 saniyedir.  Aksini belirtmediğimiz sürece bu değer kullanılır. Biz veya kubelet podu terminate istediğimiz zaman podun içerisinde çalışan master process ine hemen sigkill sinyali gönderip hemen öldürmez. Bunun yerine sigturn sinyali gönderir yani, yaptığın bir işlem varsa düzgün bir şekilde tamamla, bitince de kendini kapat sinyalini gönderir. Eğer process bu sinyali aldığında eğer ortasında olduğu bir iş varsa, onu yapmayı bırakmaz. O işlemi tamamlar ve bittikten sonra tüm bağlantıları düzgün bir şekilde kapatıp sonrasında da process de kapatılır.&#x20;

terminationGracePeriodSeconds süresi, bu container a düzgün şekilde kendisini kapatabilmesi için atadığımız bir süredir. Eğer process bu süre içerisinde düzgün bir şekilde kapatılırsa sıkıntı olmayacaktır. Fakat kapanmazsa da, bu süre sonunda sigkill sinyali gönderilip, zorla kapatılır. Dolayısıyla uygulamamızın ne kadar süre içerisinde kapanabildiğini bilmemiz o na göre de terminationGracePeriodSeconds süresini düzenlememiz gerekebilir.

{% embed url="https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-terminating-with-grace" %}

Readiness Aşamaları,

1 - Deployment dosyası içerisinde imajımızı güncelledik.

2- Yeni imajda, yeni bir pod oluşturuldu.

3- Yeni pod çalışmaya başladı.

4- Readiness check mekanizması çalışmaya başladı. initialdelayseconds süresi kadar bekledi ve ardından ilk kontrolünü yaptı.

5- Kontrol sonucu olumlu olduğu anda, yeni pod service altına eklendi.

6- Eski pod henüz kapatılmadı, Eğer işlemesi gereken istekler varsa, işlemeye devam ediyor. Ve yeni istekler gelmemesi için service altından çıkarıldı.

7- Eski podun kendini düzgün kapatabilmesi için, sigterm sinyali gönderildi.

8- Eski pod üzerindeki işlemleri tamamladıktan sonra, kendini kapadı.



Örnek Readiness Probe Yaml içeriği;

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    team: development
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: blue
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /healthcheck
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 20
          periodSeconds: 3
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Readiness ve Liveness Probe 'lar aynı anda kullanılabilir.
