---
icon: calendar-clock
---

# Cilium

#### 1. En Temelden Başlayalım: CNI (Container Network Interface) Nedir?

Kubernetes'in kendisi network işlerinden anlamaz. "Şu Pod bu Pod'a nasıl bağlanacak?", "IP adreslerini kim dağıtacak?" gibi soruları Kubernetes cevaplamaz. Bu işi yapması için sisteme dışarıdan bir plugin kurman gerekir.

İşte bu network pluginlerine CNI denir. Cilium da günümüzde piyasadaki en güçlü, en yetenekli CNI çözümlerinden biridir.

#### 2. Cilium Neden Bu Kadar Özel? (Sırrı: eBPF)

Eski nesil ağ araçları, trafiği yönetmek için standart Linux araçlarını (örneğin _iptables_ gibi güvenlik duvarı kurallarını) kullanır. Ancak cluster büyüdükçe, on binlerce _iptables_ kuralı sistemi inanılmaz derecede yavaşlatır.

Cilium ise oyunun kurallarını eBPF (Extended Berkeley Packet Filter) adlı bir Linux teknolojisiyle değiştirir.

* eBPF Nedir? İşletim sisteminin en derin kalbine (yani Kernel seviyesine) doğrudan müdahale edebilmeyi sağlayan bir yapıdır. Normalde Linux _kernel_ kodunu değiştirmek veya yeniden başlatmak çok tehlikeli ve zordur. Ancak eBPF, işletim sistemini yeniden başlatmadan _kernel_ seviyesine küçük, inanılmaz hızlı ve güvenli kod parçacıkları enjekte etmeni sağlar.
* Sonuç: Ağ trafiği, güvenlik kontrolleri ve _load balancing_ işlemleri uygulamaya ulaşmadan, doğrudan işletim sisteminin çekirdeğinde hızlı bir şekilde çözülür. Gerektiğinde geleneksel _kube-proxy_ bileşenine bile ihtiyaç bırakmaz.

#### 3. Cilium'un Ana Parçaları

* Cilium Agent: Cluster'daki her bir sunucunun (_Node_) üzerinde çalışan işçidir. O sunucudaki eBPF programlarını yönetir, ağ kurallarını kernele yazar.
* Hubble (Observability & Security): Cilium'un en sevilen özelliklerinden biridir. Hubble, ağ trafiğinin röntgenini çeker. Terminalden veya görsel bir arayüzden _"Hangi Pod, hangi Pod ile konuşuyor?", "Bağlantı nerede kopuyor?", "Kim kime ne verisi gönderiyor?"_ gibi karmaşık ağ haritalarını canlı olarak görmeni sağlar.
* Advanced Network Policies: Kötü niyetli trafiği engellemek için yazılan kurallardır.

#### 4. Use Case: Layer 3/4 vs. Layer 7 Farkı

Burası Cilium'u diğer araçlardan ayıran en büyük farktır.

Standart Kubernetes ağ kuralları sadece Layer 3 ve Layer 4 (IP adresi ve Port) seviyesinde çalışır. _Yani normalde sadece şunu diyebilirsin:_ "Frontend Pod'u, Backend Pod'unun 80 portuna erişebilsin."

Ama Cilium, Layer 7 (Uygulama Katmanı) seviyesinde çalışabilir. Uygulamanın konuştuğu API'yi, HTTP metodlarını, hatta Kafka mesajlarını bile anlar.

Örnek Senaryo: Bir `frontend` uygulaman ve bir `backend` API'n var. `frontend`'in sadece veri okumasını (HTTP GET) istiyorsun. Yanlışlıkla veya hacklenerek veri silmesini (HTTP DELETE) kesinlikle engellemek istiyorsun.

Cilium ile yazacağın (ve _CiliumNetworkPolicy_ adı verilen) YAML dosyası şöyle görünür:

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "frontend-to-backend-l7-policy"
spec:
  endpointSelector:
    matchLabels:
      app: backend          # Kuralın uygulanacağı hedef Pod'lar
  ingress:
  - fromEndpoints:
    - matchLabels:
      app: frontend         # İsteğin geldiği kaynak Pod'lar
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "GET"               # SADECE GET isteklerine izin ver!
          path: "/api/v1/users"       # SADECE bu API adresine gitsin!
```

Bu YAML Ne Yapar? Bu dosyayı sisteme uyguladığında, `frontend` Pod'u `backend`'e bir `GET /api/v1/users` isteği atarsa Cilium bunu Kernel seviyesinde anında geçirir. Ama `frontend` gidip `DELETE /api/v1/users` (kullanıcı silme) isteği atarsa, IP ve Port doğru olsa bile Cilium (Layer 7 özelliğinden dolayı) bunu anında engeller (_Drop_ eder).

#### Özetle Cilium Neden Var?

1. Networking: Pod'ları birbirine inanılmaz hızlı (eBPF ile) bağlamak için.
2. Security: Sadece IP/Port değil, HTTP/API seviyesinde (Layer 7) güvenlik kuralları yazabilmek için.
3. Observability: Hubble aracı sayesinde cluster'ın içinde dönen görünmez trafiği anlık izleyebilmek için.
