---
icon: hotdog
---

# Host Network Interfaces

Pod'lar dijital adalar gibidir; ana karaya (Host/Node) bağlanmak için bir köprüye ihtiyaç duyarlar. Bu köprünün teknik adı veth pair'dir.

**1. Sanal Kablolama**

CNI eklentisi, bir Pod oluşturduğunda aslında görünmez bir Ethernet kablosu çeker.

* İşleyiş: Bu kablonun iki ucu vardır.
  * Uç A (Pod Tarafı): Pod'un içine takılır ve genellikle `eth0` ismini alır.
  * Uç B (Host Tarafı): Düğümün (Node) ağ yığınına takılır. İsmi genellikle `veth...` veya `cali...` gibi karmaşık bir ID olur.
* Pod'dan çıkan bir veri paketi, bu sanal kabloyu (tüneli) geçerek Node'un ana ağına düşer.

**2. Adres Eşleşmesi**

Bir sistem yöneticisi, host üzerinde yüzlerce `veth` arayüzü görebilir. Hangisinin hangi Pod'a ait olduğunu anlamak için metindeki şu ipucu kritiktir:

* Pod'un içine girip (`ip netns exec`) arayüze bakıldığında `eth0@if19` gibi bir ifade görülür.
* Buradaki `@if19`, "Benim diğer ucum, Host üzerindeki 19 numaralı arayüzdür" demektir. Host üzerinde `ip addr` komutuyla 19 numaralı arayüzü bularak eşleşmeyi doğrulayabilirsiniz.

**3. CNI**

Bu karmaşık kablolama işlemini (veth oluşturma, uçları bağlama, IP atama) manuel yapmak imkansızdır.

* CNI eklentisi (Cilium, Calico vb.), Pod yaşam döngüsü boyunca bu arayüzleri oluşturur ve siler.
* Host üzerinde `cni-` veya `lxc-` ön ekiyle başlayan interface'ler ve namespace'ler, bu sürecin CNI tarafından yönetildiğinin kanıtıdır.

**4. NAT'sız İletişimin Temeli**

* Bu `veth` çifti sayesinde, Pod'un sahip olduğu IP (örn: `10.0.0.57`), Host üzerinden sanki fiziksel bir cihazmış gibi yönlendirilir. Host, paketi aldığında NAT yapmadan doğrudan diğer Node'a iletir.
