---
icon: shield
---

# Network Security

Kubernetes üzerinde yüzlerce hatta binlerce `container`'ı birden fazla `node` üzerinde koşturduğunuz bir senaryoda, sistem güvenliği artık bir seçenek değil, kritik bir zorunluluktur. Sistemdeki tek bir zafiyet, bir anda tüm `workload`'larınıza sıçrayabilir, hassas verilerinizi sızdırabilir veya hizmetlerinizi tamamen durdurabilir.

Bir Kubernetes `cluster`'ını devasa ve çok hareketli bir plaza gibi düşünebilirsiniz. İçeri kimin gireceği, içerideki odalar (yani `pod`'lar) arasında kimin dolaşabileceği ve konuşmaların dinlenip dinlenmediği sıkı kurallara bağlı olmalıdır.

Bu yazıda, Kubernetes güvenlik duruşunuzu (security posture) güçlendirecek 5 temel stratejiyi, sıfırdan ve en pratik haliyle inceleyeceğiz.

### 1. Şifreleme ve SSL Otomasyonu (cert-manager & Let's Encrypt)

Dış dünyadan `cluster`'ınıza gelen trafiğin (HTTPS) şifrelenmiş olması şarttır. Ancak sertifikaları manuel olarak oluşturmak, takip etmek ve süresi dolmadan yenilemek bir kabustur. İşte burada işi otomatize etmemiz gerekiyor.

* Let's Encrypt: Bize ücretsiz ve güvenilir SSL/TLS sertifikaları sağlayan servistir (ACME protokolü üzerinden çalışır).
* cert-manager: Kubernetes içinde çalışan ve Let's Encrypt ile konuşarak sertifika alma/yenileme işlerini (CRD'ler kullanarak) sizin yerinize yapan bir "robot"tur.

Aşağıdaki örnekte, `cert-manager`'a "Benim için Let's Encrypt üzerinden sertifika al" diyen bir `ClusterIssuer` ve `Certificate` konfigürasyonu görüyoruz:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        ingress:
          class: traefik
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-com-tls
spec:
  secretName: example-com-tls
  dnsNames:
  - example.com
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

Bu yapı sayesinde sertifikanız Kubernetes `Secret` objesi olarak saklanır ve süresi dolmaya yakınken `cert-manager` tarafından otomatik olarak yenilenir. Gece rahat uyursunuz!

> 💡 İpucu: `cert-manager`'ı `deploy` etmeden önce HTTP-01 veya DNS-01 doğrulamaları (challenge) için DNS kayıtlarınızın doğru ayarlandığından emin olun.

***

### 2. Ingress Güvenliği

`Ingress`, `cluster`'ınızın ana kapısıdır. Dışarıdan gelen HTTP/HTTPS isteklerini alır ve içerideki doğru servise yönlendirir. Bu ana kapının güvenli ve şifreli olması gerekir.

Popüler bir `Ingress Controller` olan Traefik, `cert-manager` ve Let's Encrypt ile kusursuz bir uyum içinde çalışır. Aşağıdaki konfigürasyon (IngressRoute), gelen trafiği nasıl güvenli bir şekilde karşılayıp içerideki uygulamaya ileteceğini gösterir:

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute # Traefik'in gelen trafiği yönlendirme kuralı.
metadata:
  name: secure-web
  annotations:
    # Otomasyon: Bu kural için otomatik olarak geçerli bir SSL sertifikası üret.
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  entryPoints:
    - websecure # Sadece şifrelenmiş (HTTPS) kapıdan gelen istekleri kabul et.
  routes:
    - match: Host(`app.example.com`) # Eğer kullanıcı tam olarak bu adrese geliyorsa...
      kind: Rule
      services:
        - name: web # ...isteği cluster içindeki "web" isimli Service'e...
          port: 80  # ...80 portundan teslim et.
  tls:
    # HTTPS şifrelemesi için kullanılacak sertifikanın tutulduğu Secret objesi.
    secretName: example-com-tls
```

Burada Traefik, arka planda sertifika taleplerini `cert-manager`'a devreder. Manuel hiçbir müdahaleye gerek kalmadan HTTPS aktif hale gelir.

***

### 3. CNI Network Policies

Ana kapıyı (Ingress) kilitledik diyelim, peki ya içerisi? Kubernetes'te varsayılan olarak tüm `pod`'lar birbiriyle konuşabilir. Bu çok tehlikelidir! Eğer bir `frontend` pod'u zafiyet barındırıyorsa, saldırgan rahatlıkla `database` pod'una sızabilir (buna yatay hareket / lateral movement denir).

Bunu engellemek için `NetworkPolicy` kullanırız.

#### 🗺️ Pod İletişimi Topolojisi

Aşağıdaki senaryoyu hayal edelim: Sadece Frontend'in Web'e erişmesini istiyoruz. Diğer her şeyi engelleyeceğiz.

```
       [ İnternet ]
            │
            ▼ 
      [ Frontend Pod ]
            │
            │  ✅ İZİN VERİLDİ (Network Policy)
            ▼
         [ Web Pod ]
```

Örnek Standart NetworkPolicy YAML: (Sadece 'frontend' etiketine sahip `pod`'ların, 'web' etiketine sahip `pod`'lara 80 portundan erişmesine izin verir)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
spec:
  podSelector:
    matchLabels:
      role: web
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 80
```

#### Cilium ile Gelişmiş Politikalar

Güvenliği işletim sistemi çekirdeği (kernel) seviyesine indirmek isterseniz, eBPF teknolojisini kullanan Cilium harika bir CNI alternatifidir. Cilium ile çok daha zengin politikalar yazabilirsiniz:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-frontend-to-web
spec:
  endpointSelector:
    matchLabels:
      role: web
  ingress:
    - fromEndpoints:
        - matchLabels:
            role: frontend
      toPorts:
        - ports:
            - port: "80"
              protocol: TCP
```

***

### 4. Mutual TLS (mTLS) - Karşılıklı Güven

Standart TLS/SSL'de sadece istemci (client) sunucunun (server) kimliğini doğrular.

mTLS (Mutual TLS) ise iki yönlüdür. `Cluster` içinde A servisi B servisiyle konuşmak istediğinde, her iki taraf da veriyi takas etmeden önce birbirinin kimliğini doğrular. Bu iki yönlü kimlik doğrulama, araya girme (man-in-the-middle) saldırılarını tamamen ortadan kaldırır.

> ⚠️ Dikkat: Süresi dolan veya yanlış yapılandırılan sertifikalar mTLS bağlantılarını anında bozar. Sertifika yaşam döngülerini mutlaka izleyin ve yenilemeleri otomatikleştirin.

***

### 5. Hubble ile Gözlemlenebilirlik

Güvenlikte altın bir kural vardır: Göremediğiniz bir şeyi koruyamazsınız. Ağınızda hangi `pod` hangisiyle konuşuyor? Sistemde olağandışı bir durum var mı?

Eğer Cilium kullanıyorsanız, onunla entegre çalışan Hubble size ağ akışları (network flows), uygulama performansı ve güvenlik olayları hakkında çok derin bir görünürlük sağlar.

Pratik CLI Kullanımı:

Öncelikle `cluster`'ınıza Cilium ve Hubble'ı kurmanız (`deploy` etmeniz) gerekir. Aşağıdaki komutlarla kurulumu yapıp canlı trafiği izlemeye hemen başlayabilirsiniz:

```bash
# 1. Cilium'u Hubble ile birlikte cluster'a hızlıca deploy edelim
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.12/install/kubernetes/quick-install.yaml

# 2. Hubble metrics sunucusunu aktifleştirelim
cilium hubble enable

# 3. Sadece 'default' namespace içindeki canlı ağ akışını (network flows) izleyelim
hubble observe --namespace default
```

Bu sayede trafiğinizi saniye saniye takip edebilir, hataları (troubleshooting) çok daha hızlı çözebilirsiniz.

