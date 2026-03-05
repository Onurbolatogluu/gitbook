---
icon: user-viewfinder
---

# Service Types

Kubernetes dünyasında uygulamalarımız (Pod'lar) sürekli olarak yaratılır, silinir ve her seferinde yeni bir IP adresi alırlar. Bu dinamik yapıda, ağ trafiğini doğru ve kesintisiz bir şekilde yönlendiren yapı taşı Service'dir.

Peki, içeride çalışan bir veritabanını dışarıya kapatırken, web sitemizin arayüzünü tüm dünyaya nasıl açarız? İşte tam bu noktada Service Types devreye girer. Bir servisin trafiği nasıl yöneteceğini belirlemek için YAML dosyasındaki `spec.type` alanını kullanırız.

Kubernetes'te ağ trafiğinin nasıl yönlendirileceğini belirleyen 4 temel servis tipi ve 1 özel durum vardır. Gelin bunları gerçek dünya senaryolarıyla inceleyelim.

### 1. ClusterIP (Varsayılan Tip: İç İletişim Uzmanı)

Bir _Service_ oluştururken herhangi bir tip belirtmezseniz, Kubernetes otomatik olarak ClusterIP tipini seçer. Bu bir şirketin iç hat telefonu gibi düşünebilirsiniz; dışarıdan bir müşteri bu numarayı arayamaz, sadece ofis içindeki çalışanlar birbirine ulaşabilir.

Sadece _Cluster_ içinden erişilebilen sabit bir IP adresi sağlar.

Ne Zaman Kullanılır?

Uygulamalarınızın internete açılmaması, tamamen güvenli bir kapalı kutuda kalması gerektiğinde kullanılır. Örneğin; veritabanları (PostgreSQL, MongoDB) veya sadece _Frontend_ üzerinden istek alan iç _Backend_ API'leri için en doğru seçenektir.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-backend
spec:
  type: ClusterIP  # Belirtilmese de varsayılan budur
  selector:
    app: backend
  ports:
    - port: 8080
      targetPort: 8080
```

### 2. NodePort (Dışarıya Açılan İlk Kapı)

Uygulamanızı dış dünyaya (veya şirket içi ağınıza) açmanın en temel yoludur. NodePort, _Cluster_ içindeki her bir Node'un (sunucunun) IP adresi üzerinde statik bir port açar. Varsayılan olarak bu port aralığı 30000 ile 32767 arasındadır.

İlginç bir mimarisi vardır: Uygulamanızın (Pod) hangi sunucuda çalıştığının hiçbir önemi yoktur. Kubernetes o portu tüm sunucularda açar. Siz 1. sunucunun IP'sine de gitseniz, 3. sunucunun IP'sine de gitseniz, arka plandaki trafik her zaman doğru Pod'u bulur (`<NodeIP>:<NodePort>`).

Ne Zaman Kullanılır?

Genellikle _development_ ve test ortamlarında, hızlıca dışarıdan erişim sağlamak istediğinizde kullanılır. Ancak port numaralarının takibi zor olduğu ve güvenlik riskleri barındırabildiği için _production_ ortamlarında doğrudan kullanılması tavsiye edilmez.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80          # Servisin iç portu, Sadece Kubernetes içindeki diğer uygulamaların (Pod'ların) bu servise ulaşmak için kullanacağı iç hat numarasıdır. Dışarıdan görülmez.
      targetPort: 80    # Pod'un içindeki uygulamanın portu
      nodePort: 31000   # Dışarıdan erişilecek port
```

### 3. LoadBalancer (Prod Ortamının Standardı)

Bu, Kubernetes'in dış dünyaya açılma konusundaki en profesyonel çözümüdür. `LoadBalancer` tipi, size dışarıdan erişilebilir statik bir IP adresi (External IP) verir ve gelen kullanıcı trafiğini içerideki sağlıklı Pod'lara eşit bir şekilde dağıtır. Ancak çalışma mantığı bulunduğunuz ortama göre değişiklik gösterir:

* Bulut Ortamlarında (Cloud): Eğer sisteminizi AWS, Google Cloud veya Azure gibi bir bulut sağlayıcı üzerinde çalıştırıyorsanız, Kubernetes doğrudan bulut API'si ile konuşur ve sizin için otomatik olarak fiziksel/sanal bir yük dengeleyici cihaz tahsis eder.
* Kendi Sunucularınızda (On-Premise / Bare-Metal): Saf Kubernetes mimarisi, bulut olmayan ortamlarda LoadBalancer oluşturamaz (IP adresi `pending` durumunda kalır). Kendi veri merkezinizde bu özelliği kullanabilmek için MetalLB veya Kube-VIP gibi ek araçlar kurmanız gerekir. Bu araçlar ağınızdaki boş IP'leri kullanarak bulut ortamını simüle eder ve size bir IP tahsis eder.

Ne Zaman Kullanılır? Production ortamlarında, yüksek erişilebilirlik gerektiren web sitelerinizi veya açık API'lerinizi internete açmanın standart ve en güvenli yoludur.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: public-frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
```

### 4. ExternalName (DNS Yönlendiricisi)

Diğer servis tiplerinden tamamen farklı çalışan, özel bir tiptir. Arkasında trafik yönlendireceği Pod'lar (selector) bulunmaz! Bunun yerine, Kubernetes içindeki uygulamalarınızı _Cluster dışındaki_ bir adrese bağlayan bir köprü (CNAME kaydı) görevi görür.

Ne Zaman Kullanılır?

Örneğin; veritabanınızı Kubernetes içinde değil de, dışarıda yönetilen bir hizmet olarak (Örn: AWS RDS üzerinde `db.example.com`) tutuyorsunuz. Kodunuzun içine bu uzun ve dış adresi yazmak yerine sadece `external-db` yazarsınız. İleride dış veritabanınızın adresi değişirse, kodunuza hiç dokunmadan sadece bu YAML dosyasını güncellersiniz.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.example.com
```

### 5. Headless Service

Normalde bir servisin amacı, arkadaki Pod'ları tek bir sanal IP arkasına gizleyip yük dağıtımı yapmaktır. Ancak YAML dosyasında `clusterIP: None` ayarını yaparsanız, servis bir Headless Service'e dönüşür.

Harika bir noktaya değindin. Kesinlikle haklısın; soyut tanımlar yerine doğrudan sistemin arkada nasıl davrandığını anlatmak, okuyucunun (ve senin) kafasındaki o "Neden buna ihtiyacım var?" sorusunu çok daha iyi çözecektir.

Blog yazındaki o maddeyi, az önce konuştuğumuz DNS (IP listesi) ve "Master/Slave" mantığıyla tamamen baştan, çok daha anlaşılır ve teknik olarak doyurucu bir şekilde revize ettim. Doğrudan sayfana ekleyebilirsin:

***

### 5. Headless Service (Lidersiz / Özel Durum)

Normalde Kubernetes'te bir servisin ana görevi, arkada çalışan Pod'ları tek bir sanal IP arkasına gizlemek ve gelen trafiği bu Pod'lar arasında rastgele (yük dengeleme ile) dağıtmaktır. Ancak YAML dosyasında `clusterIP: None` ayarını yaparsanız, bu kuralı tamamen yıkmış olursunuz ve servisiniz bir Headless Service'e dönüşür.

Standart bir servise bağlanmak istediğinizde, size tek bir IP adresi döner ve Kubernetes trafiği kime göndereceğine kendi karar verir (Kontrol Kubernetes'tedir).

Ancak bir Headless Service'e bağlandığınızda, Kubernetes aradan çekilir. Size tek bir IP adresi vermek yerine, arkada çalışan tüm Pod'ların gerçek IP adreslerini bir liste halinde doğrudan size sunar (Örneğin: `10.1.0.5`, `10.1.0.6`, `10.1.0.7`). Servis artık bir trafik yönlendirici değil, sadece bir "DNS Rehberi" olarak çalışır.

Ne Zaman Kullanılır?

Bu yapı, web siteleri için değil, Stateful uygulamalar (örneğin Redis, MongoDB, Kafka, Elasticsearch veritabanı cluster'ları) için hayati öneme sahiptir.

Çünkü bu veritabanlarında Pod'lar birbirinin aynısı değildir; aralarında "Master" ve "Slave" hiyerarşisi vardır. Yeni bir veri yazılacağı zaman bunun kesinlikle Master Pod'a gitmesi gerekir. Eğer normal servis kullanırsanız, Kubernetes isteği rastgele bir Slave Pod'a gönderebilir ve sistem "Ben sadece okuma yapabilirim" diyerek hata verir.

Headless Service kullandığınızda uygulamanızın kendi kodu (veritabanı sürücüsü) o IP listesini alır, içlerinden Master olanı kendisi tespit eder ve araya Kubernetes girmeden doğrudan hedefe bağlanır. Yani trafiğin kime gideceğini seçme özgürlüğü Kubernetes'ten alınıp, uygulamanızın kendi zekasına bırakılmış olur.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-app
spec:
  clusterIP: None      # Kubernetes'in rastgele trafik dağıtmasını engeller ve Pod IP'lerinin listesini döndürür.
  selector:
    app: stateful
  ports:
    - protocol: TCP
      port: 6379       # Örn: Redis varsayılan portu
      targetPort: 6379
```

***

### Özetle Servis Tipleri Karşılaştırması

* ClusterIP: Sadece içeriden ulaşılabilen çekirdek servisler (Core services).
* NodePort: Basit dış erişim ihtiyacı olan dev/test ortamları.
* LoadBalancer: Dış dünyaya açılan, production seviyesindeki uygulamalar.
* ExternalName: Dış bağımlılıkları (Harici DB veya SaaS API'leri) içeriye DNS ile dahil etme.
* Headless: Doğrudan Pod iletişimi gerektiren stateful uygulamalar.

