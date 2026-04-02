---
icon: meteor
---

# Demo

Bu rehberde, sıfırdan bir `cluster` üzerinde `cert-manager` kurulumu yapacak, Traefik arkasında çalışan basit bir uygulamayı önce güvenli bir test (Staging) ortamıyla şifreleyecek, ardından kusursuz bir şekilde gerçek (Production) sertifikaya geçiş yapacağız.

### 🏗️ Topoloji ve İş Akışı

Sürecimiz şu hiyerarşik adımları takip edecek:

```
 1. Kurulum: Helm ile cert-manager'ı cluster'a entegre et.
 2. Analiz: Mevcut şifresiz (HTTP) uygulamayı incele.
 3. Test: Staging Issuer kur -> Ingress'e bağla -> Başarısını doğrula.
 4. Canlıya Alma: Production Issuer kur -> Ingress'i güncelle (Kesintisiz geçiş).
```

***

### 1. cert-manager Kurulumu

Kuruluma Kubernetes dünyasının paket yöneticisi olan Helm ile başlıyoruz. Öncelikle Jetstack reposunu ekleyip, izole bir `namespace` yaratarak CRD'lerle birlikte kurulumu tamamlıyoruz.

```bash
# Jetstack Helm reposunu ekleyip güncelleyelim
helm repo add jetstack https://charts.jetstack.io
helm repo update

# cert-manager için özel bir namespace oluşturalım
kubectl create namespace cert-manager

# CRD'leri (Custom Resource Definitions) aktif ederek kurulumu başlatalım
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set installCRDs=true
```

Kurulumun başarılı olduğunu doğrulamak için `pod`'ların `Running` durumuna gelmesini beklemeliyiz:

```bash
kubectl get pods -n cert-manager
```

***

### 2. Mevcut Yapıyı İnceleme

Elimizde `default` namespace'inde çalışan, dış dünyaya Traefik Ingress ile açılmış "whoami" adında basit bir web uygulaması (`deployment` ve `service`) var. Mevcut durumu görelim:

```bash
# Pod, Service ve Deployment kaynaklarını listeleyelim
kubectl get all -n default

# Ingress kurallarını görelim
kubectl get ingress whoami-ingress -n default

# Ingress'in iç detaylarını inceleyelim
kubectl describe ingress whoami-ingress -n default
```

_Bu aşamada Ingress'in sadece 80 portunu (HTTP) dinlediğini ve şifresiz olduğunu göreceğiz._

***

### 3. Let's Encrypt Staging (Test) Issuer Oluşturma

Altın Kural: Let's Encrypt'in `production` sunucularında çok katı bir hız sınırı (rate limit) vardır. Eğer konfigürasyonunuzda bir hata varsa ve sürekli başarısız istek atarsanız, Let's Encrypt sizi banlar. Bu yüzden her zaman önce Staging ortamında test yapmalıyız.

Aşağıdaki YAML dosyasını `staging-issuer.yaml` olarak kaydedelim:

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: admin@bolatogluyapi.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
      - http01:
          ingress:
            name: whoami-ingress
```

Uygulayalım ve sisteme yansıdığını doğrulayalım:

```bash
kubectl apply -f staging-issuer.yaml

# Issuer'ın başarıyla Let's Encrypt'e kaydolduğunu kontrol edelim
kubectl describe issuer letsencrypt-staging -n default

# Kimlik bilgilerinin Secret olarak oluştuğunu görelim
kubectl get secrets -n default
```

***

### 4. Ingress'i TLS İçin Güncelleme

Şimdi Traefik'e, trafiği şifrelemesi gerektiğini ve bunu `staging` ortamından yapacağını söyleyeceğiz. `whoami-ingress.yaml` dosyasını aşağıdaki gibi düzenleyip `annotations` ve `tls` bloklarını ekliyoruz:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami-ingress
  namespace: default
  annotations:
    # cert-manager'a talimat veriyoruz:
    cert-manager.io/issuer: letsencrypt-staging
spec:
  ingressClassName: traefik
  tls:
    - hosts:
        # DNS kaydınızın Traefik LoadBalancer IP'sine baktığından emin olun!
        - app.bolatogluyapi.com
      secretName: web-ssl
  rules:
    - host: app.bolatogluyapi.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: whoami
                port:
                  name: web
```

Değişiklikleri `deploy` edelim:

```bash
kubectl apply -f whoami-ingress.yaml
```

***

### 5. Doğrulamayı (ACME Challenge) ve Sertifikayı İzleme

`apply` komutunu çalıştırdığımız anda `cert-manager` arka planda geçici bir `pod` yaratıp HTTP-01 doğrulamasını başlatır. Bu büyüyü canlı izlemek için Ingress olaylarına (events) bakalım:

```bash
kubectl describe ingress whoami-ingress -n default
```

Çıktının en altındaki `Events` kısmında şunları görmeliyiz:

1. Geçici bir `cm-acme-http-solver-...` yolu eklendiği,
2.  Sertifikanın başarıyla yaratıldığı:

    `Normal CreateCertificate 10s cert-manager-ingress-shim Successfully created Certificate "web-ssl"`

_Test sertifikamız tarayıcıda "Güvenli Değil" uyarısı verir, bu normaldir. Amacımız otomasyonun çalışıp çalışmadığını kanıtlamaktı._

***

### 6. Production (Canlı) Ortama Geçiş

Otomasyon kusursuz çalışıyor! Artık gerçek ve geçerli bir sertifika alabiliriz. Bunun için `prod-issuer.yaml` adında yeni bir dosya oluşturuyoruz:

```bash
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-production
spec:
  acme:
    # URL artık production sunucusuna bakıyor!
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@bolatogluyapi.com
    privateKeySecretRef:
      name: letsencrypt-production
    solvers:
      - http01:
          ingress:
            name: whoami-ingress
```

Sisteme uygulayalım:

```bash
kubectl apply -f prod-issuer.yaml
kubectl describe issuer letsencrypt-production -n default
```

***

### 7. Ingress'i Production'a Çevirmek

Artık YAML dosyasını baştan düzenlememize gerek yok. Tek bir `kubectl annotate` komutuyla, `Ingress` objesinin üzerindeki `staging` etiketini ezip yerine `production` etiketini yazabiliriz. Bu işlem kesinti yaratmaz.

```bash
kubectl annotate ingress whoami-ingress \
  cert-manager.io/issuer=letsencrypt-production \
  --overwrite -n default
```

Sonucu görmek için Ingress detaylarına tekrar bakalım:

```bash
kubectl describe ingress whoami-ingress -n default
```

Event loglarında şu muhteşem satırı göreceksiniz:

`Normal RenewCertificate 12s cert-manager-ingress-shim Successfully renewed Certificate "web-ssl"`

Tebrikler! Traefik Ingress'iniz artık Let's Encrypt `production` sertifikasıyla tam koruma altında ve 90 günde bir otomatik olarak yenilenecek.

***

#### 📌 Özet Tablo: Staging vs Production

| **Özellik**             | **letsencrypt-staging**               | **letsencrypt-production**                           |
| ----------------------- | ------------------------------------- | ---------------------------------------------------- |
| Kullanım Amacı          | Otomasyonu test etmek, hata ayıklamak | Canlı ortamlarda kullanıcılara güvenli hizmet sunmak |
| Tarayıcı Güveni         | Güvenilmez (Uyarı verir)              | Güvenilir (Yeşil Kilit)                              |
| ACME URL                | `...acme-staging-v02...`              | `...acme-v02...`                                     |
| Hız Sınırı (Rate Limit) | Çok esnek (Testler için ideal)        | Çok katı (Hatalı isteklerde ban yeme riski)          |
