---
icon: apartment
---

# External DNS Overview

Diyelim ki sisteme yeni bir mikroservis eklediniz ve bir Ingress yazdınız. Bulut sağlayıcınız size yeni bir LoadBalancer IP'si verdi. İnsanların bu servise `api.sirketim.com` üzerinden ulaşabilmesi için ne yaparsınız? Gidip Cloudflare, AWS Route 53 veya Google Cloud DNS paneline elinizle giriş yapar, yeni bir "A Kaydı" oluşturur ve o IP'yi oraya yapıştırırsınız.

Peki ya sisteminizde 500 tane mikroservis varsa? Ya bulut sağlayıcınız LoadBalancer IP'nizi bir gün aniden değiştirirse? Sistem anında çöker ve siz gece yarısı uyanıp panellerde IP güncellemek zorunda kalırsınız.

İşte bu manuel eziyeti bitiren ve Kubernetes'i gerçek anlamda "otonom" bir bulut sistemine dönüştüren o harika aracın adı: ExternalDNS.

***

## Kubernetes'te DNS Otomasyonu: ExternalDNS ile Manuel İşlemlere Son

Kubernetes ortamında yeni bir uygulama (Service veya Ingress) ayağa kaldırdığınızda, bu uygulamanın dış dünyadan erişilebilir olması için bir alan adına (Domain) ihtiyacı vardır. Geleneksel dünyada sistem yöneticileri DNS panellerine (Cloudflare, Route 53 vb.) girip bu kayıtları elle oluştururlar.

Ancak modern ve dinamik Cloud-Native dünyasında "manuel" kelimesine yer yoktur. IP'ler değişebilir, servisler silinip baştan kurulabilir. İşte ExternalDNS, bu DNS kayıtlarını sizin yerinize otomatik olarak yöneten bir Kubernetes aracıdır.

### ExternalDNS Nasıl Çalışır?

ExternalDNS, Kubernetes cluster'ınızın içinde sürekli çalışan sinsi ve zeki bir gözlemcidir (Pod). Tek yaptığı iş şudur:

1. Kubernetes API'sini 7/24 dinler.
2. Birisi sisteme yeni bir `Ingress` veya `Service` eklediğinde, dosyanın içine bakar.
3. Eğer dosyada `external-dns.alpha.kubernetes.io/hostname: ornek.com` şeklinde özel bir not (annotation) görürse harekete geçer.
4. Gider AWS Route 53, Cloudflare veya Google Cloud DNS hesabınıza API üzerinden (otomatik olarak) bağlanır.
5. Sizin hiçbir şeye tıklamanıza gerek kalmadan `ornek.com` adresini, uygulamanızın çalıştığı güncel IP adresine yönlendiren DNS kaydını saniyeler içinde oluşturur.

Eğer o uygulamayı (Ingress'i) silerseniz, ExternalDNS de gider ve DNS panelindeki o kaydı anında temizler. Artık "Acaba kullanılmayan eski IP'ler DNS'te çöp olarak kaldı mı?" derdiniz olmaz.

### Gerçek Dünya Sorusu: "Peki Benim Manuel Eklediğim Kayıtları Bozar Mı?"

ExternalDNS'i sisteme kurarken herkesin aklına gelen o korkutucu soru şudur: _"Ya bu bot çıldırır da Cloudflare'deki şirketimin ana web sitesinin veya mail sunucularımın (MX) DNS kayıtlarını silerse?"_

ExternalDNS geliştiricileri bu felaketi önlemek için harika bir "Sahiplik (Ownership)" mekanizması kurmuşlardır. Buna TXT Registry denir.

ExternalDNS, sizin adınıza `api.ornek.com` için bir A kaydı oluşturduğunda, hemen yanına gizli bir `TXT` (Metin) kaydı daha oluşturur. Bu TXT kaydının içine kendi imzasını (Owner-ID) atar.

Bir DNS kaydını değiştireceği veya sileceği zaman önce bakar: _"Bunun TXT imzasını ben mi atmışım?"_ Eğer imza kendisine aitse işlemi yapar. Ama kaydı siz elinizle panele girip oluşturduysanız (yanında ExternalDNS imzası yoksa), o kayda asla ama asla dokunmaz.

### ExternalDNS Nasıl Kullanılır?

ExternalDNS'i kurduktan sonra tek yapmanız gereken, Ingress YAML dosyanıza ufak bir not (annotation) eklemektir. Gerisini sistem halleder:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: benim-uygulamam
  annotations:
    # İŞTE SİHİR BURADA: ExternalDNS bu notu okur ve DNS paneline gider.
    external-dns.alpha.kubernetes.io/hostname: "api.sirketim.com"
spec:
  ingressClassName: nginx
  rules:
  - host: "api.sirketim.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
```

### Kimlik ve Yetki (IAM)

Daha önceki Ingress Controller yazımızda bahsettiğimiz kural burada da geçerlidir. ExternalDNS'in dış dünyadaki Route 53 veya Cloudflare panellerine müdahale edebilmesi için yetkiye ihtiyacı vardır.

Bu aracı (Helm ile) kurarken, ona mutlaka bulut sağlayıcınızın API Token'ını veya IAM Rolünü vermelisiniz. Aksi takdirde ExternalDNS K8s içindeki Ingress'leri görür ama gidip Cloudflare'de kayıt açmaya çalıştığında "Yetkisiz Erişim (Access Denied)" hatası alıp kilitlenir.

### Özet

ExternalDNS, Kubernetes ağ mimarisinin son ve en konforlu halkasıdır. Ingress Controller'ınız trafiği yönlendirirken, ExternalDNS o trafiğin yol tabelalarını (Domain isimlerini) otomatik olarak çakar.&#x20;

***

Gerçekten çok haklısın! Önceki yazıda "TXT sahipliği" veya "sadece Ingress'leri dinler" gibi kavramların mantığını anlattım ama motoru çalıştırırken bu ayarları hangi komutlarla (parametrelerle) yaptığımızı atlamışım. Bu eksiklik makalenin teknik derinliğini azaltmış, yakaladığın için harika oldu!

Ekran görüntülerinde yer alan bu yapılandırma argümanları (Configuration Arguments), ExternalDNS motoruna "Neye bakacaksın, nereye yazacaksın ve sınırların neler?" komutlarını verdiğimiz kontrol panelidir.

Makalene doğrudan "ExternalDNS'in Kontrol Paneli: Yapılandırma Parametreleri" başlığıyla ekleyebileceğin, görsellerdeki tüm komutların detaylı açıklamalarını aşağıda hazırladım:

***

### ExternalDNS'in Kontrol Paneli: Yapılandırma Parametreleri

ExternalDNS'i kurarken (Helm `values.yaml` içinde veya Deployment `args` kısmında) araca nasıl davranması gerektiğini söyleyen bazı kritik komutlar (argümanlar) veririz. Bu parametreleri 4 ana grupta inceleyebiliriz:

#### 1. Genel Argümanlar

Motorun Kubernetes içindeki temel çalışma prensiplerini belirler.

* `--source`: ExternalDNS'in Kubernetes içinde hangi kaynakları dinleyeceğini seçeriz. Örneğin `--source=ingress` yazarsak sadece Ingress dosyalarındaki notları okur. `--source=service` yazarsak LoadBalancer IP'lerini de takip eder. İkisini birden de verebiliriz.
* `--namespace`: Motorun yetki alanını daraltır. Eğer sadece belirli bir namespace'teki (Örn: `production`) Ingress'leri dinlemesini istiyorsanız bu parametreyi kullanırsınız. Yazmazsanız tüm cluster'ı dinler.
* `--provider`: ExternalDNS'e hangi şirketle (Cloudflare, AWS Route53, GoDaddy, Google) konuşacağını söyleriz. (Örn: `--provider=cloudflare`).
* `--policy`: En kritik güvenlik ayarlarından biridir. ExternalDNS'in yetkilerini sınırlar.
  * `sync`: Tam yetkidir. Kayıt oluşturur, günceller ve Ingress'i silerseniz gidip DNS kaydını da siler.
  * `create-only` (veya `upsert-only`): Sadece yeni kayıt açabilir veya güncelleyebilir ama asla silemez. Yanlışlıkla DNS silinmesinden korkulan ortamlarda kullanılır.

#### 2. DNS Sağlayıcı Argümanları

Dışarıdaki sisteme bağlanırken kullanılan sınırlamalardır.

* `--domain-filter`: Hayat kurtaran güvenlik filtresidir. Motorun kafası karışıp veya hatalı bir YAML yazılıp şirketinizin ana domainlerinin bozulmasını engeller. Örneğin `--domain-filter=bolatogluyapi.com` derseniz, ExternalDNS başka hiçbir alan adına (Örn: `google.com` yazılsa bile) dokunamaz.
* Provider Specific: AWS için `--aws-zone-type=public` veya Google için `--google-project=my-project` gibi sadece o bulut sağlayıcısına özel ekstra teknik ayarlardır.

#### 3. Güvenlik ve Sahiplik

ExternalDNS'in başkasının kayıtlarını bozmasını engelleyen kimlik sistemidir.

* `--registry`: Sahiplik sisteminin nasıl tutulacağını belirler. Sektör standardı `--registry=txt` kullanılmasıdır. A kaydının yanına bir de gizli TXT kaydı atmasını sağlar.
* `--txt-owner-id`: Oluşturulan o gizli TXT kaydının içine atılacak imzadır. Örneğin `--txt-owner-id=benim-canli-clusterim` derseniz, ExternalDNS sadece bu imzaya sahip olan DNS kayıtlarını günceller veya siler. Sizin elinizle açtığınız kayıtlara (imzası olmadığı için) dokunmaz.

#### 4. Gelişmiş Ayarlar

Büyük sistemlerde otomasyonu daha da akıllandırmak için kullanılır.

* `--annotation-filter`: Sadece belirli bir notu taşıyan Ingress'leri işleme almasını sağlar. Örneğin, sadece `external-dns-yonetsin: "evet"` notu olan Ingress'leri okur, diğerlerini görmezden gelir.
* `--fqdn-template`: Yazılımcıları YAML dosyasına `hostname` yazma derdinden kurtarır. Bir şablon belirlersiniz (Örn: `--fqdn-template={{.Name}}.bolatogluyapi.com`). Yazılımcı sadece `api-service` adında bir servis açar, ExternalDNS ismi otomatik alıp `api-service.bolatogluyapi.com` olarak DNS'e kaydeder.
