---
icon: rainbow-half
---

# Communication

Sistemde gereksiz bürokrasiyi kaldırmak performans getirir.

* Konteynerden-Konteynere: Aynı Pod içindeki Container'lar birbirleriyle konuşurken internete çıkmazlar. Tıpkı senin bilgisayarında çalışan iki programın birbiriyle konuşması gibi `localhost` kullanırlar. Bu, sıfır gecikme demektir.
* Poddan-Poda: Arada NAT yok. Pod A, Pod B'yi aradığında hat direkt bağlanır. Bu "Düz Ağ" (Flat Network) yapısı, işlemciye binen yükü azaltır.

**2. Büyüme Yeteneği**

Sistemi kurdun, harika çalışıyor. Peki ya kullanıcı sayısı 100 katına çıkarsa?

* IP-per-Pod Modeli: Kubernetes'te her Pod'un kendine ait IP'si vardır.
  * _Eski yöntem:_ Bir sunucuda 10 uygulama çalıştırırsan port çakışması olmasın diye uğraşırdın (Biri 80, biri 8080, biri 8081...).
  * _K8s yöntemi:_ Her Pod'un kendi IP'si olduğu için hepsi 80 portunu kullanabilir. Kimse kimsenin ayağına basmaz. Bu sayede sistemi binlerce Pod'a kadar büyütebilirsin.
* Pod'lar, Servisler ve Node'lar için IP havuzları baştan ayrılmıştır. Karmaşa çıkmaz.

**3. Kaos Yönetimi**

Pod'lar sürekli doğar ve ölür. Bu kaosu nasıl yönetiriz?

* Buradaki kilit kavram Soyutlama. Arka planda Pod'ların IP'leri sürekli değişse de, Service (ClusterIP) dediğimiz yapı sabit durur.
  * Bir şirketin müşteri hizmetleri numarasını (Service) ararsın. O gün telefonu hangi çalışanın (Pod) açtığı seni ilgilendirmez. Numara hep aynıdır.
* Dış Erişim: Dışarıdan gelen trafiği (External Traffic) düzenli bir şekilde içeri almak için NodePort veya LoadBalancer kapıları kullanılır.

**4. CNI Eklentileri**

Kubernetes, ağ işini tek bir yöntemle yapmaya zorlamaz. İhtiyaca göre "Taşeron" seçebilirsin.

* CNI (Container Network Interface): Kubernetes'in "Ağ işlerini yapacak usta" arayüzüdür.
* Kubelet (Node yöneticisi), yeni bir Pod geldiğinde CNI eklentisine (örn: Calico, Flannel) der ki: _"Buna kablo çek, IP ver."_
* Esneklik:
  * Güvenlik mi istiyorsun? -> Calico kullan (Network Policy desteği güçlü).
  * Hız mı istiyorsun? -> Cilium kullan (eBPF teknolojisiyle çok hızlı).
  * Basitlik mi istiyorsun? -> Flannel kullan.

***

Özetle: Kubernetes'te iletişim mimarisi; hız (NAT'sız yapı), büyüme (her Pod'a IP) ve sürdürülebilirlik (Service yapısı) üzerine kuruludur. CNI ise bu teoriyi pratiğe döken araçtır.
