---
icon: mobile-rotate
---

# Service Discovery and DNS #2

#### Demo Senaryosu: Kubernetes'te Service Discovery ve DNS

Bu demoda, Kubernetes ortamında uygulamaların birbirini nasıl bulduğunu (Service Discovery) adım adım test edeceğiz. Öncelikle testlerimizi yapabilmek için arkada çalışan bir Nginx uygulaması ve ona ait bir Service oluşturacağız. Ardından yeni oluşturacağımız geçici bir test _Pod_'unun, bu Nginx servisini iki farklı yöntemle (Environment Variables ve DNS) nasıl keşfettiğini inceleyeceğiz.

**1. Ön Hazırlık: Uygulamanın ve Servisin Oluşturulması (YAML)**

Öncelikle Nginx _Pod_'umuzu ve ona erişmemizi sağlayacak `nginx-service` isimli _Service_ kaynağımızı yaratıyoruz. Aşağıdaki konfigürasyonu `nginx-app.yaml` adıyla kaydedip çalıştıracağız.

```yaml
# 1. Nginx uygulamasını çalıştıracak Pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80

---
# 2. Pod'a sabit bir IP ve isim sağlayacak Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP      # Varsayılan tip, sadece cluster içinden erişim
  selector:
    app: nginx         # Hangi Pod'lara trafik gideceğini belirler
  ports:
    - protocol: TCP
      port: 80         # Servisin dinlediği port
      targetPort: 80   # Pod'un içindeki uygulamanın portu
```

Bu dosyayı sisteme uygulayıp (apply) servisimizin durumunu kontrol edelim:

```bash
# YAML dosyasını cluster'a uyguluyoruz
kubectl apply -f nginx-app.yaml

# Servisin oluştuğunu ve IP aldığını doğruluyoruz
kubectl get svc nginx-service
```

Beklenen Çıktı:

```bash
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.103.206.194   <none>        80/TCP    1m
```

Çıktıdan anlıyoruz ki; Nginx servisimiz `10.103.206.194` IP adresiyle başarılı bir şekilde ayakta. Artık keşif (discovery) testlerimize başlayabiliriz.

***

**Yöntem 1: Environment Variables İle Keşif**

Kubernetes'in temel mekanizmalarından biri olarak _kubelet_, yeni bir _Pod_ başlatıldığında, o _Pod_ ile aynı namespace içindeki tüm aktif servislerin IP ve Port bilgilerini _Pod_'un içine çevre değişkeni olarak otomatik yazar.

Adım 1: Test Pod'unu Başlatma

İçine girip komut çalıştırabileceğimiz, işi bitince kendini otomatik silen (`--rm`) bir test _Pod_'u (CentOS) ayağa kaldırıyoruz:

```bash
kubectl run -i --tty --rm test-conn --image=centos -- bash
```

Adım 2: Değişkenleri Kontrol Etme

_Pod_'un içine girdikten sonra (shell ekranında), `nginx` kelimesi geçen çevre değişkenlerini listeliyoruz:

```bash
env | grep -i nginx
```

Beklenen Çıktı:

```bash
NGINX_SERVICE_PORT=tcp://10.103.206.194:80
NGINX_SERVICE_SERVICE_HOST=10.103.206.194
NGINX_SERVICE_SERVICE_PORT=80
```

_(Gördüğün gibi Kubernetes, servisin IP adresini ve portunu bizim için otomatik olarak Pod'un içine tanımlamış.)_

Adım 3: İstek Atma

Artık kodumuz, hedef IP adresini (10.103.206.194) ezberlemek veya _hardcode_ etmek zorunda kalmadan, doğrudan bu değişkenleri kullanarak Nginx'e erişebilir:

```bash
curl http://$NGINX_SERVICE_SERVICE_HOST:$NGINX_SERVICE_SERVICE_PORT
```

* Önemli Kısıtlama: Eğer bu test _Pod_'unu farklı bir _namespace_'te ayağa kaldırsaydık, bu değişkenler oluşmayacaktı. Ayrıca _Service_'in, _Pod_'dan daha önce oluşturulmuş olması zorunludur.

***

**Yöntem 2: Cluster DNS İle Keşif (Modern ve Standart Yöntem)**

Kubernetes içinde çalışan DNS sunucusu (CoreDNS), tüm _Cluster_ genelinde _Service_ isimlerini IP adreslerine çevirir. Bu yöntem çok daha esnektir ve sıralama veya _namespace_ kısıtlamalarına takılmaz.

Adım 1: DNS Ayarlarını İnceleme (`resolv.conf`)

Test _Pod_'unun içindeyken, Kubernetes'in DNS ayarlarını _Pod_'a nasıl işlediğine bakalım:

```bash
cat /etc/resolv.conf
```

Beklenen Çıktı:

```bash
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

Buradaki `nameserver` DNS sunucusunun adresidir. `search` satırı ise biz sadece "nginx-service" yazdığımızda sistemin DNS'e sorarken sonuna otomatik olarak ekleyeceği domain uzantılarını gösterir.

Adım 2: DNS İle İstek Atma (Farklı Namespace Simülasyonu)

Eğer Nginx servisimiz farklı bir _namespace_'te olsaydı, ona tam DNS adıyla (FQDN) şu şekilde ulaşabilirdik:

```bash
curl http://nginx-service.default.svc.cluster.local
```

Adım 3: DNS İle İstek Atma (Aynı Namespace)

Bizim senaryomuzda hem Nginx servisimiz hem de test _Pod_'umuz aynı _namespace_ (`default`) içinde. `resolv.conf` dosyasındaki `search` parametresi sayesinde uzun uzadıya domain yazmamıza gerek kalmaz. Sadece _Service_ adını yazmamız yeterlidir:

```bash
curl http://nginx-service
```

DNS sunucusu arka planda bu ismi saniyesinde `10.103.206.194` adresine çevirir ve trafiği doğru yere yönlendirir.

