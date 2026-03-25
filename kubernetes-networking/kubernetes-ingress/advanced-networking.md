---
icon: wave-sine
---

# Advanced Networking

İçerideki "A servisi", "B servisine" nasıl güvenli bağlanacak? Hangi servis yavaş çalışıp tüm sistemi kilitliyor? Yeni bir versiyonu müşterilerin sadece %10'una nasıl test ettiririz? İşte modern bulut mimarisinde bu devasa problemleri çözen iki "ileri seviye" konuyu, Service Mesh ve Multi-Cluster mimarilerini kaputun altına girerek inceleyeceğiz.

## Kubernetes'te Ustalık Sınavı: Service Mesh ve Multi-Cluster Mimarileri

Kubernetes'te uygulamalarınızı mikroservislere böldüğünüzde harika bir esneklik kazanırsınız. Ancak bu esneklik, beraberinde ağ karmaşasını getirir. Geleneksel yöntemlerde, bir servisin diğerini bulması, bağlantının şifrelenmesi veya hata durumunda tekrar denenmesi (retry) gibi ağ mantıklarının hepsini yazılımcının kodun içine bizzat yazması gerekirdi.

İşte Service Mesh, yazılımcıları bu ağ ameleliğinden kurtaran ve tüm bu işleri altyapı seviyesinde çözen sihirli bir katmandır.

### 1. Service Mesh Nedir ve Nasıl Çalışır?

Service Mesh'in çalışma mantığı çok zekicedir:

Service Mesh sistemini (Örn: Istio veya Linkerd) Cluster'ınıza kurduğunuzda, sistem her bir uygulamanızın (Pod) yanına görünmez bir "Ajan/Proxy" (genellikle Envoy) yerleştirir.

Artık "Muhasebe" servisi, "Ödeme" servisiyle doğrudan konuşamaz. Muhasebe servisi paketi kendi kapısındaki Ajan'a verir; Ajan paketi şifreler, Ödeme servisinin kapısındaki Ajan'a gönderir, o da paketi çözüp kendi servisine teslim eder.

Bütün bu Ajanlar (Data Plane), merkezdeki bir beyin (Control Plane) tarafından yönetilir.

#### Service Mesh Bize Ne Kazandırır?

1.  Canary Deployments:

    Ödeme servisinizin 2.0 versiyonunu yazdınız ama hemen herkese açmaya korkuyorsunuz. Service Mesh'e tek satır kural yazarsınız: _"Trafiğin %90'ını eski versiyona, %10'unu yeni versiyona (Canary) gönder."_ Yeni versiyonda hata yoksa, oranı yavaş yavaş artırırsınız.
2.  Zero-Trust ve mTLS Şifreleme:

    Eskiden K8s içindeki servisler birbirleriyle şifresiz (HTTP) konuşurdu. Biri Cluster'a sızarsa tüm trafiği okuyabilirdi. Service Mesh, iki ajan arasındaki tüm trafiği otomatik olarak mTLS (Karşılıklı TLS) ile şifreler. Üstelik sertifikaları kendi üretir ve kendi yeniler.
3.  Observability:

    Tüm trafik Ajanların üzerinden geçtiği için, Service Mesh size muazzam bir harita çıkarır. Hangi servis hangi servisle saniyede kaç kez konuşuyor? İstekler nerede gecikiyor (Distributed Tracing)? Hepsini canlı bir ekranda (Örn: Kiali, Jaeger) saniye saniye izlersiniz.

Popüler Service Mesh Araçları:

* Istio: Sektörün en yetenekli ve en popüler (ama öğrenmesi en zor) aracıdır.
* Linkerd: İnanılmaz hafif, çok hızlı ve kurulumu saniyeler süren harika bir alternatiftir.
* Cilium: Geleceğin teknolojisi olan eBPF'i kullanarak, Ajan (Sidecar) ihtiyacını bile ortadan kaldıran devrimsel bir araçtır.

***

### 2. Multi-Cluster

Kubernetes Cluster'ınız ne kadar sağlam olursa olsun, tek bir bölgeye (Örn: Sadece AWS Frankfurt) bağımlıysanız, o bölgede yaşanacak büyük bir elektrik kesintisinde tüm sisteminiz çöker. Veya Avrupa'daki müşterileriniz uygulamanıza çok hızlı erişirken, Amerika'dakiler gecikme (latency) yaşar.

Bunun çözümü Multi-Cluster mimarisidir. Uygulamalarınızı dünyanın farklı yerlerindeki (veya farklı bulut sağlayıcılarındaki) birden fazla Kubernetes Cluster'ına dağıtırsınız.

Neden Multi-Cluster Kullanmalıyız?

* Disaster Recovery: Bir Cluster tamamen çökse bile, trafik otomatik olarak diğer sağlam Cluster'a yönlenir (High Availability).
* Vendor Lock-in'den Kurtulma: Cluster'ın birini AWS'de, diğerini Google Cloud'da, bir diğerini ise kendi şirketinizdeki fiziksel sunucularda (On-Premise) tutabilirsiniz (Hybrid Cloud).
* İzolasyon ve Güvenlik: Müşteri verilerini işleyen (Regülasyona tabi) servisleri tamamen izole edilmiş, ekstra güvenli ayrı bir Cluster'da çalıştırabilirsiniz.

***

### 3. Zirve Noktası: Service Mesh + Multi-Cluster Birleşimi

Tek başlarına harika olan bu iki teknolojiyi birleştirdiğinizde, modern bulut mimarisinin zirvesine ulaşırsınız.

Özellikle Cilium Cluster Mesh gibi modern araçlarla iki farklı Cluster'ı birbirine bağladığınızda şu sihir gerçekleşir:

Londra'daki Cluster'da çalışan bir uygulamanız (Pod), sanki aynı odadalarmış gibi, New York'taki Cluster'da çalışan bir veritabanı ile şifreli (mTLS), güvenli ve doğrudan iletişim kurabilir. Londra'daki veritabanı çökerse, Service Mesh trafiği anında (hiçbir kesinti hissettirmeden) New York'taki veritabanına kaydırır.

### Özet

Ingress ve ExternalDNS ile dışarıdan gelen müşterilere kapılarımızı kusursuzca açmıştık. Service Mesh ile içerideki mikroservis ordumuzu güvenli ve akıllı bir şekilde yönetmeyi; Multi-Cluster ile de bu devasa orduyu dünyanın dört bir yanına yıkılmaz bir şekilde dağıtmayı öğrenmiş olduk.



