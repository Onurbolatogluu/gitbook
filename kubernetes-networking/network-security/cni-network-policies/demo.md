---
icon: video
---

# Demo

Bir önceki konumuzda CNI ve eBPF tabanlı ağ politikalarının teorisini konuşmuştuk. Şimdi bu teoriyi pratiğe dökme zamanı! Bu laboratuvar çalışmasında, sıradan bir uygulamayı adım adım nasıl zırhlı bir kaleye dönüştüreceğimizi göreceğiz.

En temel Kubernetes `NetworkPolicy` kuralından başlayıp, sadece belirli bir HTTP başlığına (Header) sahip isteklere izin veren çok gelişmiş bir Cilium Layer 7 kuralına kadar uzanan bir güvenlik tünelinden geçeceğiz.

***

### 🏗️ Demo Ortamı Topolojisi

İçeride `app=demo` etiketiyle (label) çalışan bir web uygulamamız (`pod`) var. Bu uygulama dışarıya iki farklı `Service` üzerinden iki port açıyor: 80 ve 5000.

Hedefimiz: Bu uygulamaya sadece `app=admin` etiketine sahip `pod`'ların, yalnızca 80 portundan ve sadece belirli HTTP Path ve Header'ları ile erişebilmesini sağlamak. Diğer tüm `pod`'ları (`client` vb.) engelleyeceğiz.

```
 [ Admin Pod ] (app=admin) ──(Zırhı Geçecek)──▶ 
                                                [ Demo Pod ] (app=demo)
 [ Client Pod ] (Etiketsiz) ──(Bloklanacak)───▶  (Port: 80 ve 5000)
```

Hazırsak, katmanları örmeye başlayalım.

***

### Aşama 1: Varsayılan Kubernetes NetworkPolicy (Layer 3)

Önce işe standart Kubernetes yetenekleriyle başlayalım. `demo` isimli uygulamaya sadece `admin` etiketli `pod`'lardan gelen `ingress` trafiğine izin veren kuralımız şu şekildedir:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: demo-netpol
spec:
  podSelector:
    matchLabels:
      app: demo
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: admin
```

🧪 Test Edelim:

1. İzin Verilen (Admin): Etiketi doğru olan `pod`, her iki porta da erişebilir.

```bash
# 80 portuna istek (Başarılı)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl --connect-timeout 2 app-svc-80

# 5000 portuna istek (Başarılı)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl --connect-timeout 2 app-svc-5000
```

2. Reddedilen (Client): `admin` etiketi olmayan `pod` engellenir.

```bash
# Etiketsiz pod'dan istek (Başarısız - Timeout)
kubectl run --rm -i --tty client --image=curlimages/curl \
  --restart=Never -- \
  curl --connect-timeout 2 app-svc-80
```

> 💡 Önemli: Cilium'un yeteneklerine geçmeden önce, çakışma olmaması için bu standart kuralı siliyoruz:
>
> `kubectl delete networkpolicy demo-netpol`

***

### Aşama 2: Cilium Layer 3 Policy (Aynı Kural, Yeni Güç)

Şimdi aynı izolasyonu Cilium'un CRD'si (`CiliumNetworkPolicy`) ile yazalım. Mantık tamamen aynı, sadece yapı değişiyor.

`cilium-l3.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: demo-cilium-l3
spec:
  endpointSelector:
    matchLabels:
      app: demo
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: admin
```

Bunu `apply` edip test ettiğinizde, Aşama 1 ile birebir aynı sonucu (sadece `admin` erişebilir) alırsınız. Ancak artık altyapımız eBPF destekli!

***

### Aşama 3: Cilium Layer 4 Policy (Port Kısıtlaması)

`admin` pod'unun her iki porta da erişmesini istemiyoruz. Sadece TCP 80 portunu açıp, 5000 portunu kapatacağız.

`cilium-l4.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: demo-cilium-l4
spec:
  endpointSelector:
    matchLabels:
      app: demo
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: admin
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
```

🧪 Test Edelim:

```bash
# 80 portuna istek (Başarılı - İzin verildi)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl --connect-timeout 2 app-svc-80

# 5000 portuna istek (Başarısız - Timeout yedi)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl --connect-timeout 2 app-svc-5000
```

***

### Aşama 4: Cilium Layer 7 Policy (HTTP Path Kısıtlaması)

Burada standart politikaların yetersiz kaldığı o sihirli noktaya geliyoruz. Port 80 açık evet, ama `admin` her sayfayı görebilsin mi? Hayır! Sadece `/api` ve `/healthz` yollarına (path) istek atabilsin.

`cilium-l7.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: demo-cilium-l7
spec:
  endpointSelector:
    matchLabels:
      app: demo
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: admin
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
          rules:
            http:
              - method: GET
                path: /healthz
              - method: GET
                path: /api
```

🧪 Test Edelim:

```bash
# Ana dizine (/) istek (Başarısız - Cilium L7 engelledi)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl --connect-timeout 2 app-svc-80

# Sadece /api dizinine istek (Başarılı)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl --connect-timeout 2 app-svc-80/api
```

***

### Aşama 5: API Key Header Zorunluluğu (L7 İleri Seviye)

Güvenliği zirveye taşıyoruz. `/api` adresine herkes (admin etiketli olsa bile) elini kolunu sallayarak giremesin. HTTP isteğinin içine gizli bir "Şifre" (`X-API-KEY: ABC123` header'ı) koymasını zorunlu kılalım.

`cilium-l7-header.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: demo-cilium-l7-header
spec:
  endpointSelector:
    matchLabels:
      app: demo
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: admin
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
          rules:
            http:
              - method: GET
                path: /healthz
              - method: GET
                path: /api
                headers:
                  - name: X-API-KEY
                    value: ABC123
```

🧪 Final Testi:

```bash
# Header OLMADAN /api'ye istek atma (Başarısız - Access Denied)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl --connect-timeout 2 app-svc-80/api

# Doğru Header İLE /api'ye istek atma (Başarılı)
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --labels app=admin --restart=Never -- \
  curl -H "X-API-KEY: ABC123" --connect-timeout 2 app-svc-80/api
```

***

### 📊 Kural Evrimi Özeti

Bu yolculukta basit bir `label` eşleşmesinden, paketin içini açıp "Header" kontrolü yapan muazzam bir sisteme geçtik.

| **Aşama**      | **Katman** | **Konfigürasyon**       | **Açıklama**                                                  |
| -------------- | ---------- | ----------------------- | ------------------------------------------------------------- |
| 1. K8s Default | L3         | `demo-netpol`           | `app=admin` etiketli pod'lara her portu açar.                 |
| 2. Cilium L3   | L3         | `cilium-l3.yaml`        | Aynı kuralın Cilium CRD'si ile yazılmış hali.                 |
| 3. Cilium L4   | L4         | `cilium-l4.yaml`        | Sadece TCP 80 portuna girişi sınırlar (5000'i kapatır).       |
| 4. Cilium L7   | L7         | `cilium-l7.yaml`        | Paket içine bakar, sadece `/api` ve `/healthz`'ye izin verir. |
| 5. Cilium L7+H | L7+H       | `cilium-l7-header.yaml` | Ekstra güvenlik: `/api` için `X-API-KEY` header'ı arar.       |
