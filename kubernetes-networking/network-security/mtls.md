---
icon: square-tugrik
---

# mTLS

Bir önceki konularda `cluster`'ımızın ana kapısını (Ingress) güvenceye aldık ve Network Policies kimin kiminle konuşabileceğini sınırladık. Ancak hala cevaplanması gereken kritik bir soru var: "Konuşmasına izin verdiğimiz bu iki `pod` arasındaki veri akışı gerçekten güvenli mi, yoksa araya biri girip bu trafiği dinleyebilir mi?"

Tıpkı dış dünyadaki trafiği Cloudflare veya HAProxy gibi araçlarla şifrelediğimiz gibi, `cluster` içindeki `backend` servislerinin de birbirleriyle şifreli ve kimlik doğrulamalı konuşması gerekir. İşte mTLS (Mutual TLS) tam olarak bu noktada devreye girer.

***

### 1. Standart TLS vs. Mutual TLS (mTLS) Arasındaki Fark

Konuyu en iyi anlamanın yolu, standart internet trafiğiyle kıyaslamaktır.

* Standart TLS (Tek Yönlü Güven): Bir e-ticaret sitesine girdiğinizde tarayıcınız sunucunun sertifikasını kontrol eder ("Bu site gerçekten o site mi?"). Ancak sunucu _sizin_ kim olduğunuzu sormaz. Sadece sunucu kimliğini kanıtlar.
* mTLS (İki Yönlü Güven): İletişime geçen hem istemci (client) hem de sunucu (server) birbirlerine dijital sertifikalarını gösterir. "Sen kimsin?" ve "Peki sen kimsin?" diyaloğu yaşanır. Her iki taraf da birbirini doğrulamadan (Zero-Trust mantığı) tek bir bayt bile veri transfer edilmez.

#### 🗺️ Zihinde Canlandırma: El Sıkışma (Handshake) Topolojisi

Diyelim ki Bolatoğlu Yapı'nın arka planda çalışan iç sistemlerinde, `Siparis (Order)` servisi `Fatura (Billing)` servisiyle konuşmak istiyor.

```
  [ Sipariş Pod'u (Client) ]                                [ Fatura Pod'u (Server) ]

  1. "Merhaba, ben seninle konuşmak istiyorum" ──────────▶
                                                            2. "Merhaba, al bu benim sertifikam."
                                              ◀──────────
  3. (Sertifikayı doğrular, onaylar)
  4. "Tamam inandım, al bu da BENİM sertifikam" ─────────▶
                                                            5. (Sertifikayı doğrular, onaylar)
                                              ◀──────────
               (✅ İKİ YÖNLÜ ŞİFRELEME VE GÜVENLİ KANAL OLUŞTURULDU)
```

Bu iki yönlü doğrulama sayesinde, eğer bir saldırgan `Sipariş` pod'unu taklit etmeye çalışırsa (Man-in-the-Middle - Ortadaki Adam saldırısı), elinde geçerli bir istemci sertifikası olmadığı için `Fatura` servisi bağlantıyı anında reddeder.

***

### 2. Kubernetes'te mTLS Nasıl Uygulanır? (Service Mesh)

Teoride harika, peki pratikte bunu nasıl yapacağız? Uygulamalarımızın (örneğin yazdığınız bir Node.js veya Python kodunun) içine girip sertifika doğrulama kodları mı yazacağız? Kesinlikle hayır.

Modern Kubernetes mimarilerinde mTLS, uygulamadan tamamen bağımsız bir şekilde Service Mesh araçlarıyla çözülür. En popüler olanları Istio ve Linkerd'dir. (Cilium'un da mTLS desteği vardır ancak şu an beta aşamasındadır).

#### Sidecar Proxy Mantığı

Service Mesh araçları, `pod`'unuzun içine uygulamanızın hemen yanına (sidecar) görünmez bir Proxy (genellikle Envoy proxy) yerleştirir. Uygulamanız sadece dışarıya düz (şifresiz) HTTP isteği attığını sanır. Ancak o istek `pod`'dan çıkmadan önce bu Proxy tarafından yakalanır, mTLS ile şifrelenir, karşı tarafın sertifikası sorulur ve karşı `pod`'daki Proxy'ye teslim edilir.

```
      POD A (Sipariş)                                    POD B (Fatura)
 ┌───────────────────────┐                          ┌───────────────────────┐
 │                       │     mTLS Şifreli Ağ      │                       │
 │  [Uygulama] (HTTP)    │  (Sertifikalar Konuşur)  │    (HTTP) [Uygulama]  │
 │       │               │                          │               ▲       │
 │       ▼               │                          │               │       │
 │  [Sidecar Proxy] ─────┼──────────────────────────┼─────▶ [Sidecar Proxy] │
 │                       │                          │                       │
 └───────────────────────┘                          └───────────────────────┘
```

Bu mimarinin en büyük avantajı; sertifikaların oluşturulması, süresi dolduğunda yenilenmesi (rotation) ve iptal edilmesi işlemlerinin tamamının Service Mesh kontrolcüsü tarafından otomatik yapılmasıdır. Yazılımcının ruhu bile duymaz.

***

### 3. Pratik Senaryo: Istio ile mTLS'i Zorunlu Kılmak

Eğer sisteminizde Istio Service Mesh kuruluysa, mTLS'i belirli bir `namespace` genelinde zorunlu kılmak inanılmaz derecede basittir. Kubernetes'e standart bir YAML göndererek tüm trafiği anında zırhlayabilirsiniz.

Aşağıdaki YAML (`PeerAuthentication` objesi), belirtilen `namespace` içindeki tüm `pod`'lar arası iletişimin kesinlikle (STRICT) mTLS ile yapılmasını emreder. Düz HTTP ile gelen tüm iç istekler anında reddedilir.

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default-strict-mtls
  namespace: bolatoglu-core
spec:
  mtls:
    mode: STRICT
```

Bu kodu `apply` ettiğiniz saniyeden itibaren, o `namespace` içindeki tüm mikroservisler sadece sertifika gösteren diğer servislerle konuşabilir.

***

### 4. Avantajlar ve Bedeller (Trade-offs)

Mühendislikte her güzel şeyin bir bedeli vardır. mTLS'i sisteminize entegre etmeden önce artılarını ve eksilerini tartmak gerekir:

Avantajları (Neden Kullanmalıyız?):

* Ağınıza sızan bir hacker, `pod`'lar arasındaki trafiği dinlese bile sadece şifrelenmiş anlamsız karakterler görür.
* KVKK, GDPR, PCI-DSS (Kredi kartı güvenliği) gibi standartlar genellikle servisler arası trafiğin şifrelenmesini şart koşar.
* Service Mesh sayesinde uygulamaların kaynak koduna dokunmadan tüm sistemi şifreleyebilirsiniz.

Zorlukları (Maliyetleri Nelerdir?):

* Şifreleme ve şifre çözme işlemleri CPU ve RAM tüketir. Ağ trafiğinde (latency) milisaniyelik gecikmeler yaratabilir. Yoğun trafikli `cluster`'larda altyapı (kaynak) planlamasını buna göre yapmak gerekir.
* Sorun anında şifrelenmiş trafiği debug etmek (örneğin `tcpdump` veya `wireshark` ile okumak) çok daha zordur.
* `cluster` dışındaki legacy sistemler bu sertifika yapısını desteklemiyorsa, entegrasyon zorlaşır.
