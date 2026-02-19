---
icon: circle-microphone-lines
---

# Pod to Pod Communication

Kubernetes ortamında ağ yapısının en temel kuralı şudur: Her Pod, Cluster içinde kendine ait benzersiz bir IP adresine sahip olur. Bu mimari sayesinde Pod'lar, aynı Node (sunucu) veya farklı Node'lar üzerinde olsalar bile hiçbir NAT işlemine gerek kalmadan birbirleriyle doğrudan iletişim kurabilirler.

Bu iletişimi arka planda yöneten sisteme CNI (Container Network Interface) denir.

### 1. CNI (Cilium) Durumunu Doğrulama

Pod'ların iletişim kurabilmesi için öncelikle CNI eklentisinin sağlıklı çalıştığından emin olmak gerekir.

```bash
# Cilium bileşenlerinin durumunu kontrol etmek için:
cilium status
```

_Not: Çıktıda `Cilium: OK` ve `Operator: OK` görülmelidir._

### 2. Pod Dağıtımı ve Ağ Arayüzlerinin (Interfaces) Oluşumu

Pod'lar oluşturulduğunda CNI eklentisi devreye girerek onlara IP atar ve sanal ağ kabloları (veth pair) oluşturur. Bu kablonun bir ucu Node üzerinde, diğer ucu ise Pod'un yalıtılmış kendi network alanında (`Network Namespace`) bulunur.

```bash
# Örnek bir YAML dosyasıyla Pod'ları ayağa kaldırma
kubectl apply -f pods.yaml

# CNI'ın (Cilium) arka plan da endpointleri nasıl oluşturduğunu izlemek için Node üzerinde log takibi:
journalctl -u cilium -f
```

#### Pod İçindeki Ağ Arayüzünü İnceleme

Pod'a atanan `veth`  interface'lerinin Node üzerinde nasıl göründüğünü kontrol edebiliriz:

```bash
# Node üzerindeki veth arayüzlerini listeleme
ip addr | grep -A1 lxc

# Pod'un kendi Network Namespace'ine girip arayüzü (eth0) kontrol etme
ip netns exec <network-namespace-id> ip addr show eth0
```

_Bu işlem sonucunda Pod'un aldığı IP adresini (`Örn: 10.0.1.207`) görebiliriz._

### 3. Pod'ların Geçici (Ephemeral) Doğası ve IP Değişimi

Kubernetes'te Pod'lar kalıcı değildir. Bir Pod silinip yeniden oluşturulduğunda eski IP adresini korumaz, CNI havuzundan yeni bir IP alır.

```bash
# Pod'u silme işlemi (Eski IP ve endpoint silinir)
kubectl delete pod pod1

# Pod'u yeniden oluşturma (Yeni bir IP atanır)
kubectl apply -f pods.yaml
```

### 4. IP Üzerinden İletişim Testi

Farklı IP adresleri alan Pod'ların birbiriyle konuşabildiğini doğrulamak için `ping` veya `curl` komutları kullanılır.

```bash
# Tüm Pod'ların IP adreslerini hızlıca listeleme
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}: {.status.podIP}{"\n"}{end}'

# pod1 içinden, aynı Node'daki pod2'ye (10.0.1.14) ping atma
kubectl exec -it pod1 -- ping -c 4 10.0.1.14

# pod1 içinden, farklı bir Node'daki pod3'e (10.0.0.245) ping atma
kubectl exec -it pod1 -- ping -c 4 10.0.0.245

# pod1 içinden, pod3 üzerindeki 80 portuna (HTTP) istek atma
kubectl exec -it pod1 -- curl -vvv 10.0.0.245:80
```

### 5. DNS Üzerinden İletişim Testi (A Records)

Cilium gibi modern CNI'lar, oluşturulan her Pod IP'si için otomatik olarak bir DNS A kaydı oluşturur. Bu sayede IP adresi yerine DNS ismiyle de erişim sağlanabilir.

Kayıt Formatı: `<ip-cizgili-format>.<namespace>.pod.cluster.local`

```bash
# pod1 içinden, pod3'e DNS ismiyle ping atma (10.0.0.245 IP'si için)
kubectl exec -it pod1 -- ping -c 4 10-0-0-245.default.pod.cluster.local
```

> ÖNEMLİ UYARI: Pod IP'leri sürekli değişebileceği için (Pod silinmesi/çökmesi durumunda), bu DNS formatı production ortamlarında kullanılmamalıdır. Pod'lar arası kalıcı ve güvenilir iletişim için her zaman IP'leri sabitleyen ve yük dağıtımı yapan Kubernetes Service objeleri tercih edilmelidir.

***

### Use Cases

1.  Microservices Haberleşmesi:

    Bir e-ticaret uygulamasında "Ödeme (Payment)" Pod'unun, "Kullanıcı (User)" Pod'una API isteği atarak kullanıcının bakiye bilgisini doğrulaması işlemi bu iletişim ağı üzerinden gerçekleşir.
2.  Log ve Metrik Toplama:

    Cluster içinde çalışan bir `Prometheus` (izleme aracı) Pod'unun, diğer tüm uygulama Pod'larına erişip onların anlık sağlık durumlarını (metriklerini) çekmesi.
3.  Veritabanı İşlemleri:

    Backend uygulaması olarak çalışan bir Pod'un, verileri yazmak için Redis veya PostgreSQL veritabanı Pod'uyla doğrudan iletişim kurması.

***

### Cheat Sheet

| **Ne İçin Kullanılır?**       | **Komut**                                           |
| ----------------------------- | --------------------------------------------------- |
| CNI Durum Kontrolü            | `cilium status`                                     |
| Pod'ların IP'lerini Listeleme | `kubectl get pods -o wide` _(veya jsonpath)_        |
| Pod İçinde Komut Çalıştırma   | `kubectl exec -it <pod-adi> -- <komut>`             |
| Farklı Pod'a Ping Atma        | `kubectl exec -it pod1 -- ping -c 4 <hedef-pod-IP>` |
| Pod Loglarını İzleme          | `kubectl logs <pod-adi> -f`                         |
