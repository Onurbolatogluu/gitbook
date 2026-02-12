---
icon: globe
---

# Kubernetes Ağ Modeli

Kubernetes, sıfırdan yeni bir ağ teknolojisi icat etmemiştir. Bunun yerine, Linux işletim sisteminin var olan güçlü ağ özelliklerini (Namespaces, Veth Pairs) kullanarak bir orkestrasyon sağlar.

**1. Pod İçi Mimari: "Pause" Konteyneri ve Paylaşılan Namespace**

Bir Pod'un içinde birden fazla konteyner (örneğin ana uygulama ve bir log toplayıcı) çalışabilir.&#x20;

* Başlangıç: Pod oluşturulduğunda, Kubernetes önce "Pause" (veya Infra) adında çok küçük, görünmez bir konteyner başlatır.
* Görevi: Bu konteynerin tek işi, bir IP adresi almak ve Ağ Alanını (Network Namespace) açık tutmaktır.
* Asıl işi yapan konteynerler (örneğin NGINX), bu "Pause" konteynerinin hazırladığı ağ alanına dahil olur.
* Sonuç : Eğer ana uygulamanız hata verip çökerse (Crash), Kubernetes sadece o uygulamayı yeniden başlatır. "Pause" konteyneri arka planda hiç kapanmadığı için IP adresi değişmez, uygulama kaldığı yerden devam eder. _(Not: Eğer Pod komple silinir ve yeni bir Pod oluşturulursa, o zaman yeni bir Pause konteyneri ve yeni bir IP gelir.)_

**2. Pod-to-Pod İletişim**

Daha önce bahsettiğimiz "IP-per-Pod" modelinin teknik karşılığıdır.

* Her Pod'un kendi IP'si vardır.
* Kubernetes, Node'lar üzerindeki yönlendirme tablolarını öyle bir ayarlar ki, trafik bir Node'dan diğerine giderken paketin içindeki Pod IP'si değiştirilmez (NAT uygulanmaz).

**3. CNI ve Sanal Kablolama (Veth Pairs)**

* Bir Pod oluşturulduğunda, CNI eklentisi sanal bir kablo çeker. Bu kablonun iki ucu vardır:
  1. Uç A (Pod tarafı): Pod'un içine `eth0` olarak takılır.
  2. Uç B (Node tarafı): Host makineye (Node) takılır ve genellikle `veth...` veya `cali...` gibi rastgele bir isim alır.
* Bu sanal kablo sayesinde Pod içindeki trafik, Node'un ana ağ hattına aktarılır.

**4. Node Seviyesinde İnceleme Araçları**

Bir sistem yöneticisi olarak, "Kubernetes bozulursa" Linux komutlarıyla durumu analiz etmeniz gerekir.&#x20;

**A. `ip addr` ile Sanal Kabloları Görmek**

Node üzerindeki tüm ağ arayüzlerini listeler. Kubernetes ortamında yüzlerce `veth` (sanal ethernet) arayüzü görmek normaldir.

* Senaryo: Node üzerinde o an kaç tane Pod'un ağa bağlı olduğunu hızlıca anlamak istiyorum.
* Komut:

```bash
ip addr show | grep veth
```

* Örnek Çıktı:

```bash
4: veth1a2b3c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
6: veth8d9e0f@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
```

Analiz: Buradaki her bir `veth...` satırı, aslında çalışan bir Pod'un dış dünyaya açılan kapısıdır. `@if3` ifadesi, bu kablonun diğer ucunun (Pod'un içindeki `eth0`'ın) indeks numarasını gösterir.

**B. `lsns` ile İzole Alanları Listelemek**

Linux üzerindeki namespace'leri listeler. Hangi işlemin (Process) hangi ağ alanını kullandığını bulmak için kritiktir.

* Senaryo: Sistemde çalışan gizli ağ alanlarını (Network Namespaces) ve bunları kullanan süreçleri (Process ID - PID) bulmak istiyorum.
* Komut:

```bash
lsns -t net
```

* Örnek Çıktı:

```bash
NS         TYPE  NPROCS   PID USER   COMMAND
4026531992 net      126     1 root   /usr/lib/systemd/systemd
4026532247 net        1  3456 root   /pause
4026532248 net        1  4567 root   nginx
```

* Analiz: Listede `/pause` komutunu görüyorsanız, bu bir Kubernetes Pod'unun ağ alanıdır. `PID` sütunundaki numara (örneğin 3456), o Pod'un ağ alanına girmek için kullanacağınız anahtardır.

**Gemini said**

Harika bir fikir. Dokümantasyonuna "teori" kısmını yazmışsın, şimdi "pratik" (hands-on) kısmını ekleyerek okunabilirliği ve faydayı artıralım.

İşte dokümanına ekleyebileceğin, senaryo bazlı gerçek hayat örnekleri:

***

#### 4. Node Seviyesinde İnceleme Araçları (Pratik Örnekler)

Bir sistem yöneticisi olarak, "Kubernetes bozulursa" Linux komutlarıyla durumu analiz etmeniz gerekir. İşte sık kullanılan senaryolar:

**A. `ip addr` ile Sanal Kabloları Görmek**

Node üzerindeki tüm ağ arayüzlerini listeler. Kubernetes ortamında yüzlerce `veth` (sanal ethernet) arayüzü görmek normaldir.

* Senaryo: Node üzerinde o an kaç tane Pod'un ağa bağlı olduğunu hızlıca anlamak istiyorum.
* Komut:

```bash
ip addr show | grep veth
```

* Örnek Çıktı:

```bash
4: veth1a2b3c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
6: veth8d9e0f@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
```

* Analiz: Buradaki her bir `veth...` satırı, aslında çalışan bir Pod'un dış dünyaya açılan kapısıdır. `@if3` ifadesi, bu kablonun diğer ucunun (Pod'un içindeki `eth0`'ın) indeks numarasını gösterir.

***

**B. `lsns` ile İzole Alanları Listelemek**

Linux üzerindeki namespace'leri listeler. Hangi işlemin (Process) hangi ağ alanını kullandığını bulmak için kritiktir.

* Senaryo: Sistemde çalışan gizli ağ alanlarını (Network Namespaces) ve bunları kullanan süreçleri (Process ID - PID) bulmak istiyorum.
* Komut:

```shellscript
lsns -t net
```

* Örnek Çıktı:

```shellscript
NS         TYPE  NPROCS   PID USER   COMMAND
4026531992 net      126     1 root   /usr/lib/systemd/systemd
4026532247 net        1  3456 root   /pause
4026532248 net        1  4567 root   nginx
```

* Analiz: Listede `/pause` komutunu görüyorsanız, bu bir Kubernetes Pod'unun ağ alanıdır. `PID` sütunundaki numara (örneğin 3456), o Pod'un ağ alanına girmek için kullanacağınız anahtardır.

**C. `nsenter` ile Pod'un İçine Sızmak**

`lsns` ile bulduğumuz PID'yi kullanarak, sanki o Pod'un içindeymişiz gibi komut çalıştırmamızı sağlar. `kubectl exec` çalışmadığında hayat kurtarır.

* Senaryo: Bir Pod'un IP'si var ama erişilemiyor. `kubectl` komutları da yanıt vermiyor. Doğrudan Node üzerinden o Pod'un gözünden ağa bakmak istiyorum.
* Komut: (Önce `lsns` ile PID bulunur, örneğin: 3456)

```bash
nsenter -t 3456 -n ip addr
```

* Bu komut, Node üzerindeki 3456 ID'li sürecin (Pod'un) ağ namespace'ine girer ve orada `ip addr` komutunu çalıştırır.
* Node'un kendi IP'lerini değil, sadece o Pod'un `eth0` IP adresini (örn: 10.244.1.5) görürsünüz.

{% hint style="info" %}
Modern Kubernetes sistemlerinde (Containerd/CRI-O kullanan), `ip netns list` komutu bazen boş dönebilir. Çünkü bu araçlar namespace dosyalarını varsayılan Linux klasöründe (`/var/run/netns`) tutmazlar. Bu yüzden `lsns` ve `nsenter` ikilisini kullanmak, Kubernetes dünyasında daha güvenilir bir yöntemdir.
{% endhint %}

***

Özet: Pod dediğimiz şey, aslında ortak bir Network Namespace paylaşan ve Node'a 'veth' kablosuyla bağlı olan konteynerler grubudur. Bu yapıyı ayakta tutan gizli kahraman ise Pause Container'dır.
