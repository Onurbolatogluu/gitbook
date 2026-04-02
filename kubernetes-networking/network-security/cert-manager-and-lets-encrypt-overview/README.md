---
icon: square-half-stroke
---

# Cert Manager and Lets Encrypt Overview

Bir Kubernetes `cluster` ortamında uygulamalarınızı dış dünyaya güvenli (HTTPS) bir şekilde açmak kritik bir gereksinimdir. Ancak SSL/TLS sertifikalarını manuel olarak talep etmek, `Secret` objeleri olarak sisteme yüklemek ve süresi dolmadan yenilemek, `production` ortamlarında sürdürülebilir bir yöntem değildir.

Bu rehberde, manuel süreçleri tamamen ortadan kaldıran ve sertifika yaşam döngüsünü baştan sona otomatize eden iki harika aracın (cert-manager ve Let's Encrypt) birlikte nasıl çalıştığını, mimarisini ve kurulumunu sıfırdan, adım adım inceleyeceğiz.

***

### 1. cert-manager Nedir ve Nasıl Çalışır?

cert-manager, Kubernetes için geliştirilmiş açık kaynaklı bir eklentidir (add-on). Temel görevi, TLS sertifikalarının alınmasını, yenilenmesini ve yönetilmesini otomatize etmektir. Sadece Let's Encrypt ile değil; HashiCorp Vault veya self-signed sertifika otoriteleriyle de entegre çalışabilir.

Öne Çıkan Özellikleri:

* Sertifika taleplerini ve yenilemelerini otomatik yapar.
* Standart tekil alan adlarını ve Wildcard (örn: `*.example.com`) sertifikaları destekler.
* Kubernetes mimarisine tam uyum sağlayarak kendi CRD'lerini (`Issuer`, `ClusterIssuer`, `Certificate`) sisteme ekler.
* Alınan sertifikaları ve private keyleri doğrudan Kubernetes `Secret` objeleri olarak güvenle depolar.

#### Mimari ve CRD (Custom Resource Definition) Yapısı

cert-manager, arka planda çalışan ve Kubernetes API'sini sürekli izleyen `controller`'lardan oluşur. Siz bir sertifika talep ettiğinizde, bu `controller`'lar istenen durum (desired state) ile mevcut durumu eşitler.

Aşağıdaki tablo, cert-manager'ın temel yapıtaşlarını gösterir:

<table data-header-hidden><thead><tr><th width="159.5377197265625"></th><th width="165.89520263671875"></th><th></th></tr></thead><tbody><tr><td><strong>CRD Objesi</strong></td><td><strong>Kapsam (Scope)</strong></td><td><strong>Ne İşe Yarar?</strong></td></tr><tr><td>Issuer</td><td>Namespaced</td><td>Sadece bulunduğu <code>namespace</code> içindeki sertifika taleplerini karşılayabilen yetki tanımlamasıdır.</td></tr><tr><td>ClusterIssuer</td><td>Cluster</td><td>Tüm <code>cluster</code> genelinde (herhangi bir <code>namespace</code>'ten) gelen taleplere yanıt verebilen, global yetki tanımlamasıdır.</td></tr><tr><td>Certificate</td><td>Namespaced</td><td>Alınmasını istediğiniz sertifikanın detaylarını (alan adı, hangi <code>Secret</code> içinde saklanacağı ve hangi Issuer'ın kullanılacağı) belirten manifestodur.</td></tr></tbody></table>

### 2. cert-manager Kurulumu

Sistemin bu yeni objeleri (CRD'leri) tanıyabilmesi için kurulumu eksiksiz yapmalıyız. Kurulum için en yaygın ve güvenli yöntem Helm kullanmaktır.

Yöntem 1: Helm ile Kurulum (Önerilen)

```bash
# Jetstack reposunu ekleyip güncelliyoruz
helm repo add jetstack https://charts.jetstack.io
helm repo update

# cert-manager'ı kendi namespace'ini yaratarak ve CRD'leri aktif ederek kuruyoruz
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  --set installCRDs=true
```

Yöntem 2: kubectl ile Kurulum (Alternatif)

Eğer Helm kullanmıyorsanız, doğrudan YAML dosyasını apply edebilirsiniz:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml
```

#### 🛠️ Hayat Kurtaran CLI Aracı: cmctl

cert-manager'ı yönetmek ve hataları ayıklamak (troubleshooting) için resmi `cmctl` aracını kullanabilirsiniz. Bu araç, uzun `kubectl` komutları yerine süreci çok basitleştirir:

```bash
# Kurulumun ve API'nin düzgün çalışıp çalışmadığını doğrular:
cmctl check api

# Sorunlu bir Issuer veya Certificate objesinin iç detaylarını inceler:
cmctl inspect issuer <issuer-adi>

# Süresinin dolmasını beklemeden bir sertifika yenileme işlemini manuel tetikler:
cmctl renew <certificate-adi>
```

***

### 3. Let's Encrypt ve ACME İş Akışı

Let's Encrypt, herkesin kullanımına açık, ücretsiz ve otomatik bir Sertifika Otoritesi'dir (CA). Sistemlerle konuşmak için ACME (Automated Certificate Management Environment) protokolünü kullanır.

* Sertifikaların geçerlilik süresi 90 gündür. Bu kısa süre, otomasyonu (cert-manager kullanımını) zorunlu kılar.
* İki farklı ortam (endpoint) sunar: Staging (Test) ve Production (Canlı).

| **Ortam**  | **ACME Endpoint URL**                                    | **Kullanım Senaryosu**                                                                                         |
| ---------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Staging    | `https://acme-staging-v02.api.letsencrypt.org/directory` | Testler içindir. Sertifika verir ama tarayıcılar güvenmez. Çok sık talep yapabilirsiniz (rate limit esnektir). |
| Production | `https://acme-v02.api.letsencrypt.org/directory`         | Gerçek, güvenilir sertifikalar verir. Kurulumları Staging'de test ettikten sonra buraya geçilmelidir.          |

#### 🗺️ ACME Doğrulama Topolojisi

cert-manager ve Let's Encrypt arasındaki diyalog (HTTP-01 Challenge) şu şekilde gerçekleşir:

```
 1. TALEP: 
 [ cert-manager ] -----> "example.com için sertifika istiyorum" -----> [ Let's Encrypt ]

 2. Challenge: 
 [ Let's Encrypt ] ----> "O zaman example.com'un sana ait olduğunu kanıtla. 
                          Şu token'ı web sitende yayınla." -----> [ cert-manager ]

 3. Fulfill: 
 [ cert-manager ] -----> (Geçici bir pod ve Ingress kuralı oluşturarak 
                          istenen token'ı yayınlar)

 4. DOĞRULAMA (Validate): 
 [ Let's Encrypt ] ----> (İnternet üzerinden http://example.com/.well-known/acme-challenge/ adresine 
                          gider ve token'ı okur. Doğruysa onaylar.)

 5. TESLİMAT: 
 [ Let's Encrypt ] ----> Al bakalım sertifikan! -----> [ cert-manager ] (Secret olarak kaydeder)
```

***

### 4. Pratik Entegrasyon ve Konfigürasyon Senaryosu

Şimdi öğrendiklerimizi birleştirelim. Önce bir Issuer tanımlayacağız, ardından bir Ingress objesinde bu Issuer'ı çağıracağız.

#### Adım 1: Staging Issuer Oluşturmak

Aşağıdaki YAML, `namespace` içerisinde Let's Encrypt'in test sunucusuna bağlanacak bir kimlik tanımlar.

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # Staging sunucusunu kullanıyoruz (Production'a geçerken burası güncellenmeli)
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-staging-key
    solvers:
    # Let's Encrypt'e kimliğimizi kanıtlamak için HTTP-01 yöntemini ve nginx ingress sınıfını kullanacağız.
    - http01:
        ingress:
          class: nginx
```

#### Adım 2: Kubernetes Ingress'i Yapılandırmak

Uygulamamızı dışarı açarken, Ingress manifestomuza ufak bir `annotation` (not) ekleyerek süreci başlatıyoruz.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    # cert-manager'a "Sertifikamı letsencrypt-staging isimli Issuer üzerinden al" diyoruz.
    cert-manager.io/issuer: letsencrypt-staging
spec:
  tls:
    - hosts:
        - example.com
      # Alınan sertifikanın depolanacağı Secret objesinin adı
      secretName: web-tls
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

Siz bu Ingress dosyasını `deploy` ettiğiniz anda; cert-manager `annotations` kısmını fark eder, Let's Encrypt ile iletişime geçer, doğrulama (challenge) işlemlerini arka planda halleder ve başarılı olursa sertifikayı `web-tls` isimli `Secret` içine yerleştirir. Artık uygulamanız güvenli bir şekilde dış dünyaya hizmet vermeye hazırdır.

