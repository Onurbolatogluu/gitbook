---
icon: democrat
---

# Demo

Kubernetes'in sihir gibi görünen otomatik trafik yönlendirme yeteneğinin arkasında aslında sürekli güncellenen dinamik bir liste vardır. Bu makalede, standart bir web uygulaması (Nginx) kurarak sistemin trafik yönlendirme haritasını (Endpoints) nasıl oluşturduğunu ve biz uygulamayı büyüttükçe (Scale) bu haritayı nasıl güncellediğini adım adım inceleyeceğiz.

### 1. Hazırlık: Uygulamayı ve Servisi Ayağa Kaldırmak

Öncelikle trafiği karşılayacak bir uygulamaya ve bu trafiği içeri alacak bir Service'e ihtiyacımız var. Arka planda 1 replica'dan oluşan bir Nginx Deployment'ı ve bunu sarmalayan bir `ClusterIP` servisi oluşturduğumuzu varsayalım.

Sistemimizin ilk durumunu kontrol edelim:

```bash
# Çalışan Pod'umuzu görelim
kubectl get pods

# ÇIKTI:
# NAME                                READY   STATUS    AGE
# nginx-deployment-7c79c4bf97-z79j4   1/1     Running   5m

# Servisimizi görelim
kubectl get services

# ÇIKTI:
# NAME            TYPE        CLUSTER-IP      PORT(S)   AGE
# nginx-service   ClusterIP   10.98.220.254   80/TCP    5m
```

Her şey hazır. `nginx-service` adında bir kapımız var. Peki bu kapı, arkadaki o tek Pod'un IP'sini biliyor mu?

### 2. Klasik Adres Defterini (Endpoints) İncelemek

Kubernetes'in biz servisi yarattığımız an arka planda aynı isimle oluşturduğu Endpoints'e bakalım:

```bash
kubectl get endpoints nginx-service
```

Çıktı:

```bash
NAME            ENDPOINTS      AGE
nginx-service   10.0.0.55:80   5m
```

Harika! Sistem, Pod'umuzun gerçek IP adresini (`10.0.0.55`) ve portunu bulup rehbere yazmış.

Peki bu Endpoints'in içinde tam olarak neler var? Detaylarına inelim:

```bash
kubectl describe endpoints nginx-service
```

Çıktı Detayı:

```bash
Name:         nginx-service
Namespace:    default
Subset:
  Addresses:  10.0.0.55
  Ports:
    Name   Port  Protocol
    nginx  80    TCP
```

Şu an trafik geldiğinde sistem doğrudan bu listedeki tek adrese yönelecek.

### 3. Sistemi Büyütmek (Scaling) ve Otomatik Güncelleme

Gerçek dünyada sistemler sabit kalmaz. Kampanya dönemindeyiz ve sitemize inanılmaz bir trafik gelmeye başladı. Manuel hiçbir Load Balancer ayarı yapmadan, sunucu sayımızı 1'den 2'ye çıkaralım:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Yeni Pod'umuz saniyeler içinde ayağa kalktı. Gidip Endpoints rehberimize tekrar bakalım, Kubernetes yeni gelen sunucuyu fark etmiş mi?

```bash
kubectl describe endpoints nginx-service
```

Çıktı Detayı:

```bash
Subset:
  Addresses: 10.0.0.55,10.0.1.248
  Ports:
    Name   Port  Protocol
    nginx  80    TCP
```

İşte Kubernetes'in gücü. Biz hiçbir yere IP adresi yazmadık, hiçbir ayar dosyasını değiştirmedik. Sistem yeni Pod'un IP'sini (`10.0.1.248`) anında tespit etti ve trafiği artık bu iki cihaz arasında paylaştırmaya başladı.

_(Eğer Pod'lardan birini `kubectl delete pod` komutuyla silerseniz, o IP'nin saniyesinde bu listeden çıkarıldığını ve müşterilerin hata almasının engellendiğini görebilirsiniz.)_

### 4. Yeni Nesil Mimariyi (Endpoint Slices) İncelemek

Bildiğiniz gibi `Endpoints` nesnesi 1000 IP sınırına sahipti. Sistemimiz devasa boyutlara ulaştığında (örneğin 10.000 Pod) Kubernetes artık listeyi bölmek için Endpoint Slices kullanır.

{% hint style="info" %}
Pod sayın ister 2 olsun, ister 10.000 olsun fark etmez; modern Kubernetes'in ağ yöneticisi (kube-proxy) trafiği yönlendirirken artık her zaman sadece `Endpoint Slices` dosyasına bakar. Çünkü Topoloji (yakınlık) gibi akıllı özellikler sadece Slices'ın içinde vardır. Eski `Endpoints` nesnesi sadece "eski sistemler ağlamasın" diye kenarda güncellenmeye devam eden nostaljik bir dosyadan ibarettir.
{% endhint %}

Şu anki küçük sistemimizde bile Kubernetes'in arka planda bizim için hazırladığı bu yeni nesil dilime göz atalım:

```bash
kubectl get endpointslices
```

Çıktı:

```bash
NAME                  ADDRESSTYPE   PORTS   ENDPOINTS             AGE
nginx-service-87c5j   IPv4          80      10.0.0.55,10.0.1.248  7m
```

Dikkat ederseniz ismin sonuna rastgele bir kod (`-87c5j`) eklenmiş. Çünkü bu, listeyi bölecek olan binlerce dilimden sadece ilki.

İçine girip YAML formatında bakalım, önceki derslerimizde konuştuğumuz o "Topoloji" (Pod nerede çalışıyor?) bilgisini kendi gözlerimizle görelim:

```bash
kubectl get endpointslice nginx-service-87c5j -o yaml
```

Kritik YAML Çıktısı:

```yaml
endpoints:
- addresses:
  - 10.0.0.55
  conditions:
    ready: true               # ÖNEMLİ: Cihaz sağlıklı, trafik gönderebilirsin.
  nodeName: node01            # ÖNEMLİ: Topoloji! Bu Pod fiziksel olarak "node01" sunucusunda.
  targetRef:
    kind: Pod
    name: nginx-deployment-7c79c4bf97-z79j4
- addresses:
  - 10.0.1.248
  conditions:
    ready: true
  nodeName: node02            # Bu Pod ise "node02" sunucusunda.
```

Bu Çıktı Bize Ne Anlatıyor?

1. `ready: true`: Kubernetes sadece IP'yi listeye eklemekle kalmaz, uygulamanın gerçekten çalışıp çalışmadığını (Health Check) kontrol eder. Eğer uygulama kilitlenirse burası `false` olur ve kube-proxy o IP'ye trafik göndermeyi anında keser.
2. `nodeName`: Bir önceki yazımızda bahsettiğimiz "Trafiği en yakındakine gönder" (Topology Aware Routing) mantığının kanıtı buradadır. Endpoint Slices, her IP'nin fiziksel olarak hangi sunucuda barındığını harfiyen bilir.

#### Özet

Siz geliştirici olarak sadece bir YAML dosyası yazıp `apply` dersiniz. Ancak arka planda Kubernetes:

1. Pod'larınızı bulur,
2. Sağlık durumlarını kontrol eder,
3. Hangi fiziksel sunucuda olduklarını haritalandırır,
4. Bunları dilimler (Slices) halinde paketleyip tüm Node'lara dağıtır.

Ve tüm bunları milisaniyeler içinde yapar!
