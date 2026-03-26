---
icon: wave-sine
---

# Advanced Networking

Diyelim ki bir e-ticaret uygulaması yapıyorsun. Bu uygulama tek bir monolith değil, **microservice**'lerden oluşuyor:

* `user-service` → Kullanıcı bilgileri
* `order-service` → Sipariş işlemleri
* `payment-service` → Ödeme işlemleri
* `notification-service` → Bildirimler

Bu service'ler birbirleriyle sürekli konuşuyor. İşte sorun burada başlıyor: **Bu service'ler arası iletişimi kim yönetecek? Güvenlik nasıl sağlanacak? Bir service çökerse ne olacak?**

İşte **Service Mesh** ve **Multi-Cluster** tam bu sorunlara çözüm getiriyor.

Önce genel topology'yi göstereyim:

Şimdi sana bu konuyu adım adım, gerçek hayat örnekleriyle anlatacağım.

### 1. Problemi Anlamak: Microservice'ler Neden Baş Ağrıtıyor?

Önce şunu hayal et: Bir e-ticaret uygulaması var. Eskiden tek bir büyük uygulama (monolith) idi. Şimdi onu parçalara ayırdın — her parça bir microservice. Ama bu parçalar birbirleriyle konuşmak zorunda.

Sorun şu: Her microservice'in içine "şu service'e bağlanırken şifrele", "eğer o service çökerse şuraya git", "trafiğin %10'unu yeni versiyona yönlendir" gibi logic'ler yazman gerekiyor. Bu, **her bir service'in kodunu networking logic ile kirletiyor**.

İşte Service Mesh diyor ki: _"Bu networking işlerini bana bırak, sen sadece uygulama kodunu yaz."_

Bunu bir görselle açıklayayım:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Yukarıdaki diyagramda farkı görebilirsin. Üstte her service kendi içinde retry, mTLS, logging gibi şeyleri taşımak zorunda. Altta ise bu işler **sidecar proxy**'ye devredilmiş — service'ler sadece kendi işlerini yapıyor.

### 2. Service Mesh Nedir?

Service Mesh, microservice'ler arasındaki iletişimi yöneten **ayrı bir altyapı katmanı**. Bunu şöyle düşün:

Bir şirkette çalışıyorsun. Her departman (service) birbirine mesaj göndermek istiyor. İki yol var:

* **Mesh olmadan**: Herkes kendi kuryesini tutar, kendi şifrelemesini yapar, kendi log'unu tutar. Kaos.
* **Mesh ile**: Şirketin profesyonel bir posta departmanı var. Sen mektubu yaz, posta departmanı taşıma, şifreleme, loglama, yönlendirme — her şeyi halleder.

#### Service Mesh'in 3 Ana Bileşeni

**a) Control Plane** — Beyindir. Tüm konfigürasyonu, policy'leri ve service discovery'yi yönetir. Örneğin Istio'da `istiod` bu rolü üstlenir.

**b) Data Plane** — Sidecar proxy'lerin toplamıdır. Her pod'un yanına bir proxy konur ve tüm gelen/giden trafik bu proxy üzerinden geçer.

**c) Sidecar Proxy** — Her pod'un içine yerleştirilen küçük bir proxy (genellikle **Envoy**). Uygulama kodu hiç değişmez — proxy tüm network trafiğini intercept eder.

Tamam, şimdi üçünü birlikte, net başlıklarla ve diyagramlar bölümleri boğmayacak şekilde gönderiyorum.

***

## Service Mesh Ne Sağlıyor?

Önce şunu bil: Istio'yu kurduğunda Kubernetes'e **yeni obje tipleri** eklenir. Bildiğin `Pod`, `Service`, `Deployment` gibi objeler Kubernetes'in kendi built-in objeleri. Istio ise **CRD** (Custom Resource Definition) denen mekanizma ile Kubernetes'e kendi objelerini tanıtır. Yani Kubernetes'e diyor ki: _"Bak, artık `VirtualService` diye bir obje tipi de var, bunu da tanı."_

Bu yüzden bu YAML'lar yabancı geliyor — çünkü bunlar Kubernetes'in değil, **Istio'nun** objeleri. Ama yazım şekli aynı: `apiVersion`, `kind`, `metadata`, `spec` — hepsi aynı yapı.

***

### a) Traffic Management

Diyelim ki `payment-service`'in v1'i production'da çalışıyor, yeni v2'yi de deploy ettin. Ama v2'ye birden tüm trafiği vermek istemiyorsun — önce %5 ile test etmek istiyorsun. Buna **Canary Deployment** deniyor.

Bunun için Istio'da **iki obje** lazım:

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

#### Obje 1: DestinationRule — "Hedefleri tanımla"

Normal Kubernetes `Service` objesi, label selector ile pod'ları bulur. Ama `Service` objesi "v1 pod'ları şunlar, v2 pod'ları şunlar" diye ayırmaz — hepsini tek bir havuza atar. **DestinationRule** ise o havuzu **subset**'lere (alt gruplara) böler:

```yaml
apiVersion: networking.istio.io/v1beta1    # Istio'nun API grubu
kind: DestinationRule                       # Obje tipi: "Hedef kuralı"
metadata:
  name: payment-service
spec:
  host: payment-service          # Hangi Kubernetes Service'e uygulanacak?
  subsets:                       # Alt grupları tanımla
    - name: v1                   # Bu alt grubun adı: "v1"
      labels:
        version: v1              # version=v1 label'ına sahip pod'lar
    - name: v2                   # Bu alt grubun adı: "v2"
      labels:
        version: v2              # version=v2 label'ına sahip pod'lar
```

Bunu şöyle düşün:

```
Kubernetes Service "payment-service":
  ├── Tüm pod'lar (v1 + v2 karışık)

DestinationRule uygulandıktan sonra:
  ├── subset "v1" → sadece version=v1 label'lı pod'lar
  └── subset "v2" → sadece version=v2 label'lı pod'lar
```

Tek başına DestinationRule trafik yönlendirmez — sadece **grupları tanımlar**.

#### Obje 2: VirtualService — "Trafiği yönlendir"

Bu obje, gelen trafiğin **hangi subset'e, ne oranda** gideceğini belirler:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService                        # "Sanal servis" — trafik yönlendirme kuralı
metadata:
  name: payment-service
spec:
  hosts:
    - payment-service            # Hangi host'a gelen trafiği yönetiyoruz?
  http:                          # HTTP trafiği için kurallar
    - route:
        - destination:
            host: payment-service
            subset: v1           # DestinationRule'da tanımladığımız "v1" grubu
          weight: 95             # Trafiğin %95'i buraya
        - destination:
            host: payment-service
            subset: v2           # DestinationRule'da tanımladığımız "v2" grubu
          weight: 5              # Trafiğin %5'i buraya
```

İki obje birlikte çalışıyor:

* **DestinationRule** = "Havuzdaki pod'ları grupla" (v1, v2 diye ayır)
* **VirtualService** = "Trafiği bu gruplara şu oranda dağıt" (%95 v1, %5 v2)

#### A/B Testing Örneği

Canary'de oran bazlı dağıtım vardı (%95-%5). A/B Testing'de ise **koşula göre** yönlendirme yaparsın:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: payment-service
spec:
  hosts:
    - payment-service
  http:
    # Kural 1: Header eşleşirse → v2'ye git
    - match:
        - headers:
            user-group:
              exact: beta-testers
      route:
        - destination:
            host: payment-service
            subset: v2

    # Kural 2: Hiçbir koşul eşleşmezse → v1'e git (default)
    - route:
        - destination:
            host: payment-service
            subset: v1
```

Istio yukarıdan aşağıya kuralları kontrol eder. İlk eşleşen kural uygulanır. Hiçbiri eşleşmezse en sondaki "koşulsuz" route çalışır — bir nevi `else` bloğu.

***

### b) Security — mTLS

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

**TLS** = Tek taraflı kimlik doğrulama. Tarayıcında `https://` kullandığında olan şey bu — sen sunucunun kim olduğunu doğrularsın, ama sunucu senin kim olduğunu doğrulamaz.

**mTLS** (mutual TLS) = Çift taraflı kimlik doğrulama. Hem client hem server birbirinin certificate'ını kontrol eder.

```
Normal TLS:
  Browser ──────→ Server
  "Sen gerçekten google.com musun?"  ✓
  "Browser sen kimsin?"              ✗ (sorulmaz)

mTLS:
  Order-service ──────→ Payment-service
  "Sen gerçekten payment-service mısın?"   ✓
  "Sen gerçekten order-service misin?"      ✓  (karşılıklı!)
```

Service Mesh bunu **otomatik** yapar. Her pod'un yanındaki sidecar proxy:

* Otomatik certificate alır (control plane'den)
* Her outgoing request'i encrypt eder
* Her incoming request'in certificate'ını doğrular
* Certificate'ları periyodik olarak rotate eder (yeniler)

**Uygulama kodunda hiçbir şey değişmez.** App hâlâ `http://payment-service:80` diye plain HTTP call yapar. Proxy bunu yakalayıp mTLS'e çevirir.

#### PeerAuthentication Objesi

```yaml
apiVersion: security.istio.io/v1beta1     # Istio'nun security API grubu
kind: PeerAuthentication                   # "Eş kimlik doğrulama" — Istio CRD'si
metadata:
  name: default
  namespace: production                    # Bu namespace'deki tüm pod'lara uygulanır
spec:
  mtls:
    mode: STRICT
    # STRICT     = Sadece mTLS kabul et. Plain-text trafik reddedilir.
    # PERMISSIVE = mTLS'i de plain-text'i de kabul et. (Geçiş dönemi için)
    # DISABLE    = mTLS kapalı.
```

`PERMISSIVE` mode önemli — mesh'e geçiş yaparken kullanırsın. Tüm service'lere aynı anda sidecar ekleyemezsin. Geçiş döneminde bazıları proxy'li, bazıları proxy'siz olur. `PERMISSIVE` her ikisini de kabul eder. Migration tamamlanınca `STRICT`'e geçersin.

#### mTLS Akışı (Adım Adım)

```
1. Order-service  →  (plain HTTP)      →  Kendi Envoy proxy'si
2. Envoy proxy    →  (mTLS encrypted)  →  Karşı taraf Envoy proxy
3. Karşı Envoy    →  (plain HTTP)      →  Payment-service
```

Uygulama hiçbir zaman TLS ile uğraşmıyor. Proxy her şeyi hallediyor.

***

### c) Observability

Bu kısım da proxy sayesinde çalışıyor. Her request proxy'den geçtiği için, proxy şu bilgileri otomatik toplar:

#### Metrics

Prometheus formatında expose edilir. Kurulum yapmadan şunları görürsün:

```
istio_requests_total{source="order-service", destination="payment-service", response_code="200"}
istio_request_duration_milliseconds{source="order-service", destination="payment-service"}
```

"order-service'den payment-service'e kaç request gitti, kaçı 200 döndü, latency ne kadar?" — hepsi hazır.

#### Distributed Tracing

Bir kullanıcı "Sipariş Ver"e tıkladığında arka planda şu chain çalışıyor diyelim:

```
User → API Gateway → order-service → payment-service → notification-service
```

Service mesh ile proxy'ler bunu otomatik izler. Jaeger veya Zipkin dashboard'unda:

```
Trace ID: abc-123
├─ API Gateway          (2ms)
├─ order-service        (15ms)
│  ├─ DB query          (5ms)
│  └─ payment-service   (45ms)
│     └─ Stripe API     (40ms)
└─ notification-service (8ms)

Total: 70ms
```

Böylece "nerede yavaşlık var?" sorusunun cevabını anında bulursun. Bu örnekte Stripe API call'u 40ms ile darboğaz.

#### Logging

Her request'in access log'u otomatik tutulur:

```
[2025-03-26T10:15:32] "GET /api/orders" 200
  source=user-service  destination=order-service
  duration=15ms  bytes=2048
```

***

### Özet Tablo

| Fayda                     | Ne yapar?                             | Istio objeleri                       | Kod değişir mi? |
| ------------------------- | ------------------------------------- | ------------------------------------ | --------------- |
| **a) Traffic Management** | Trafiği oran veya koşula göre böler   | `VirtualService` + `DestinationRule` | Hayır           |
| **b) Security (mTLS)**    | Tüm trafiği şifreler, kimlik doğrular | `PeerAuthentication`                 | Hayır           |
| **c) Observability**      | Metric, trace, log toplar             | Otomatik (obje gerekmez)             | Hayır           |

Üçünün ortak noktası: **Uygulama koduna dokunmuyorsun.** Her şeyi sidecar proxy hallediyor.

### 4. Popüler Service Mesh Çözümleri

| Provider    | Özelliği                                    | Ne zaman seçersin?                       |
| ----------- | ------------------------------------------- | ---------------------------------------- |
| **Istio**   | En zengin feature set, Envoy proxy kullanır | Büyük ve karmaşık microservice ortamları |
| **Linkerd** | Çok hafif, kolay kurulum, düşük overhead    | Basitlik ve performance istiyorsan       |
| **Cilium**  | eBPF tabanlı, sidecar'sız çalışabilir       | Zaten Cilium CNI kullanıyorsan           |

Istio ve Linkerd **sidecar proxy** modeli kullanır (her pod'a bir proxy konur). Cilium ise **eBPF** kullanarak kernel seviyesinde çalışır — sidecar'a gerek kalmaz, bu da daha az resource consumption demektir.

***

### 5. Multi-Cluster Nedir?

Şimdi ikinci büyük konuya geçelim. Diyelim ki tek bir Kubernetes cluster'ın var ve her şey güzel çalışıyor. Ama ya:

* O cluster'ın olduğu data center çökerse?
* Avrupa'daki kullanıcılara düşük latency vermek istersen?
* Bazı workload'ların yasal olarak belirli bir ülkede kalması gerekiyorsa (GDPR gibi)?

İşte **Multi-Cluster** bunları çözer: Birden fazla Kubernetes cluster'ı birlikte çalıştırmak.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### Multi-Cluster Use Case'leri

**1. High Availability & Disaster Recovery** Frankfurt'taki data center'da yangın çıktı → Trafik otomatik olarak US veya Asia cluster'a yönlenir. Kullanıcılar hiçbir şey farketmez.

**2. Low Latency** Türkiye'deki bir kullanıcı EU cluster'a bağlanır (yakın). Japonya'daki kullanıcı Asia cluster'a bağlanır. Herkes kendine en yakın cluster'dan hizmet alır.

**3. Compliance (GDPR)** Avrupa vatandaşlarının verileri Avrupa'da kalmalı → EU cluster sadece AB verileriyle çalışır. US cluster Amerika verilerini işler.

**4. Scalability** Black Friday geldi, trafik 10x arttı → Yeni bir cluster daha ayağa kaldırıp workload'ı dağıtabilirsin.

#### Multi-Cluster Yönetim Araçları

* **Kubernetes Federation**: Birden fazla cluster'ı tek bir API'den yönetmeni sağlar.
* **Google Anthos**: Google'ın multi-cluster management platformu. On-prem + cloud cluster'larını birlikte yönetir.
* **Rancher**: SUSE'nin multi-cluster yönetim aracı.

### 6. Service Mesh + Multi-Cluster = Süper Güç

İşte en güçlü kombinasyon burası. Service mesh'i tek cluster'da kullanmak güzel, ama onu multi-cluster ile birleştirince bambaşka bir seviyeye çıkıyor:

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Bu kombinasyonun sağladığı şeyleri somut bir örnekle açıklayayım:

Diyelim EU cluster'daki `order-service`, US cluster'daki `payment-service`'i çağırmak istiyor. Normalde bu çok zor — farklı cluster'lar, farklı network'ler. Ama service mesh + multi-cluster ile:

1. **Cross-cluster service discovery**: EU'daki order-service, `payment-service` diye çağırdığında mesh otomatik olarak US cluster'daki service'i bulur.
2. **mTLS**: Cluster'lar arası trafik bile encrypted. Ortadaki hiçbir network cihazı içeriği okuyamaz.
3. **Failover**: US cluster çökerse, mesh trafiği EU'daki backup payment-service'e yönlendirir.
4. **Single pane of glass**: Tüm cluster'lardan gelen metric ve trace'leri tek bir dashboard'da görürsün.

### 7. Cilium Cluster Mesh — eBPF'in Gücü

#### eBPF Nedir?

Normalde network trafiği Linux kernel'ında işlenir. Sidecar proxy modeli (Istio/Linkerd) ise her paketi userspace'e çıkarır, proxy'den geçirir, tekrar kernel'a sokar. Bu ekstra hop latency ekler.

**eBPF** ise direkt **kernel içinde** küçük programlar çalıştırmana izin verir. Yani trafik kernel'dan hiç çıkmadan işlenir.

```
Traditional sidecar model:
  App → [userspace] → Proxy → [kernel] → Network → [kernel] → Proxy → [userspace] → App
  
Cilium eBPF model:
  App → [kernel: eBPF programı] → Network → [kernel: eBPF programı] → App
```

#### Cilium Cluster Mesh Ne Sağlıyor?

* **Cross-cluster service discovery**: Cluster A'daki pod, Cluster B'deki service'i sanki aynı cluster'daymış gibi çağırabilir
* **Cross-cluster load balancing**: Trafik en sağlıklı ve en yakın pod'a yönlendirilir. hangi cluster'da olursa olsun
* **Unified network policy**: Tek bir policy tanımlarsın, tüm cluster'larda geçerli olur
* **Encryption**: WireGuard veya IPsec ile trafik şifrelenir

#### Cilium Cluster Mesh Nasıl Kurulur? (Basit Senaryo)

Öncelikle iki cluster'da da Cilium kurulu olmalı ve cluster'lar arası network connectivity olmalı (VPN, VPC peering gibi).

```bash
# Cilium CLI ile Cluster Mesh'i aktif et
cilium clustermesh enable --context cluster1
cilium clustermesh enable --context cluster2

# İki cluster'ı birbirine bağla
cilium clustermesh connect --context cluster1 --destination-context cluster2

# Durumu kontrol et
cilium clustermesh status --context cluster1
```

Bağlantı kurulduktan sonra, bir service'i cross-cluster'a açmak için sadece bir annotation ekliyorsun:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-service
  annotations:
    service.cilium.io/global: "true"   # Bu service her iki cluster'dan da erişilebilir
spec:
  selector:
    app: payment
  ports:
    - port: 80
```

Bu kadar! Artık her iki cluster'daki pod'lar `payment-service`'e erişebilir ve Cilium otomatik olarak en yakın/sağlıklı endpoint'e yönlendirir.

***

### 8. Özet: Ne Zaman Ne Kullanırsın?

| Senaryo                                               | Çözüm                                                       |
| ----------------------------------------------------- | ----------------------------------------------------------- |
| Microservice'ler arası güvenli iletişim istiyorum     | **Service Mesh** (Istio, Linkerd, veya Cilium)              |
| Canary deployment / A/B testing yapmak istiyorum      | **Service Mesh** — traffic splitting                        |
| Service'ler arası tüm trafiği encrypt etmek istiyorum | **Service Mesh** — mTLS                                     |
| Uygulamam birden fazla region'da çalışmalı            | **Multi-Cluster**                                           |
| Bir region çökerse hizmet devam etmeli                | **Multi-Cluster** + failover                                |
| GDPR uyumluluğu lazım                                 | **Multi-Cluster** — region-isolated cluster'lar             |
| Tüm bunları bir arada istiyorum                       | **Service Mesh + Multi-Cluster** (örn: Cilium Cluster Mesh) |
| Zaten Cilium CNI kullanıyorum                         | **Cilium Cluster Mesh** — eBPF ile sidecar'sız mesh         |
