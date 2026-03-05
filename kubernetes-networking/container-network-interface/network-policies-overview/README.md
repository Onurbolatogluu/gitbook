---
icon: diagram-project
---

# Network Policies Overview

<figure><img src="../../../.gitbook/assets/Screenshot 2026-02-24 at 11.01.13.png" alt=""><figcaption></figcaption></figure>

En basit tabirle, cluster içindeki trafik ışıkları veya güvenlik görevlileridir. Hangi Pod'un nereye gidebileceğini veya hangi Pod'un kimden veri alabileceğini belirleyen kurallar bütünüdür. Bu kurallar ancak Calico veya Cilium gibi bir CNI kuruluysa çalışır; standart boş bir Kubernetes bunları anlayamaz.

#### 2. Kurallar Neye Göre Yazılır?

Bir kural yazarken hedefi 3 farklı şekilde seçebilirsin:

* podSelector: Etiketlere (label) göre çalışır. "Sadece `app: frontend` etiketine sahip Pod'lara izin ver" diyebilirsin.
* namespaceSelector: "Sadece `team: frontend` isimli Namespace içindeki Pod'lardan gelen trafiği kabul et" diyebilirsin.
* ipBlock: Daha fiziksel bir kuraldır. "172.17.0.0/16 IP bloğundan gelenlere izin ver" diyebilirsin.

#### 3. Trafiğin Yönü (Ingress ve Egress)

Kuralları yazarken trafiğin yönünü belirtmek zorundasın:

* Ingress (Geliş): Dışarıdan veya başka Pod'lardan, _benim_ Pod'uma doğru gelen trafiktir.
* Egress (Gidiş): _Benim_ Pod'umdan çıkıp dışarıya (veya başka Pod'a) giden trafiktir.

#### 4. Örnek Bir YAML Kod Bloğu İncelemesi

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: app1          # 1. Kimi koruyoruz? 'app1' etiketli Pod'ları.
  policyTypes:
  - Ingress              # 2. Hem gelen trafiği...
  - Egress               # 3. ...hem de giden trafiği kontrol edeceğiz.
  ingress:
  - from:                # 4. İÇERİ GİRİŞ İZNİ: Kimler gelebilir?
    - namespaceSelector:
        matchLabels:
          team: frontend # 'frontend' ekibinin odasından gelenler gelebilir.
    - podSelector:
        matchLabels:
          app: app2      # VEYA 'app2' etiketli podlar gelebilir.
  egress:
  - to:                  # 5. DIŞARI ÇIKIŞ İZNİ: Biz nereye gidebiliriz?
    - ipBlock:
        cidr: 10.0.0.0/24 # Sadece 10.0.0.0/24 IP aralığına gidebiliriz.
```

_(Not: Bu kural uygulandığında, `app1` Pod'u bu yazılanlar DIŞINDAKİ tüm trafiğe kapatılır - Default Deny)._

{% hint style="info" %}
Kısa bir ek bilgi: Eğer `podSelector`'ın başındaki tire (`-`) işaretini silseydin, o zaman VE (AND) olarak çalışırdı (Yani sadece frontend namespace'indeki app2 podlarına izin verirdi). Tire işareti olduğu için VEYAdır.
{% endhint %}

#### 5. Varsayılan (Default) Kurallar Atamak

Sistem yöneticilerinin en çok kullandığı yöntemdir. Bir ortama yeni bir Pod eklendiğinde güvenliği şansa bırakmamak için "Her Şeyi Yasakla" (Default Deny) kuralı yazılır.

```bash
# Bu kural 'default' namespace'indeki TÜM Pod'lara gelen tüm trafiği keser.
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {} # Süslü parantez içi boşsa, "Herkesi seç" demektir.
  policyTypes:
  - Ingress
```

#### 6. Standart Network Policy'nin Sınırları

Bu standart kuralların bazı yetersizlikleri vardır:

* Sadece IP ve Port seviyesinde (Layer 3 ve Layer 4) çalışır. "Sadece HTTP GET isteklerine izin ver" gibi uygulama seviyesinde (Layer 7) kurallar yazamazsın.
* Kuralların işleyip işlemediğini veya engellenen trafiği sana log olarak göstermez (Loglama özelliği yoktur).
* Doğrudan bir `Service` objesini hedef alamazsın, sadece etiket (label) seçebilirsin.

Peki bu sınırları nasıl aşıyoruz? İşte tam burada önceki konularımızda işlediğimiz Cilium veya Calico gibi gelişmiş CNI araçları devreye giriyor. Örneğin Cilium, standart kuralın yapamadığı _Layer 7 HTTP filtrelemeyi_ ve _Hubble ile loglamayı_ mümkün kılar.
