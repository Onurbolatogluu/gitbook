---
icon: square-half-stroke
---

# Demo

## Uygulamalı Rehber: Kubernetes ExternalDNS Kurulumu ve Otomasyon Testi

Sistemimize yeni bir uygulama kurduğumuzda, gidip DNS panellerinde (Cloudflare, AWS Route 53, GoDaddy vb.) manuel olarak IP adresi girmek, modern DevOps kültüründe kabul edilemez bir zaman kaybıdır.

Bu laboratuvar çalışmasında, Kubernetes Cluster'ımıza ExternalDNS kuracak, onu API anahtarlarımızla DNS sağlayıcımıza bağlayacak ve yazdığımız tek bir satır kodla DNS kayıtlarının otomatik oluşmasını izleyeceğiz.

### Senaryomuz

* Alan Adımız: `bolatogluyapi.com`
* Amacımız: İçeride çalışan bir API servisi için yazdığımız Ingress dosyasını sisteme uyguladığımız an, Cloudflare (veya GoDaddy) panelimizde `api.bolatogluyapi.com` kaydının otomatik olarak oluşmasını sağlamak.

***

### 1. Adım: Helm ile ExternalDNS Kurulumu

Öncelikle ExternalDNS'in resmi Helm deposunu sistemimize ekliyoruz:

```bash
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm repo update
```

Kurulum için bir `values.yaml` dosyası oluşturmalıyız. Bu dosya, ExternalDNS'in hangi alan adını yöneteceğini ve o panele girebilmek için hangi şifreyi (API Token) kullanacağını barındırır.

Kendi bilgisayarınızda `external-dns-values.yaml` adında bir dosya oluşturun ve içini sağlayıcınıza göre doldurun:

```yaml
# external-dns-values.yaml

provider: cloudflare # GoDaddy kullanıyorsanız buraya 'godaddy' yazılır.

cloudflare:
  apiToken: "SİZİN_GİZLİ_API_ANAHTARINIZ" # DNS sağlayıcınızdan aldığınız yetki anahtarı

# Sahiplik (TXT Registry) Ayarları - Önceki yazıda bahsettiğimiz güvenlik kilidi!
txtOwnerId: "benim-k8s-clusterim"
txtPrefix: "ext-dns-"

# ExternalDNS sadece bu alan adında işlem yapabilsin, diğerlerini bozmasın diye filtre koyuyoruz:
domainFilters:
  - bolatogluyapi.com
```

_Not: Güvenlik pratiği olarak, gerçek üretim (production) ortamlarında API anahtarlarınızı doğrudan YAML içine yazmak yerine Kubernetes `Secret` objeleri veya Vault sistemleri kullanmalısınız._

Dosyamız hazır olduğuna göre motoru çalıştırıyoruz:

```bash
helm install external-dns external-dns/external-dns --values ./external-dns-values.yaml --namespace kube-system
```

Kurulumun başarılı olduğunu loglardan teyit edelim:

```bash
kubectl logs -f deployment/external-dns -n kube-system
```

_Eğer loglarda "Connected to provider" veya "All records are already up to date" gibi ifadeler görüyorsanız, tebrikler! Kubernetes artık DNS panelinizle canlı olarak konuşuyor._

***

### 2. Adım: Ingress Oluşturmak

Şimdi sistemde `api-service` adında bir uygulamanızın çalıştığını varsayalım. Bu uygulamayı dışarı açmak için standart bir Ingress YAML dosyası hazırlıyoruz. Ancak içine ExternalDNS'i tetikleyecek o sihirli notu (annotation) ekliyoruz.

`api-ingress.yaml` dosyasını oluşturun:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bolatoglu-api-ingress
  annotations:
    # İŞTE SİHİR BURADA: ExternalDNS bu notu okuyacak!
    external-dns.alpha.kubernetes.io/hostname: "api.bolatogluyapi.com"
spec:
  ingressClassName: traefik
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
```

#### Bulut vs. On-Premise

Eğer bu testi AWS veya GKE gibi gerçek bir bulut ortamında yapıyorsanız, yukarıdaki YAML tek başına yeterlidir. Bulut sağlayıcı size bir LoadBalancer IP'si verir, ExternalDNS de o IP'yi okuyup doğrudan DNS'e yazar.

Ancak lokal bir bilgisayarda (Minikube vb.) veya kendi fiziksel sunucunuzda test yapıyorsanız sistem size otomatik bir dış IP veremez. Bu durumda hedefin hangi IP olacağını ExternalDNS'e manuel olarak söylemeniz gerekir. Bunun için `annotations` kısmına şu satırı da eklemelisiniz:

```yaml
    external-dns.alpha.kubernetes.io/target: "192.168.1.50" # Sizin fiziksel sunucu IP'niz
```

***

### 3. Adım: Sonuçları Canlı İzlemek

Hazırladığınız Ingress dosyasını sisteme uygulayın:

```bash
kubectl apply -f api-ingress.yaml
```

Şimdi nefesleri tutup hemen ExternalDNS'in log ekranına geri dönün:

```bash
kubectl logs -f deployment/external-dns -n kube-system
```

Loglarda saniyeler içinde şuna benzer mükemmel bir satır düşecektir:

`msg="Desired change: CREATE api.bolatogluyapi.com A record"`

Şimdi DNS sağlayıcınızın (Cloudflare/GoDaddy) web paneline giriş yapın. Hiçbir şeye dokunmadığınız halde `api.bolatogluyapi.com` adresinin otomatik olarak açıldığını ve yanına `txtOwnerId` imzalı koruma kaydının (TXT) atıldığını kendi gözlerinizle göreceksiniz.

Eğer az önce oluşturduğunuz Ingress'i silecek olursanız (`kubectl delete -f api-ingress.yaml`), ExternalDNS gidip DNS panelinizdeki o kaydı anında temizleyecektir.
