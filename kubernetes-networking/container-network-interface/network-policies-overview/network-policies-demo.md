---
icon: video
---

# Network Policies Demo

Ortamımız: Cluster'ımızda `frontend`, `backend` ve `database` adında 3 farklı Pod çalışıyor.

Amacımız: Sistemi tamamen dış dünyaya ve izinsiz iç erişimlere kapatıp, SADECE `frontend`'in `backend`'e (80 portundan) ulaşabilmesini sağlamak.

**Aşama 1: Mevcut Durumu (Tehlikeyi) Görmek**

Kubernetes'te varsayılan olarak tüm Pod'lar birbiriyle ve dış dünyayla özgürce konuşabilir. Önce bu durumu test edelim.

```bash
# 1. Frontend'den Backend'e ping at (Başarılı olur)
kubectl exec -it frontend -- curl -m 3 http://backend

# 2. Frontend'den internete ping at (Başarılı olur)
kubectl exec -it frontend -- ping -c 1 google.com
```

_Sonuç:_ Şu an sistemimiz tamamen korumasız, herkes her yere gidebiliyor.

**Aşama 2: Tüm Kapıları Kilitlemek (Default Deny)**

Önce her şeyi yasaklayan iki temel kural yazıyoruz: Hem gelen (Ingress) hem giden (Egress) trafiği tamamen kesiyoruz.

1\. Default Deny Ingress (Gelişleri Yasakla) YAML:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}    # Süslü parantez boş, yani "Tüm Pod'ları seç"
  policyTypes:
  - Ingress
```

2\. Default Deny Egress (Çıkışları Yasakla) YAML:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}    # "Tüm Pod'ları seç"
  policyTypes:
  - Egress
```

Bu iki dosyayı (`kubectl apply -f ...` ile) sisteme yükledikten sonra Aşama 1'deki testleri tekrar yaparsan, tüm ping isteklerinin "Timeout" (Zaman Aşımı) yediğini göreceksin. Sistem artık %100 kapalı bir kutu.

**Aşama 3: Sadece İhtiyaç Olan İletişime İzin Vermek (Specific Allow)**

Şimdi kapalı kutumuzda sadece tek bir delik açacağız: `frontend`, `backend`'in 80 (HTTP) portuna gidebilsin. Bunun için iki tarafın da güvenlik görevlisine emir vermeliyiz.

1\. Backend'in Kapısındaki Görevliye Emir (Allow Ingress):

_"Eğer sana gelen kişi 'frontend' ise ve 80 portuna girmek istiyorsa, onu içeri al."_

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-ingress
spec:
  podSelector:
    matchLabels:
      app: backend          # Kuralın Sahibi: Backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend     # Gelen Kişi: Frontend
    ports:
    - protocol: TCP
      port: 80              # İzin Verilen Kapı: 80
```

2\. Frontend'in Kapısındaki Görevliye Emir (Allow Egress):

_"Eğer dışarı çıkıp 'backend' isimli hedefin 80 portuna gideceksen, çıkışına izin veriyorum."_

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-allow-egress
spec:
  podSelector:
    matchLabels:
      app: frontend         # Kuralın Sahibi: Frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend      # Gidilecek Hedef: Backend
    ports:
    - protocol: TCP
      port: 80              # Çıkılacak Kapı: 80
```

**Aşama 4: Sonuçları Doğrulama**

Kuralları yükledikten sonra testlerimizi yapıyoruz:

```bash
# 1. Frontend'den Backend'in 80 portuna web isteği at (BAŞARILI OLUR)
kubectl exec -it frontend -- curl -m 3 http://backend

# 2. Frontend'den dış dünyaya (Google'a) ping at (BAŞARISIZ OLUR)
kubectl exec -it frontend -- ping -c 1 google.com
```

Önemli Teknik Not: Bu aşamadan sonra `frontend`'den `backend`'e `ping` atmaya çalışırsan başarısız olur. Neden mi? Çünkü biz kurallarda sadece `TCP Port 80`'e izin verdik. Ping komutu ise portları kullanmaz, _ICMP_ adında farklı bir protokol kullanır ve biz ICMP'ye izin vermedik. Bu, Network Policy'nin ne kadar nokta atışı çalıştığının en güzel kanıtıdır!
