---
icon: message-medical
---

# Örnekler

#### 1. Senaryo Mimarisi ve Topoloji

Test ortamımız, Kubernetes ağının farklı iletişim katmanlarını simüle etmek için iki farklı Pod yapısı üzerine kurulmuştur. İlk olarak ortamdaki Pod'ların durumuna ve IP adreslerine bakalım.

Komut:

```bash
kubectl get pods -o wide
```

Çıktı:

```bash
NAME   READY   STATUS    RESTARTS   AGE   IP          NODE     NOMINATED NODE
pod1   2/2     Running   0          5m    10.0.0.57   node01   <none>
pod2   1/1     Running   0          5m    10.0.0.58   node01   <none>
```

_(Gördüğünüz gibi Pod'lar `node01` üzerinde çalışıyor ve CNI tarafından atanmış benzersiz IP'lere sahipler.)_

***

**A. Pod1 (Multi-Container Pod):**

Bu Pod, Kubernetes'in "Sidecar" mimarisine örnektir. İçinde `container1` (uyuyan istemci) ve `nginx` (web sunucusu) bulunur.

* Test Amacı: Aynı Pod içindeki konteynerlerin, dışarı çıkmadan localhost üzerinden haberleşebildiğini kanıtlamak.

Komut (Pod'un içinden test):

```bash
kubectl exec -it pod1 -c container1 -- curl localhost:80
```

Beklenen Çıktı:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

_(Sonuç: `container1`, yan odasındaki `nginx` servisine ağ kablosuna bile değmeden, sadece `localhost` diyerek erişti.)_

***

#### 2. L3 Bağlantısı

Bu adım, Kubernetes'in "Düz Ağ" modelini kanıtlar. Yani "Her Pod, diğer her Pod ile NAT olmadan konuşabilir."

* IP Tahsisi: CNI, Pod'lara Node'un IP bloğundan (`192.168.x.x`) tamamen bağımsız bir bloktan (`10.0.0.x`) IP vermiştir.

Komut (Pod1'den Pod2'ye Ping):

```bash
kubectl exec -it pod1 -- ping 10.0.0.58
```

Beklenen Çıktı:

```bash
PING 10.0.0.58 (10.0.0.58): 56 data bytes
64 bytes from 10.0.0.58: seq=0 ttl=64 time=0.086 ms
64 bytes from 10.0.0.58: seq=1 ttl=64 time=0.094 ms
```

_(Sonuç: İletişim doğrudan gerçekleşti. Arada NAT veya port yönlendirme olmadığı için IP adresleri değişmeden paketler yerine ulaştı.)_

***

#### 3. Altyapı ve CNI Entegrasyonu (L2 Bağlantısı)

Şimdi `node01` sunucusuna SSH ile bağlanıp kaputu açalım. Sanal ağın CNI tarafından nasıl inşa edildiğini göreceğiz.

**A. Sanal Ethernet (veth) Çiftleri**

CNI, Pod'ları ana makinenin ağına bağlamak için sanal kablolar (`veth`) çeker.

Komut (Node üzerinde):

```bash
ip addr show | grep veth
```

Beklenen Çıktı:

```bash
4: veth1a2b@if3: <BROADCAST,MULTICAST,UP> ... master cni0
6: veth8d9e@if3: <BROADCAST,MULTICAST,UP> ... master cni0
```

_(Buradaki her `veth...` satırı, çalışan bir Pod'un fiziksel dünyaya açılan kapısıdır. `@if3` ifadesi, kablonun diğer ucunun Pod'un içinde olduğunu gösterir.)_

**B. Pause Konteynerleri**

Her Pod için arka planda çalışan ve ağ alanını (namespace) ayakta tutan "Pause" sürecini görelim.

Komut (Node üzerinde):

```bash
ps aux | grep pause
```

Beklenen Çıktı:

```bash
root  3456  0.0  0.1 ... /pause
root  3490  0.0  0.1 ... /pause
```

_(Bu süreçler, Pod'ların "Tapu Sahibi"dir. Uygulamanız çökse bile bu süreç yaşadığı sürece Pod IP'si değişmez.)_

**C. Namespace İzolasyonu**

Her Pod'un kendine ait izole bir ağ ortamı (Network Namespace) olduğunu doğrulayalım.

Komut (Node üzerinde):

```bash
lsns -t net
```

Beklenen Çıktı:

```bash
NS         TYPE  NPROCS   PID USER   COMMAND
4026532247 net        2  3456 root   /pause
4026532248 net        1  4567 root   nginx
```

_(Bu çıktı, `nginx` ve `/pause` süreçlerinin ana makinenin ağında değil, kendilerine özel hazırlanmış sanal odalarda (Namespace) çalıştığını kanıtlar.)_

***

#### 🎯 Özet

1. Aynı Pod İçi: Konteynerler `curl localhost` ile konuşur.
2. Farklı Pod'lar: Pod'lar birbirine `ping 10.0.0.x` ile doğrudan (NAT'sız) ulaşır.
3. Altyapı: Tüm bu sihir, Node üzerindeki `veth` arayüzleri ve `/pause` konteynerleri sayesinde gerçekleşir.
