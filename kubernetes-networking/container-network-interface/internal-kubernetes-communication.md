---
icon: computer-mouse-button-right
---

# Internal Kubernetes Communication

#### 1. Kubernetes Network Model (Altın Kurallar)

Kubernetes'in ağ yapısını kurarken koyduğu 3 tane değişmez, çok temel kural vardır:

1. Tüm Pod'lar birbirleriyle NAT olmadan konuşabilir: Yani bir Pod, cluster içindeki diğer bir Pod'a doğrudan kendi IP adresiyle ulaşabilir.&#x20;
2. Tüm Node'lar, tüm Pod'larla NAT olmadan konuşabilir: Node'lar da kendi içlerindeki veya diğer nodelardaki Pod'lara doğrudan erişebilir.
3. Bir Pod'un kendi gördüğü IP adresi, diğerlerinin onu gördüğü IP adresiyle aynıdır: Pod'un IP'si sabittir ve herkes o Pod'u aynı IP ile tanır.

Bu kurallar sayesinde Kubernetes ağı, sanki tek bir devasa switch'e bağlı dev bir ofis ağı gibi dümdüz (flat) bir yapıya sahip olur.

***

#### 2. Pod-to-Pod Communication (Aynı Node Üzerinde)

<figure><img src="../../.gitbook/assets/Screenshot 2026-02-19 at 11.51.07.png" alt=""><figcaption></figcaption></figure>

Diyelim ki elinde tek bir sunucu (Node) var ve içinde iki tane Pod çalışıyor (Örn: Biri _Frontend_, diğeri _Backend_). Bunlar nasıl konuşur?

* Her Pod'un kendi sanal ağ kartı vardır.
* Kubernetes, bu sanal ağ kartlarını Linux işletim sisteminin içindeki sanal bir köprüye (Bridge - genellikle `cbr0` veya `docker0` olarak adlandırılır) bağlar.
* Bu bağlantıyı veth pair (virtual ethernet pair) dediğimiz görünmez sanal kablolarla yapar.
* Frontend Pod'u, Backend Pod'una veri göndermek istediğinde, paket sanal kablodan (veth) çıkar, köprüye (bridge) gelir ve oradan diğer sanal kablo üzerinden Backend Pod'una ulaşır. Bu işlem inanılmaz hızlıdır çünkü fiziksel ağa hiç çıkmaz, sadece makinenin RAM/CPU'sunda gerçekleşir.

***

#### 3. Pod-to-Pod Communication (Farklı Node'lar Arasında)

<figure><img src="../../.gitbook/assets/Screenshot 2026-02-19 at 11.52.21.png" alt=""><figcaption></figcaption></figure>

Şimdi işi büyütelim. _Frontend_ Pod'umuz Node-1'de, _Backend_ Pod'umuz Node-2'de olsun. Bunlar nasıl konuşacak?

* İşte burada CNI (Container Network Interface) dediğimiz araçlar devreye girer (örneğin Calico, Cilium, Flannel vb. eklentiler).
* Paket Node-1'den çıktığında, fiziksel ağ (şirketinizin ağı veya AWS/Google Cloud ağı) Pod'ların kullandığı IP adreslerini tanımaz.
* CNI bu noktada paketi alır, fiziksel ağın anlayabileceği şekilde Encapsulation (paketleme - örn: VXLAN, IP-in-IP) yapar veya yönlendirme (routing) tablolarını günceller.
* Paket Node-2'ye ulaştığında paket açılır ve içindeki asıl veri Node-2'deki Backend Pod'una teslim edilir.

***

#### 4. Network Policies

Hatırlarsan ilk kuralımız "Herkes herkesle konuşabilir" idi. Ancak gerçek dünyada (Production ortamında) bu büyük bir güvenlik açığıdır! Veritabanı Pod'una herkesin ulaşmasını istemeyiz. İşte Network Policies, Pod'lar arasındaki bir nevi Güvenlik Duvarı gibi çalışır.

Kullanım Alanı (Use Case): Sadece _Frontend_ Pod'unun _Backend_ Pod'una gitmesine izin ver, diğer herkesi engelle.

_Örnek YAML:_

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend # Bu kural 'backend' etiketli podlara uygulanacak
  policyTypes:
  - Ingress # Gelen trafiği kontrol et
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend # Sadece 'frontend' etiketli podlardan GİRİŞE izin ver
    ports:
    - protocol: TCP
      port: 8080
```

_(Not: Ingress -> Gelen trafik, Egress -> Çıkan trafik demektir)._

***

#### 5. Services & DNS

Pod'lar geçicidir; ölebilirler, silinebilirler ve yeniden oluştuklarında IP adresleri değişir. Eğer Frontend, Backend'e IP adresi ile bağlanmaya çalışırsa ve Backend Pod'u ölürse sistem çöker. Bu yüzden Service kavramı vardır. Service, arkadaki Pod'ların IP'si değişse bile onlara ulaşmak için sabit bir IP adresi ve isim sağlar.

Ayrıca Kubernetes'in içinde otomatik bir DNS (Telefon rehberi - genelde CoreDNS) sistemi vardır. Frontend uygulamanız koda IP yazmak yerine doğrudan Service'in adını yazar: `http://my-backend-service.my-namespace.svc.cluster.local`

Terminalden bir Pod'un içine girip şu komutu yazarsan sana o servisin o anki güncel IP'sini söyler:

```bash
nslookup my-service.default.svc.cluster.local
```

***

#### 6. Service Mesh

<figure><img src="../../.gitbook/assets/Screenshot 2026-02-19 at 12.01.18.png" alt=""><figcaption></figcaption></figure>

Sisteminiz çok büyüdüğünde (örneğin 500 farklı mikroservisiniz olduğunda), temel Kubernetes Service'leri yetersiz kalır. "A trafiğinin %10'unu yeni sürüme gönder (Canary Deployment)", "Trafik şifreli (mTLS) aksın" veya "Bağlantı koparsa 3 kere daha otomatik tekrar dene (Retry)" gibi gelişmiş istekleriniz olur.

Bunun için Service Mesh (örneğin Istio, Linkerd) kullanılır. Çalışma mantığı Sidecar Proxy modeline dayanır. Sizin uygulama Pod'unuzun içine siz fark etmeden ikinci bir küçük konteyner (Proxy - genelde Envoy kullanılır) eklenir. Uygulamanız dışarıya veri göndereceği zaman trafiği önce bu Proxy alır, tüm şifreleme ve yönlendirme işlemlerini o yapar.
