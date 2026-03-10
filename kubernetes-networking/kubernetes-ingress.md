---
icon: hand-point-down
---

# Kubernetes Ingress

Kubernetes'te uygulamalarınızı dış dünyaya açmayı öğrendiğinizde aklınıza muhtemelen ilk olarak `LoadBalancer` veya `NodePort` servis tipleri gelir. Ancak sisteminiz büyüdükçe her bir uygulama için ayrı bir LoadBalancer oluşturmak büyük bir mimari krize dönüşür.

Diyelim ki sisteminizde 10 farklı web (mikroservis) uygulamanız var. Eğer standart yöntemi kullanırsanız, bu 10 uygulamanın her biri için bulut sağlayıcınızdan (AWS, Google Cloud vb.) ayrı birer dış IP (LoadBalancer) satın almanız gerekir. Bu durum hem yönetimi imkansızlaştırır, hem faturalarınızı inanılmaz derecede şişirir, hem de bu 10 IP'nin her birine ayrı ayrı SSL sertifikası kurmanızı gerektirir.

İşte tam bu noktada, Kubernetes'in trafik yönlendirme şefi olan Ingress sahneye çıkar.

### Ingress Tam Olarak Nedir?

Ingress, LoadBalancer'ı tamamen hayatımızdan çıkarmaz; aksine onu akıllıca kullanmamızı sağlar. 10 farklı uygulama için 10 ayrı LoadBalancer oluşturmak yerine, sisteme bir Ingress Controller (motor) kurduğunuzda, bulut sağlayıcınız sistemin en önüne otomatik olarak sadece 1 adet LoadBalancer tahsis eder.

Dışarıdan gelen tüm trafik sizin hiçbir şey yapmanıza gerek kalmadan bu tek kapıdan içeri girer ve hemen arkasında çalışan "Ingress" adındaki akıllı santrale (yönlendiriciye) uğrar. Ingress, gelen isteğin üzerindeki adresi (domain veya URL) okur ve "Sen muhasebeye (A servisine) gideceksin, sen ise insan kaynaklarına (B servisine) gideceksin" diyerek trafiği içerideki doğru servislere dağıtır.

#### Service vs. Ingress Karşılaştırması

| **Özellik**       | **Service (LoadBalancer)**                                             | **Ingress**                                                                       |
| ----------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Kullanım Maliyeti | Her dışarı açılan uygulama için 1 adet yeni IP satın alır (Pahalıdır). | Arkada kaç uygulama olursa olsun, her şeyi tek IP arkasında toplar (Ekonomiktir). |
| Bağlantı Türü     | TCP/UDP seviyesinde çalışır. İçeriği okuyamaz (Kördür).                | HTTP ve HTTPS trafiğini okur, anlar ve karar verir (Akıllıdır).                   |
| Yönlendirme       | Yoktur. Tek IP, sadece tek bir servise gider.                          | Tek IP üzerinden domain (host) ve URL yoluna (path) göre yönlendirme yapar.       |
| SSL/TLS Desteği   | Kendi başına şifre çözme yeteneği yoktur.                              | SSL sertifikalarını doğrudan kendi üzerinde çözer (TLS Termination).              |

***

### Önemli Bir Detay: Ingress Controller

`Ingress` objesinin kendisi aslında sadece bir kural dosyasından (YAML) ibarettir. Siz kuralları yazarsınız ama içeride bu kuralları uygulayacak fiziki bir trafik polisi (yönlendirici yazılım) yoksa hiçbir şey çalışmaz.

Bu yüzden Ingress kurallarınızın çalışabilmesi için Cluster'ınıza bir Ingress Controller kurmak zorundasınız. Bunlar, Nginx, Traefik veya HAProxy, Ingress Controller'ların ta kendileridir. O tek LoadBalancer'ın hemen arkasında durup, yazdığınız YAML kurallarını anında okuyan ve trafiği fiilen dağıtan motorlar bunlardır.

Sisteme NGINX Ingress Controller gibi bir motor kurduğunuzda, bu kurulum paketi aslında kendi içinde `type: LoadBalancer` olan bir servis barındırır.

***

### Örnek Bir Ingress YAML Dosyası İncelemesi

İşte yukarıda bahsettiğimiz "Yola Dayalı (Path-based)" yönlendirmenin YAML dünyasındaki karşılığı:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ornek-ingress
spec:
  rules:
  - host: "bolatogluyapi.com"
    http:
      paths:
      # KURAL 1: EĞER URL "/api" İLE BİTİYORSA
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-api-service
            port:
              number: 80
      
      # KURAL 2: EĞER URL "/blog" İLE BİTİYORSA
      - path: /blog
        pathType: Prefix
        backend:
          service:
            name: blog-frontend-service
            port:
              number: 80
```

### Özetle Ingress Bize Ne Kazandırır?

1. Maliyet Tasarrufu: Onlarca uygulama için onlarca dış IP satın almaktan kurtarır. Her şeyi tek IP arkasında birleştirir.
2. Merkezi SSL Yönetimi: Her uygulamanın içine ayrı ayrı SSL sertifikası kurmakla uğraşmazsınız. Sertifikayı Ingress'e yüklersiniz, şifreleme ana kapıda çözülür.
3. Temiz Mimari: Trafik yönlendirme kurallarını kodunuzun içinden çıkarıp, merkezi bir altyapı bileşenine bırakmış olursunuz.

