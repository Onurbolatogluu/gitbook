# 🍒 Bileşenler

Kubernetes ortamını oluşturan bir çok servis mevcuttur.

![](../.gitbook/assets/1\_QWJijlj7kwd0hIYk8Wsnow.png)

### Kube-ApiServer

Kubernetes control plane kısmının, en önemli bileşeni ve giriş noktasıdır. Tüm diğer komponent ve node bileşenlerin direkt iletişim kurduğu tek komponent.&#x20;

* Tüm iletişim api üzerinden gerçekleşir.
* Kullanıcılar kubectl komut satırı client 'ı veya rest api çağrıları arayıcılığıyla api server ile bağlantı kurabilirler.
* Biz kubernetes ile bağlantı kurmak için, api server 'a erişir, authentication işlemlerini tamamlar ve istekleri iletiriz.&#x20;
* Tüm komponentler iletişimi api server arayıcılığıyla gerçekleştirir.

### ETCd

Tüm cluster bilgisi, metadata bilgileri ve kubernetes de oluşturulan tüm objelerin bilgilerinin tutulduğu, key-value veri deposudur.

* Kubernetes çalışması için, gerekli tüm veriler burada tutulur.
* API hariç, diğer komponentler etcd ile direkt haberleşemezler. ETCd ile iletişim kurmaları gerektiğinde, kube-apiserver arayıcılığıyla yaparlar.

### Scheduler

Yeni oluşturulan, veya bir node ataması yapılmamış pod'ları izler ve üzerinde çalışacakları bir node seçer. (POD : container olarak düşünebiliriz) Pod 'un kaynak isteğini, özel gereksinimi, çeşitli parametreleri göz önünde bulundurur. Ve seçme algoritması sayesinde, pod için en uygun node hangisi olduğuna karar verir.

### Controller Manager

Mantıksal olarak, her controller ayrı bir süreçtir. Ancak karmaşıklığı azaltmak için hepsi tek bir binary olarak derlenmiştir. Ve tek bir proccess olarak çalışır. Bu controller'den bazıları şunlardır,

* Node controller
* Job controller
* Service account-Token controller
* Endpoints controller

Controller manager bileşeni, istenilen durum ile mevcut durumu gözlemlerler. API arayıcılığıyla Etcd 'de saklanan, cluster durumunu izler ve eğer mevcut durum ile istenilen durum arasında fark varsa, bu durumu oluşturan kaynakları gerektiği gibi oluşturur, günceller veya silerek bu durumu eşitler.



Control Plane dediğimiz bu kısmı master node dediğimiz sistemler üzerinde çalıştırırız. Tüm bu komponentler tek bir Linux yüklü sisteme kurulup, oradan erişilebildiği gibi, yüksek erişilebilirlik sağlaması adına birden fazla sisteme kurulabilir.



### Worker Node

Container runtime, container'dan sorumlu olan kısımdır. Kubernetes bir kaç container runtime destekler, docker-containerd-CRI-O. Worker node cluster 'a dahildir. Varsayılan olarak container engine containerd 'dir.

### Kubelet

Cluster 'da çalışan her node bir agent'dir. Pod içerisinde tanımlanan container'ların, çalıştırılmasını sağlar. Kubelet çeşitli mekanizmalar arayıclığıyla sağlanan bir dizi pod tanımı alır ve bu pod tanımda belirtilen container 'ların, çalışır durumda ve sağlıklı olmasını sağlar.

* Her worker node üzerinde bulunur.
* API server arayıcılığıyla etcd 'yi kontrol eder ve scheduler tarafından bulunduğu node üzerinde çalışması gereken pod belirtildiyse, kubelet bu pod'u bulunduğu node üzerinde yaratır.&#x20;
* ContainerD 'ye haber gönderir ve belirtilen özelliklerde container'ın, bulunduğu node üzerinde çalıştırılmasını sağlar.

### Kube-Proxy

Node'lar üzerinde ağ kurallarını yönetir. Ve bu ağ kuralları cluster 'ın içindeki veya dışındaki ağ oturumları tarafından pod'larımızla ağ iletişimine izin verir.

