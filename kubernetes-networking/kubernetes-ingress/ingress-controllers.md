---
icon: arrow-rotate-left
---

# Ingress Controllers

Kubernetes dünyasına yeni adım atanların en çok karıştırdığı iki kavram vardır: Ingress Resource ve Ingress Controller. Bu ikisi arasındaki farkı anlamadan, trafiğinizi güvenli ve performanslı bir şekilde yönetmeniz imkansızdır.

* Ingress Resource (Kurallar Dosyası): Sizin yazdığınız o standart YAML dosyasıdır. "Eğer `bolatogluyapi.com/api` adresine istek gelirse, trafiği `backend-service`'e gönder" dediğiniz kağıt parçasıdır. Kendi başına hiçbir işe yaramaz; trafiği yönlendirecek kas gücüne sahip değildir.
* Ingress Controller (Motor): O kağıt parçasını 7/24 okuyan ve içeri giren trafiği fiziksel olarak ilgili servislere yönlendiren gerçek çalışan uygulamadır (Pod'lardır). Eğer sisteminizde aktif bir Controller yoksa, yazdığınız o harika Ingress kuralları sonsuza dek havada asılı kalır ve dışarıdan gelen hiçbir istek hedefine ulaşamaz.

Peki sistemimize bir motor takacağımız zaman önümüzde hangi seçenekler var?

### 1. Cloud-Based

Eğer sisteminiz AWS, Google Cloud (GCP) veya Azure gibi bir bulut altyapısında çalışıyorsa, bu devlerin size sunduğu özel motorları kullanabilirsiniz.

* AWS Load Balancer Controller: AWS ortamında çalışır, otomatik olarak Application Load Balancer (ALB) oluşturur.
* Google Cloud Load Balancer: GKE (Google Kubernetes Engine) ile mükemmel entegre çalışır.

Avantajı: Kurulumu ve yönetimi çok kolaydır. Bulut sağlayıcının kendi güvenlik duvarı (WAF) ve yetkilendirme (IAM) sistemleriyle doğrudan konuşur.

### 2. Self-Managed

Bulut sağlayıcısına bağımlı kalmak istemiyorsanız veya kendi fiziksel sunucularınızda (On-Premise) çalışıyorsanız, Kubernetes ekosisteminin bağımsız şampiyonlarını kullanmanız gerekir.

* NGINX Ingress Controller: Sektörün tartışmasız en çok kullanılan, en çok dökümana sahip olan standart motorudur. reverse proxy, SSL/TLS ve rate-limiting gibi her türlü ihtiyacı karşılar.
* Traefik: Dinamik yapılandırma konusunda harikadır, kendi paneli (dashboard) vardır.
* HAProxy Ingress: Performans ve düşük gecikme (low-latency) odaklı, saniyede milyonlarca isteği karşılayabilen bir canavardır.

***

### En Kritik Mimari Karar: Motoru Nasıl Çalıştıracağız?

Diyelim ki NGINX Ingress Controller kurmaya karar verdiniz. Önünüze çok kritik bir altyapı sorusu çıkacaktır: _"Bu motoru sunucularıma Deployment olarak mı kurayım, yoksa DaemonSet olarak mı?"_

Bu soru, sisteminizin performansını doğrudan etkiler:

#### Seçenek A: Deployment Modeli

Bu modelde, "Bana 3 tane NGINX Pod'u ver" dersiniz. Kubernetes bu 3 Pod'u, müsait bulduğu sunuculara (Node) rastgele dağıtır.

* Artıları: Gelen trafik arttığında Pod sayısını otomatik olarak (HPA ile) 3'ten 10'a çıkarabilirsiniz. Sunucu kaynaklarını (CPU/RAM) çok daha verimli kullanır.
* Eksileri: Yük dağılımı dengesiz olabilir. Bazen 3 Ingress Pod'unun 2'si aynı sunucuya (Node) denk gelebilir.

#### Seçenek B: DaemonSet Modeli

Bu modelde sayı vermezsiniz. Kubernetes'e şu emri verirsiniz: _"Cluster'ımdaki HER BİR SUNUCUYA (Node) kesinlikle 1 adet NGINX Ingress Pod'u kuracaksın!"_ Sisteminizde 50 sunucu varsa, 50 tane motorunuz olur. Yeni bir sunucu eklendiği an, otomatik olarak 51. motor da kurulur.

* Artıları: Gelen trafik sistemdeki tüm sunuculara kusursuz bir şekilde, eşit dağılır. Bir sunucu çökse bile diğer tüm sunucularda hazır bekleyen motorlar olduğu için yüksek erişilebilirlik (High Availability) kusursuzdur.
* Eksileri: Her sunucuda sürekli bir NGINX çalışacağı için, trafik az olsa bile boş yere CPU ve RAM tüketir (Kaynak israfı).

Özetle Hangisini Seçmelisiniz?

Eğer trafiğiniz çok dalgalıysa ve sunucu kaynaklarınızı idareli kullanmak istiyorsanız Deployment; "Benim için hız, eşit dağılım ve kesintisizlik kaynaktan çok daha önemli" diyorsanız DaemonSet modelini tercih etmelisiniz.

