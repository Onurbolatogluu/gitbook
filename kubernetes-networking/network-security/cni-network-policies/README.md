---
icon: bench-tree
---

# CNI Network Policies

Varsayılan (default) bir Kubernetes kurulumunda, `cluster` içindeki tüm `pod`'lar birbiriyle hiçbir kısıtlama olmadan özgürce konuşabilir. Bu durumu, yüzlerce insanın bulunduğu ve herkesin herkesle fısıldaşabildiği devasa bir salona benzetebiliriz. Ancak iş `production` ortamına geldiğinde, uygulamanızın güvenliği için bu iletişimi sıkı kurallara bağlamanız gerekir.

İşte tam bu noktada Network Policies devreye girer. Ancak standart Kubernetes yetenekleri bazen yetersiz kalabilir. Bu yazıda, standart politikaların ötesine geçen CNI tabanlı gelişmiş ağ politikalarını, özellikle de modern altyapıların favorisi olan Cilium üzerinden A'dan Z'ye inceleyeceğiz.

***

### 1. Temel Kavramlar: Ingress, Egress ve Whitelist Modeli

Network politikalarını anlamak için trafik yönlerini ve güvenlik duruşunu netleştirmemiz gerekir:

* Ingress (Geliş): Bir `pod`'a dışarıdan gelen trafiktir.
* Egress (Çıkış): Bir `pod`'dan dışarıya başlatılan trafiktir.

#### 🛡️ Default Deny ve Whitelist

Güvenlikte en iyi pratik "Default Deny" duruşudur. Bir network politikası uyguladığınız an, Kubernetes o `pod` için "Whitelist" modeline geçer. Yani: Açıkça izin verilmeyen her türlü trafik anında bloklanır (drop).

```
       [ İzin Verilmeyen Pod ] ──(❌ DROP)─┐
                                           │
  [ Frontend Pod ] ──(✅ İZİN VERİLDİ)──▶ [ Hedef Pod (Backend) ] ──(✅ İZİN VERİLDİ)──▶ [ Database Pod ]
                                           ▲ (Ingress Kontrolü)          (Egress Kontrolü) ▼
                                           │                                               │
       [ Dış Dünya (İnternet) ] ──(❌ DROP)┘                                   (❌ DROP) [ Başka Sunucu ]
```

***

### 2. Standart Kubernetes Politikaları vs. CNI Uzantıları

Kubernetes kendi içinde temel bir ağ politikası desteği sunar (sadece IP ve Port bazlı). Ancak Cilium veya Calico gibi gelişmiş bir CNI eklentisi kullandığınızda, sisteminize adeta "süper güçler" eklemiş olursunuz.

| **Özellik**                   | **Standart (Built-In) Policy** | **CNI Eklentileri (Örn: Cilium)**                   |
| ----------------------------- | ------------------------------ | --------------------------------------------------- |
| Layer 7 Filtreleme            | ✖️ Yok                         | ✅ HTTP, gRPC, Kafka bazlı derin kurallar            |
| Şifreleme & İzolasyon         | ✖️ Yok                         | ✅ mTLS ve IPsec desteği                             |
| Gelişmiş Kısıtlamalar         | ✖️ Yok                         | ✅ IP Whitelist/Blacklist, Rate Limiting             |
| Gözlemlenebilirlik            | Kısıtlı                        | ✅ Gerçek zamanlı metrikler ve anlık trafik izleme   |
| Çoklu Cluster (Multi-Cluster) | ✖️ Yok                         | ✅ Birden fazla `cluster`'ı kapsayan global kurallar |

#### Neden Cilium?

Cilium, Linux çekirdeğindeki eBPF teknolojisini kullanır. Bu sayede paketleri `pod`'a gelmeden çok daha önce, çekirdek (kernel) seviyesinde inanılmaz bir hızla analiz eder ve filtreler. Performans kaybı yaratmadan Layer 3'ten Layer 7'ye kadar derinlemesine güvenlik sağlar.

***

### 3. Katman Katman Ağ Politikaları (Layer 3, Layer 4, Layer 7)

Cilium ile ağ politikaları (CiliumNetworkPolicy) yazarken kurallarınızı OSI modelinin farklı katmanlarına göre belirleyebilirsiniz. Her politikanın kalbinde, kuralın hangi `pod`'lara uygulanacağını belirten `endpointSelector` bulunur.

#### 🌐 Layer 3 (Ağ Katmanı) Politikaları

Bu katman, derin paket incelemesi (Deep Packet Inspection) yapmadan, sadece IP adreslerine, `label`'lara (etiketlere) veya DNS isimlerine bakarak trafiğe yön verir.

Örnek 1: Sadece belirli `pod`'lardan gelen Ingress trafiğine izin vermek:

```yml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: layer3-ingress-only
spec:
  endpointSelector:
    matchLabels:
      role: backend
  ingress:
    - fromEndpoints:
        - matchLabels:
            role: frontend
```

_(Bu kural: Sadece `role: frontend` etiketli pod'ların, `role: backend` etiketli pod'lara erişmesine izin verir.)_

Örnek 2: Belirli IP bloklarına (CIDR) Egress çıkışına izin vermek:

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: layer3-egress-cidr
spec:
  endpointSelector:
    matchLabels:
      role: backend
  egress:
    - toCIDR:
        - "20.1.1.1/32"
    - toCIDRSet:
        - cidr: "10.0.0.0/8"
          except:
            - "10.96.0.0/12"
```

Bu kural, `backend` pod'larının dışarıya (egress) sadece `20.1.1.1` IP'sine ve `10.0.0.0/8` ana ağına veri göndermesine izin verir.

Ancak `except` komutuyla bir istisna koyarak, izin verilen bu geniş ağın içindeki `10.96.0.0/12` bloğuna erişmelerini kesinlikle engeller.

***

🚪 Layer 4 (Taşıma Katmanı) Politikaları

Bu katman TCP ve UDP portlarını kontrol eder. Cilium varsayılan olarak `ping` (ICMP) isteklerini bloklar.

Örnek 3: Dışarıya sadece TCP 80 portundan çıkışa izin vermek:

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: layer4-example
spec:
  endpointSelector:
    matchLabels:
      app: myService
  egress:
    - toPorts:
        - ports:
            - port: "80"
              protocol: TCP
```

***

#### 🧠 Layer 7 (Uygulama Katmanı) Politikaları

Standart Kubernetes politikalarının yapamadığı sihir buradadır. Sadece IP ve Porta bakmaz; paketin içine girerek HTTP metodunu (GET, POST), URL yollarını (path) veya HTTP başlıklarını (headers) okuyup karar verir.

Örnek 4: HTTP Path ve Header bazlı ince ayar güvenlik:

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: layer7-example
spec:
  endpointSelector:
    matchLabels:
      app: myService
  ingress:
    - toPorts:
        - ports:
            - port: "80"
              protocol: TCP
      rules:
        http:
          - method: GET
            path: "/path1$"
          - method: PUT
            path: "/path2$"
            headers:
              - "X-My-Header: true"
```

_(Bu mükemmel kural şunları söyler: `myService` pod'una 80 portundan gelen bir istek eğer `GET /path1` ise kabul et. Eğer `PUT /path2` ise, anında kabul etme, önce HTTP header'ına bak; `X-My-Header: true` varsa içeri al!)_

***

### 4. Hepsini Birleştirelim: Kapsamlı (L3-L7) Politika Örneği

Sisteminizdeki bir uygulamanın hem kimden trafik alacağını (Ingress) hem de kime trafik gönderebileceğini (Egress) tek bir devasa kuralda birleştirebiliriz:

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "example-policy"
spec:
  # Kural kime uygulanacak?
  endpointSelector:
    matchLabels:
      app: myapp
      
  # GELEN TRAFİK KURALLARI (Ingress)
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: frontend
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
          rules:
            http:
              - method: "GET"
                path: "/public"
                
  # GİDEN TRAFİK KURALLARI (Egress)
  egress:
    - toEndpoints:
        - matchLabels:
            app: database
      toPorts:
        - ports:
            - port: "3306"
              protocol: TCP
```

Bu konfigürasyon, `app=myapp` etiketli `pod`'u zırh gibi sarar:

* İçeriye sadece: `frontend` uygulamasından, 80 portuna ve sadece `GET /public` yoluna gelen istekleri alır.
* Dışarıya sadece: `database` uygulamasının 3306 (MySQL) portuna veri gönderebilir. Başka hiçbir yere gidemez.

***

### 5. Kapsam (Scope): Namespace vs. Cluster-Wide

Son olarak, yazdığınız bu politikaların `cluster` içinde nereye etki edeceğini belirlemeniz gerekir. Cilium bu noktada iki farklı obje türü sunar:

| **Obje Türü (Kind)**             | **Etki Alanı (Scope)** | **Ne Zaman Kullanılır?**                                                                                                                   |
| -------------------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `CiliumNetworkPolicy`            | Tek bir `Namespace`    | Sadece belirli bir projenin veya uygulamanın izolasyonunu sağlarken.                                                                       |
| `CiliumClusterwideNetworkPolicy` | Tüm `Namespace`'ler    | Tüm şirketi kapsayan global güvenlik kuralları yazarken (Örn: "Tüm cluster genelinde dışarıya DNS sorguları hariç tüm çıkışları engelle"). |

