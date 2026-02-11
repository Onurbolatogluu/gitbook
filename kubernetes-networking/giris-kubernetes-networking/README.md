# 💥 Giriş : Kubernetes Networking

Eskiden (sanal makineler veya Docker dünyasında), uygulamaların birbirini bulması için karmaşık port yönlendirmeleri yapardık. Kubernetes diyor ki: _"Bu çok karışık. Biz her şeyi düz (flat) bir ağa koyalım, herkes birbirine ismen veya doğrudan ulaşabilsin."_

**1. Temel Kural: Herkes Herkese Doğrudan Ulaşır**

Kubernetes'in altın kuralı şudur: NAT (Network Address Translation) yok. Bunu şöyle düşünebilirsin: Eskiden bir şirket santralini arayıp dahili numara tuşlardın (Docker port mapping). Kubernetes'te ise herkesin doğrudan bir cep telefonu numarası (IP Adresi) var.

* IP-per-Pod: Cluster içindeki her Pod, kendine ait benzersiz bir IP adresi alır.
* Node Erişimi: Her Node, üzerindeki tüm Pod'lara erişebilir. Kimse "Sen hangi porttan çıkıyorsun?" diye sormaz.

**2. Pod İçindeki Yaşam**

Bir Pod'u tek odalı bir ev gibi düşün. İçindeki Container'lar da ev arkadaşlarıdır.

* Tek Kapı Numarası: Evin (Pod'un) tek bir IP adresi vardır.
* Localhost: Ev arkadaşları (Container'lar) birbirleriyle konuşmak için telefona ihtiyaç duymazlar, aynı odadalar. Yani birbirlerine `localhost` üzerinden ulaşırlar.
* Port Paylaşımı: Aynı evde iki kişi aynı anda banyoyu kullanamaz. Benzer şekilde, bir Pod içindeki iki Container aynı portu (örn: 80) kullanamaz.

**3. İletişim Senaryoları**

Kubernetes'te trafik 4 ana yoldan akar:

1. Container-to-Container: (Aynı Pod içinde)
   * Aynı odadaki iki kişinin konuşması.
   * _Yöntem:_ `localhost` üzerinden.
2. Pod-to-Pod: (Farklı Podlar arası)
   * Yan komşuna gitmek.
   * _Yöntem:_ Doğrudan Pod IP'si ile. Arada NAT yok.
3. Pod-to-Service: (En Yaygın Senaryo)
   * Pod'lar ölümlüdür, ölüp dirildiklerinde IP'leri değişir. Bu yüzden onlara doğrudan IP ile gitmeyiz. Service dediğimiz sabit bir "Danışma" noktasına gideriz.
   * _Yöntem:_ ClusterIP üzerinden konuşulur.
4. External-to-Service:
   * İnternetteki bir kullanıcının sitene girmesi.
   * _Yöntem:_ NodePort veya LoadBalancer kullanılarak dışarıdaki trafik içeri, ilgili Service'e yönlendirilir.

**4. IP Yönetimi**

Herkesin IP'si farklı bir havuzdan gelir ki karışıklık çıkmasın:

* Pod'lar: CNI Plugin (aşağıda açıklayacağım) tarafından bir havuzdan (CIDR block) IP alır.
* Service'ler: Kubernetes'in beyni (kube-apiserver) tarafından sanal bir IP alır.
* Node'lar: Fiziksel ağdan veya Bulut sağlayıcısından (AWS, Azure vb.) IP alır.

**5. CNI (Container Network Interface) Nedir?**

Burası önemli. Kubernetes aslında "ağ kablosunu nasıl takacağını" bilmez. Sadece "Bir ağa ihtiyacım var" der. Bu işi yapan taşeron firmalara veya eklentilere CNI denir.

Kubelet (Node üzerindeki işçi), bir Pod yaratılacağı zaman CNI eklentisine seslenir: _"Hey, yeni bir Pod geliyor, buna bir IP ver ve ağa bağla."_

Popüler CNI Eklentileri:

* Calico: Güvenlikçidir. Kimin kiminle konuşabileceğini (Network Policy) çok iyi yönetir. Çok popülerdir.
* Flannel: Basittir. "Sadece ağ çalışsın, detaya girmeyelim" diyorsan bunu kullanırsın (Overlay network).
* Cilium: Yeni nesildir. Çok hızlıdır ve Linux çekirdeğindeki eBPF teknolojisini kullanır.&#x20;

***

Özetle: Kubernetes'te her Pod'un bir IP'si vardır ve NAT yapmadan diğerleriyle konuşabilir. Bu, eski usül sistemlere göre hayatı çok kolaylaştırır.
