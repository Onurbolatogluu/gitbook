---
icon: server
---

# Giriş : Kubernetes Services

#### Kubernetes "Service" Nedir? Neden İhtiyacımız Var?

Kubernetes ortamında uygulamalarımız Pod'lar içerisinde çalışır. Ancak Pod'lar doğaları gereği "geçicidir" (ephemeral). Bir Pod hata verip çökebilir, güncellenirken silinebilir veya sistemdeki yük arttığında sayısı çoğaltılabilir (scale).

Bir Pod her yeniden yaratıldığında yeni bir IP adresi alır.

Şimdi şöyle bir senaryo düşün: Bir _Frontend_ uygulaman var ve veri çekmek için _Backend_ uygulamanın çalıştığı Pod'a bağlanması gerekiyor. Eğer Backend Pod'u çöker ve yerine yenisi gelirse IP adresi değişecektir. Frontend, eski IP'yi arayacağı için sistem çöker.

İşte Service tam burada hayat kurtarır. Service, arkadaki Pod'lar sürekli değişse, silinse veya yenileri eklense bile, onlara ulaşmak için sabit bir IP adresi ve bir DNS adı sağlar. Service, gelen istekleri alır ve arkada ayakta olan, sağlıklı (healthy) Pod'lara dağıtır. Yani içeride bir nevi "Load Balancer" görevi görür.

#### Bir Service Nasıl Tanımlanır?

Bu, Kubernetes'e "bana bir Service oluştur" dediğimiz konfigürasyon dosyasıdır.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

{% hint style="info" %}
Eğer YAML dosyasında `spec` altında herhangi bir type (Service tipi) belirtmezsen, Kubernetes varsayılan olarak bunu ClusterIP olarak oluşturur.
{% endhint %}

Bu dosyanın en can alıcı kısımlarını basitçe açıklayalım:

* `kind: Service`: Kubernetes'e oluşturacağımız kaynağın bir Service olduğunu söylüyoruz.
* `name: backend-service`: Bu Service'in adı. Cluster içindeki diğer uygulamalar (örneğin Frontend), backend'e ulaşmak için IP adresi ezberlemek yerine direkt `http://backend-service` adresine istek atabilirler.
* `selector` (`app: backend`): En kritik nokta burasıdır. Service, "Ben hangi Pod'lara trafik göndereceğim?" sorusunun cevabını burada bulur. Üzerinde `app: backend` etiketi (label) olan tüm Pod'ları bulur ve trafiği onlara yönlendirir.
* `port: 80`: Service'in kendisinin dinlediği port numarasıdır.
* `targetPort: 8080`: Arkadaki Pod'un içinde çalışan asıl uygulamanın (örneğin Java veya Node.js uygulamanın) dinlediği porttur. Service, 80 portundan aldığı isteği, Pod'un 8080 portuna iletir.

#### Service Tipleri ve Kullanım Alanları

**1. ClusterIP (Varsayılan Tip)**

Bir Service oluştururken tip belirtmezsen, otomatik olarak `ClusterIP` olur.

* Ne Yapar?: Sadece _Cluster_ içerisinden erişilebilen bir IP verir. Dış dünyadan (internetten) kimse bu Service'e ulaşamaz.
* Kullanım Alanı: Dışarıya kapalı olması gereken iç iletişimler için kullanılır. Örneğin; _Frontend_'in _Backend_ ile konuşması veya _Backend_'in _Database_ ile konuşması gereken durumlarda kullanılır. Veritabanını internete açmak istemezsin, sadece içerideki Backend ulaşsın istersin.

**2. NodePort**

* Ne Yapar?: Uygulamanı dış dünyaya açmanın en temel yoludur. _Cluster_ içindeki her bir _Node_'un (sunucunun) üzerinde statik, spesifik bir port açar (genellikle 30000 ile 32767 arasında bir port). Dışarıdan biri `Node_IP_Adresi:Acilan_Port` şeklinde erişebilir.
* Kullanım Alanı: Kendi lokal bilgisayarında geliştirme yaparken veya hızlıca dışarıdan bir test yapmak istediğinde kullanılır. Ancak güvenlik ve esneklik açısından _production_ ortamları için pek tavsiye edilmez.

**3. LoadBalancer**

* Ne Yapar?: Uygulamanı internete (dış dünyaya) açmanın en yaygın ve standart yoludur. AWS, Google Cloud veya Azure gibi bir bulut sağlayıcı kullanıyorsan, bu tipte bir Service oluşturduğunda, bulut sağlayıcın gidip sana otomatik olarak gerçek bir fiziksel/sanal Load Balancer cihazı oluşturur ve uygulamanı dışarıdan gelen kullanıcılara açar.
* Kullanım Alanı: _Production_ ortamında kullanıcıların web sitene tarayıcılarından girebilmesini sağlamak için kullanılır.

**4. ExternalName**

* Ne Yapar?: Bu biraz farklı çalışır. Trafiği senin _Pod_'larına yönlendirmek yerine, _Cluster_ içindeki uygulamalarının, _Cluster dışındaki_ bir adrese gitmesini sağlar. Arkasında bir IP veya Pod tutmaz, sadece bir CNAME (DNS yönlendirmesi) kaydı görevi görür.
* Kullanım Alanı: Diyelim ki veritabanını Kubernetes içinde değil de, AWS RDS gibi dışarıda yönetilen (managed) bir hizmet olarak kullanıyorsun. İçerideki uygulamalarının dışarıdaki `veritabanım.rds.amazonaws.com` adresine erişmesi gerekiyor. Bu Service sayesinde içerideki uygulaman sadece `db-service` ismini çağırır, Service onu otomatik olarak dışarıdaki AWS adresine yönlendirir.

Özetle Service; Kubernetes dünyasındaki ağ trafiğinin trafik polisidir. Hangi isteğin kime gideceğini, dışarıdan kimin içeri girebileceğini düzenler.&#x20;
