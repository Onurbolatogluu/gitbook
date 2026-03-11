---
icon: layer-minus
---

# Demo

Traefik'in Kubernetes ortamındaki trafiği nasıl mükemmel bir şekilde yönettiğini önceki yazılarımızda konuşmuştuk. Peki bu motoru sistemimize nasıl entegre edeceğiz?

Kubernetes'te Traefik kurmanın iki yolu vardır:

1. Manuel (Zor) Yol: RBAC yetkilerini (ClusterRole, ServiceAccount), Deployment tanımlarını ve Service (LoadBalancer) YAML dosyalarını tek tek elinizle yazmak.
2. Helm (Akıllı) Yol: Tüm bu karmaşık dosyaları tek bir paket halinde, sektör standardı olan Kubernetes paket yöneticisi (Helm) ile saniyeler içinde kurmak.

Bu laboratuvarda,  production ortamlarının vazgeçilmezi olan Helm yöntemini kullanacağız.

### 1. Adım: Helm ile Traefik Motorunu Kurmak

Öncelikle Traefik'in resmi Helm deposunu sistemimize ekliyor ve güncelliyoruz:

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
```

Kurulumun temiz olması için Traefik'e özel bir isim alanı (namespace) yaratalım:

```bash
kubectl create namespace traefik
```

Ve motoru çalıştırıyoruz! _(Not: Eğer bulut ortamındaysanız, bu komut sizin için otomatik olarak bir LoadBalancer IP'si de yaratacaktır)._

```bash
helm install traefik traefik/traefik --namespace=traefik
```

Her şeyin sağlıklı bir şekilde ayağa kalktığını doğrulamak için kontrol edelim:

```bash
kubectl get all -n traefik
```

_Çıktıda Traefik Pod'unun `Running` durumunda olduğunu ve bir `LoadBalancer` (veya NodePort) servisinin oluştuğunu görmelisiniz._

***

### 2. Adım: Test Uygulamasını (Whoami) Ayağa Kaldırmak

Trafiği yönlendirebilmemiz için içeride çalışan bir uygulamaya ihtiyacımız var. Traefik ekibinin geliştirdiği, kendisine gelen HTTP isteklerinin detaylarını ekrana basan çok hafif bir test uygulaması olan `whoami`'yi kullanacağız.

Aşağıdaki `whoami-app.yaml` dosyasını oluşturun:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
  labels:
    app: whoami
spec:
  replicas: 1
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: traefik/whoami
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
spec:
  type: ClusterIP
  selector:
    app: whoami
  ports:
    - port: 80
      targetPort: 80
```

Dosyayı sisteme uygulayın:

```bash
kubectl apply -f whoami-app.yaml
```

Şu an içeride çalışan, dünyadan tamamen izole edilmiş bir uygulamamız ve kapıda bekleyen bir Traefik motorumuz var. Şimdi bu ikisini birbirine bağlayacağız.

***

### 3. Adım: Ingress Kuralını Yazmak

Traefik'e _"Sana gelen trafiği içerideki whoami servisine yönlendir"_ emrini vereceğimiz standart Ingress YAML dosyamızı hazırlıyoruz.

Aşağıdaki `whoami-ingress.yaml` dosyasını oluşturun. _(Önceki yazılarımızda bahsettiğimiz IngressClass detayına dikkat edin!)_

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami-ingress
spec:
  # SİHİRLİ KELİME: Bu kuralı sadece Traefik motoru işlesin!
  ingressClassName: traefik
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: whoami-service
                port:
                  number: 80
```

Dosyayı sisteme uygulayın:

```bash
kubectl apply -f whoami-ingress.yaml
```

***

### 4. Adım: Test ve Logları İzlemek

İşlem tamam! Şimdi dış kapımızın (LoadBalancer veya NodePort) IP adresini öğrenelim:

```bash
kubectl get svc -n traefik
```

Tarayıcınızı açıp `http://<Sistem-IP-Adresiniz>` adresine (veya belirlediğiniz NodePort'a) gittiğinizde, `whoami` uygulamasının size cevap verdiğini, IP adresinizi ve sunucu detaylarını ekrana bastığını göreceksiniz.

Trafiği Canlı İzlemek İstiyorsanız:

Traefik'in arka planda bu isteği nasıl karşılayıp yönlendirdiğini görmek için terminalden Traefik Pod'unun loglarına canlı olarak bağlanabilirsiniz:

```bash
kubectl logs -f deployment/traefik -n traefik
```

Sayfayı her yenilediğinizde, log ekranına yeni bir HTTP isteğinin (Access Log) düştüğünü görebilirsiniz.

{% hint style="info" %}
On-prem ortamlarda sana o "dış IP'yi" verecek API yoktur. Dolayısıyla Traefik'in o `LoadBalancer` servisi sonsuza dek `<pending>` (bekliyor) durumunda takılı kalır ve sistemine dışarıdan asla erişemezsin.

Bu sorunu çözmek için (eğer MetalLB gibi bir eklentin yoksa), Traefik'in kurulum dosyasındaki (`values.yaml`) o `LoadBalancer` ayarını mecburen `NodePort` olarak değiştirmen gerekir. Böylece sistem sana statik bir IP veremese bile, sunucunun fiziksel IP'si üzerinden örneğin `32080` gibi bir port açarak Traefik motoruna dışarıdan ulaşmanı sağlar.
{% endhint %}

***

***

Yukarıdaki yazımızda Traefik motorunu sistemimize kurmuş ve trafiği nasıl yönlendirdiğimizi konuşmuştuk. Traefik'i anlatırken onun en büyük güçlerinden birinin "Gözlem ve Yönetim Paneli (Dashboard)" olduğundan bahsetmiştik.

Peki körü körüne terminalden komut yazmak yerine, sistemimize giren trafiği, kuralları ve anlık sağlığı görsel bir arayüzden nasıl izleriz? Hatta Traefik'in o meşhur "Hot Reload" özelliğinin gerçekten çalışıp çalışmadığını kendi gözlerimizle nasıl kanıtlarız?

Traefik, sisteminize kurulduğu anda arka planda muazzam bir veri toplar. Hangi porttan kaç kişi giriyor, kurallar (Routers) doğru çalışıyor mu, arkadaki mikroservislerin (Pods) sağlığı ne durumda? Normalde bu veriler güvenlik nedeniyle dış dünyaya tamamen kapalıdır.

Bu laboratuvar çalışmasında, Traefik'in içindeki bu Dashboard'u dışarı açacak ve sistemimizin anlık röntgenini çekeceğiz.

### 1. Adım: Dashboard'u Dışarı Açmak (NodePort ile)

Traefik API'si ve Dashboard'u, varsayılan olarak pod'un içindeki `9000` portunda çalışır. Buna kendi bilgisayarımızdan erişebilmek için bir Service (NodePort) oluşturmamız gerekiyor.

Aşağıdaki `service-dashboard.yaml` dosyasını oluşturun:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: traefik-dashboard-service
  namespace: traefik
spec:
  type: NodePort
  ports:
    - port: 9000
      targetPort: 9000
      nodePort: 30000   # Sabit bir port belirliyoruz (Tarayıcıdan buraya gideceğiz)
  selector:
    app.kubernetes.io/instance: traefik-traefik
```

Dosyayı sisteme uygulayın:

```bash
kubectl apply -f service-dashboard.yaml
```

### 2. Adım: API'yi "Güvensiz" Modda Aktif Etmek

Güvenlik prensipleri gereği Traefik paneli şifresiz erişime kapalıdır. Biz şu an bir test/laboratuvar ortamında olduğumuz için şifreleme ve yetkilendirme (Auth) ayarlarıyla vakit kaybetmeden paneli direkt erişime açacağız. _(Uyarı: Bunu asla Production ortamında yapmayın!)_

Mevcut Traefik Deployment'ını terminal üzerinden canlı olarak düzenliyoruz:

```bash
kubectl edit deployment traefik -n traefik
```

Açılan metin editöründe, `containers` altındaki `args:` (argümanlar) bölümünü bulun ve aşağıdaki iki satırı ekleyin:

```yaml
    args:
      - "--api.dashboard=true"     # Arayüzü aç
      - "--api.insecure=true"      # Şifresiz/Güvensiz erişime izin ver
      # ... diğer argümanlar ...
```

Dosyayı kaydedip çıktığınızda Kubernetes, Traefik pod'unu yeni ayarlarla saniyeler içinde yeniden başlatacaktır. Pod'un çalıştığını teyit edin: `kubectl get pods -n traefik`

### 3. Adım: Kontrol Paneline Giriş ve Keşif

Artık tarayıcınızı açıp doğrudan `http://<Sunucu-IP-Adresiniz>:30000` adresine gidebilirsiniz. Karşınıza Traefik'in modern ve detaylı arayüzü çıkacaktır!

Ekranda önceki yazımızda öğrendiğimiz 4 temel yapı taşını göreceksiniz:

* EntryPoints: Traefik'in dünyayı dinlediği kapılar (web: 80, websecure: 443 vb.).
* Routers: Yazdığınız Ingress kurallarının listesi (Hangi URL nereye gidiyor?).
* Services: Arka planda çalışan uygulamalarınız ve anlık sağlık durumları.
* Middlewares: Trafiğe uyguladığınız hız sınırları veya şifre kuralları.

### 4. Adım: "Hot Reload" Canlı Test Etmek

Geldik en heyecanlı kısma. Traefik'in yeniden başlatılmaya gerek duymadan değişiklikleri anında algıladığını (Hot Reload) özelliğinden bahsetmiştik. Şimdi bunu görsel olarak kanıtlayalım.

Dashboard'da "Services" sekmesini açık tutun ve uygulamanızın (Örn: `whoami`) üzerine tıklayın. Orada şu an sadece 1 adet sağlıklı uç nokta (endpoint/pod) göreceksiniz.

Şimdi terminale dönün ve içerideki uygulamamızın sayısını aniden 5'e çıkarın:

```bash
kubectl scale deployment whoami --replicas=5
```

Hiçbir şeyi yeniden başlatmayın, sayfayı yenilemeyin. Sadece Dashboard'u izleyin. Birkaç saniye içinde Traefik'in yeni açılan 4 Pod'u otomatik olarak keşfettiğini ve listedeki sağlıklı sunucu sayısının canlı olarak 5'e çıktığını göreceksiniz. Trafik anında 5 pod arasında dengelenmeye başlayacaktır.
