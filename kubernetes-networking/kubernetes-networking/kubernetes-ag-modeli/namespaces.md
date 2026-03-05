---
icon: input-text
---

# Namespaces

Teknik olarak bir Pod, aynı Network Namespace'i (Ağ Ad Alanını) paylaşan konteynerler grubudur.

* İşleyiş: Pod içindeki ana uygulama (örneğin Java) ve yan uygulamalar (örneğin Log agent), Linux çekirdeğinde aynı ağ namespace'i kullanır.
* Pod içindeki herhangi bir konteynerde `ip addr` çalıştırıldığında aynı IP ve MAC adresi görülür. Bu nedenle `localhost` iletişimi mümkündür.

**2. "Pause" Konteyneri**

* Görevi: Bu konteynerin tek işi, bir Network Namespace oluşturmak ve onu "açık tutmaktır".
* Neden Gerekli? Eğer ana uygulamanız (örneğin Nginx) çökerse, Namespace'in de yok olmaması gerekir. "Pause" konteyneri, Pod yeniden başlatılana kadar ağ kimliğini (IP adresini) korur.
* Node üzerinde `lsns -t pid` komutuyla bakıldığında, network namespaceleri tutan süreçlerin aslında bu "Pause" konteynerleri olduğu görülür.

**3. Veth Çiftleri**

İzole edilmiş bir Namespace'in (Pod) dış dünyayla konuşabilmesi için bir kabloya ihtiyacı vardır.

* Sanal Kablolama: CNI eklentisi, Veth Pair (Sanal Ethernet Çifti) oluşturur.
  * Bir uç Pod'un Namespace'ine takılır (`eth0`).
  * Diğer uç Host'un (Node) Namespace'ine takılır (`veth...`).
* Pod'dan çıkan veri, bu sanal tünelden geçerek Node'un ana ağına ulaşır ve oradan yönlendirilir.

**4. Sorun Giderme Araçları**

1. `lsns`: Sistemdeki tüm izole alanları listeler. Hangi Pod'un hangi ID'ye sahip olduğunu bulmak için kullanılır.
2. `ip netns identify <PID>`: Bir sürecin (Process) hangi CNI ağına ait olduğunu doğrular. İsimlerin `cni-` ile başlaması, yönetimin CNI eklentisinde olduğunu gösterir.
3. `ip netns exec <ns-adı> ip addr`: "Dışarıdan içeriye bakmak" için kullanılır. Node üzerindeyken, sanki Pod'un içindeymiş gibi komut çalıştırmanızı sağlar.
