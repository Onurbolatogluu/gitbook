---
icon: server
---

# Giriş : Kubernetes Services

#### 1. Temel Problem: Neden "Service" Kullanıyoruz?

Kubernetes ortamında uygulamalarımız Pod'lar içinde çalışır. Ancak Pod'lar geçicidir (ephemeral); çökerlerse silinip yeniden yaratılırlar ve her yeniden yaratıldıklarında IP adresleri değişir.

Eğer `frontend` uygulaman, `backend` uygulamana bağlanmak için doğrudan Pod'un IP adresini kullanırsa, `backend` Pod'u yeniden başladığında IP değişeceği için bağlantı kopar.

İşte Service bu sorunu çözer. Service, arkasındaki Pod'ların IP'leri sürekli değişse bile onlara ulaşmak için sabit bir IP adresi ve isim (DNS name) sağlayan bir ağ bileşenidir.

#### 2. Service, Hangi Pod'a Gideceğini Nasıl Anlar?

Service, gelen trafiği doğru Pod'lara yönlendirmek için Selector kullanır. YAML dosyasında `selector: app: backend` tanımını yaptığında Service, ortamdaki `app: backend` etiketine (label) sahip tüm Pod'ları bulur. Arka planda Kubernetes, EndpointSlices adı verilen bir mekanizma ile bu Pod'ların anlık IP adreslerini sürekli takip eder ve günceller.

#### 3. Service Discovery (Uygulamalar Servisleri Nasıl Bulur?)

Cluster içindeki bir Pod (örneğin `frontend`), hedefine (örneğin `backend-service`) ulaşmak için IP aramaz. Bunu iki yöntemle çözer:

* Environment Variables (Statik Yöntem): Bir Pod yaratıldığı ilk saniyede, cluster'da o an var olan tüm Service'lerin IP bilgileri Pod'un içine ortam değişkeni olarak yazılır. Pod'un içine girip `env` komutunu çalıştırırsan şunu görürsün: `BACKEND_SERVICE_HOST=10.96.0.5` _Dezavantajı:_ Pod çalışmaya devam ederken cluster'a yeni bir Service eklenirse, eski Pod'un bu yeni servisten haberi olmaz. Değişkenlerin güncellenmesi için Pod'un silinip yeniden başlatılması gerekir.
* DNS Yöntemi (Dinamik ve Modern Yöntem): Ortam değişkenleriyle uğraşılmaz. Kubernetes içindeki CoreDNS komponenti, tüm Service'lerin isimlerini ve anlık IP'lerini bilir. Sen `frontend` Pod'unun içinden terminale sadece şu komutu yazarsın: `curl http://backend-service:8080` İstek CoreDNS'e gider, CoreDNS anında `backend-service` ismini güncel IP'ye çevirir ve trafiği ulaştırır. Sisteme yeni bir Service eklendiğinde hiçbir Pod'u yeniden başlatmana gerek kalmaz.

#### 4. Service Tipleri ve Kullanım Alanları

İhtiyaca göre trafiğin nereden geleceğini belirleyen 4 temel Service tipi vardır:

1\. ClusterIP (Varsayılan)

* Ne Yapar? Servise sadece cluster içinden erişilebilecek bir dahili IP atar. Dışarıdan erişim tamamen kapalıdır.
* Kullanım Alanı: Sadece içerideki uygulamaların erişmesi gereken veritabanları veya backend servisleri için kullanılır.

2\. NodePort

* Ne Yapar? Servisi, cluster'daki tüm Node'ların (sunucuların) kendi IP adresleri üzerinden, sabit bir portla (30000-32767 arası) dışarı açar.
* Kullanım Alanı: Geliştirme ve test süreçleri. Örneğin MacBook'unda yerel bir cluster çalıştırırken tarayıcından veya terminalden hızlıca `curl http://<Node-IP>:30005` yaparak uygulamaya erişmek için kullanırsın.

3\. LoadBalancer

* Ne Yapar? Bulut sağlayıcısı (AWS, Google Cloud vb.) üzerinde fiziksel/sanal bir Load Balancer oluşturur ve sana gerçek bir Public IP verir.
* Kullanım Alanı: Production ortamında web arayüzünü internetteki gerçek kullanıcılara açmak için kullanılır.

4\. ExternalName

* Ne Yapar? Trafiği cluster içindeki bir Pod'a değil, dışarıdaki bir DNS adresine (CNAME) yönlendirir.
* Kullanım Alanı: Cluster içindeki uygulamanı buluttaki yönetilen bir veritabanına (örn: AWS RDS) bağlamak için. Koda uzun AWS adresi yazmak yerine, Service ismini yazarsın.

#### Örnek Bir Service YAML Dosyası

Bir Jenkins pipeline'ı aracılığıyla uygulamanı deploy ederken kullanacağın tipik bir `ClusterIP` Service tanımı şu şekildedir:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-backend-service
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: backend-app     # Trafiği bu etikete sahip Pod'lara gönderir
  ports:
    - protocol: TCP
      port: 80           # Diğer Pod'ların bu servise gelirken kullanacağı port (örn: curl http://my-backend-service:80)
      targetPort: 8080   # Trafiğin Pod'un içindeki uygulamaya iletileceği asıl port
```

Bu dosyayı `kubectl apply -f service.yaml` ile çalıştırdığında Service oluşur ve ağ trafiğini yönetmeye başlar.
