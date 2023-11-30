---
description: AWS dünyasının CDN hizmeti
---

# 🎆 CloudFront

AWS yüzlerce lokasyona yayılmış CDN hizmet noktalarına edge locations deniliyor. Hizmeti devreye aldığımız zaman kullanıcıların statik veriye kendilerine en yakın edge lokasyonundan erişmeye başlıyor.&#x20;

#### Hizmetleri

* Statik dosyaların önbelleklenmesi (caching)
* Web formları, yorum ve giriş kutuları (sepete ekle) düğmelerini ve ya son kullanıcılardan veri yükleyen diğer özelliklere sahip dinamik içerikleri de cacheler.
* Canlı yayınları, talebe bağlı videoları, cache yapabiliriz.
* HTTP get-head-post-put-delete-options-patch- metotlarını destekler.
* Api isteklerini hızlandırır. SSL-TLS desteği vardır.
* AWS Shield ile entegre layer 3-4 ddos mitigation ve AWS waf ile entegre layer 7 koruması mevcuttur.
* Lambda desteği mevcuttur.

#### CloudFront Oluştururken,

* Create distribution
* Web ve ya, RTMP tabanlı mı olacak? Burada bunu seçiyoruz.

#### Origin Settings

Origin Domain Name : CDN ' hangi sunucudan(servisten) verileri alacak. Örnek bir LB sunucusu vb.

Origin Path : Sunucuya direkt / path 'ine mi gidilsin? Yoksa örnek olarak /image klasörü vb alt klasöre mi gidilsin? Bunu burada belirtiyoruz.

Origin ID : Bu CDN ağının unic adı, istersek değiştirebiliriz.

Origin SSL Protocols : Cloudfront ile kaynak sunucu eğer ssl protokolü kullanacaksa, hangi protokole izin vereceksek bunu seçiyoruz. Buna göre trafik edge ve origin arasında belirttiğimiz protokol ile gerçekleşecek.

Origin Protocol Policy : Cloudfront ile kaynak sunucu arasındaki bağlantının, http mi, https mi bunu seçiyoruz. Match viewer seçeneği, misal kullanıcı websitemize https geliyorsa, cloudfront'da origine https olarak gidecektir.

Origin Response Timeout : Cloudfront ile origin sunucudan kaç saniye boyunca cevap bekleyeceğini belirtiyoruz. Buraya 30 yazarsak, cloudfront bir veri için origine gider 30 saniye boyunca cevap alamazsa timeout alır.

Origin Keep-alive Timeout : Kaç saniye haberleşme olmasa da bağlantının korunacağını belirliyoruz.

HTTP Port : Origin sunucumuz 80'den değilde 8080den cevap veriyorsa, bunu belirtiyoruz.&#x20;

Origin Custom Headers : Kaynak sunucuya her istek gönderildiğinde eklemek istediğimiz bir HTTP header bilgisi olup olmadığını belirtiyoruz. Özellikle birden fazla CDN kullanırken paketin nereden geldiğini HTTP loglarında görmek için önemli bir durumdur.

Buraya kadar bahsettiklerimiz Cloudfront ile origin sunucunun nasıl haberleştiği ile alakalıdır.

#### CloudFront (Cache behavior settings)

Buradaki ayarlar, cloudfront servisinin caching ile ilgili ayarlarıdır.

Path Pattern : Nelerin cache'leneceğini belirtiyoruz. "\*" seçili ise tüm statik içerikler cachelenecek.

Viewer Protocol Policy : Cloudfront 'un dış dünyadan gelen isteklere göre nasıl davranması gerektiğini söylüyoruz. \
\
HTTP and HTTPS > İki protokole de izin ver.\
Redirect HTTP to HTTPS > HTTP'den gelen istekleri HTTPS'e yönlendir.\
HTTPS only : Sadece HTTPS çalışır.

Allowed HTTP Methods : Örnek olarak kullanıcılar origin sunucu ile hangi HTTP metotlarıyla iletişim kurmasını istiyorsak bunu seçiyoruz.

Cached HTTP Methods : Hangi HTTP metotlarının cachelenmesini istiyorsak burada belirtiyoruz.

Cache based on Selected Request Headers : İstek headerlarına göre cacheleme yapmak istiyorsak, bir alt sütunda bunu seçebiliriz.

Object Caching : Cloudfront cache yaparken dosyaların, ne kadar zaman tutulacağına 2 şekilde karar veriyor.

1 - Cache ayarlarını origin sunucu üzerinden yapabiliriz. Sunucumuz bu değerleri cache-control header bilgisi ile cloudfront a gönderiyor. Cloudfront buradaki değerlere göre bir objeyi ne kadar cache'de tutacağını buna göre karar veriyor.

2 - Cache süresi vb ayarları cloudfront'un karar vermesini sağlayabiliriz. (customize)

Minimum TTL : Bir objenin cache'de kalabileceği minimum süreyi belirtir.

Maximum TTL : Bir objenin cache'de kalabileceği maksimum süreyi belirtir.\
\
Origin sunucu header bilgilerinde belirterek header bilgisinde yer alan süre zarfında cacheleme isteği gönderebiliyor.\
Origin bir değer belirtmediyse Default TTL de belirtilen süre boyunca obje cachelenebiliyor.\
Time To Live yani TTL süresi boyunca obje için origin sunucuya aynı obje için istek gönderilmiyor.

Smooth Streaming : Canlı bir yayın dağıtıyorsak seçebiliriz.

Restrict viewer access(signed URL or Signed cookies) : Diyelim ki bir eğitim firmasıyız, Müşterilere eğitim videoları satıyoruz. Ve ya bir yazılım firmasıyız,  müşterilere yazılım paketleri satıyoruz. Bir uygulama yazdık ve müşterilere bu paketleri ya da videoları, geçici URLler üzerinden gönderiyor. URL her seferinde yeniden oluşturuluyor ve sadece URL li alan kullanıcı belirli bir zaman aralığında buna erişebiliyor. Yani biz müşterilerimize, URLler arayıcılığıyla sadece onun erişebileceği içerik gönderip, işin bitince de bu içeriği expried edebiliyoruz. Bunu signed URL or Signed cookies ile yapıyoruz. Bu işlemleri uygulamanın içine gömer, Buradaki seçeneği de aktif edersek,  Cloudfront bu tarz dağıtımlar yapmamıza izin verir.

{% embed url="https://www.youtube.com/watch?ab_channel=Bitfumes&v=oPKuRsemap0" %}

{% embed url="https://www.youtube.com/watch?ab_channel=visualskyman&v=5RELL2nhY_Q" %}

Compress objects Automatically :  Cache'lenen obje sıkıştırılmaya izin veriyorsa, Cloudfront bu objeleri sıkıştırılmasını sağlıyor.

Lambda Function Associations : Lambda fonksiyonunu tetiklememizi sağlar. Bir web sitemiz var diyelim, bu web sitesinin yenisini yaptık test etmek için de kullanıcıların %90nını eski siteye, %10nunu yeni siteye gitsin. Buna uygun bir kod yazıp lambdaya yükleriz. Her gelen isteğin bu kodun tetiklenmesini sağlayabiliriz.

#### Ana Ayarlar,

Price Class : Cloudfront'un dünyada hangi edge bölgelerine yayılacağını belirtiyoruz.

AWS WAF web ACL : Cloudfront'un waf hizmetini kullanabiliriz.

Alternate Domain names (cnames) : Cloudfront da kullanacağmız domain ismini giriyoruz. web sitemiz www.test.com ise bunu belirtiyoruz.

SSL cert : Eğer biz sitemizi HTTPS protokolü ile yayınlamak istersek, sertifika seçmemizi istiyor. Cloudfront bunu bize kendisi de sağlıyor.

#### Distribution Settings

Restrictions  : geo restrictions : Ülke bazlı engelleme yapabiliriz.

invalidations : Cachelenen bir içeriği cache'ini temizleyebiliriz. Örnek olarak origin sunucuda bir resim dosyası var ve sonradan bunu sildik. Fakat cloudfront bu resmi halen cache'inde tutuyor. İlgili resmin path'ini (yolunu) girip cache'ini temizleyebiliriz. /images/deneme.jpg gibi..
