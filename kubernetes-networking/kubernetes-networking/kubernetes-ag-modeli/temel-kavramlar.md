---
icon: diagram-sankey
---

# Temel Kavramlar

**1. Pod Mimarisi: Paylaşılan Namespace ve "Pause" Konteyneri**

* Kubernetes bir Pod oluştururken, önce "Pause Container" (veya _Infra Container_) adı verilen, çok az kaynak tüketen bir işlem başlatır. Bu konteynerin tek amacı, bir Network Namespace oluşturmak ve bu alanı açık tutmaktır.
* Asıl uygulama konteynerleri (örneğin Java uygulaması veya Nginx), başlatıldığında bu mevcut namespace'e dahil olur.
* Bu mimari sayesinde, bir Pod içindeki tüm süreçler aynı IP adresini ve port aralığını paylaşır. `localhost` üzerinden yapılan iletişim, fiziksel ağ kartına hiç uğramadan çekirdek seviyesinde gerçekleştiği için ağ gecikmesi neredeyse sıfırdır.

**2. Pod-to-Pod Bağlantı**

* Geleneksel Docker yapılarındaki gibi port mapping veya NAT işlemine gerek yoktur. Pod A'nın IP'si neyse, hedef Pod B bu paketi o IP adresinden geldiği şekliyle görür.
* Bu durum, ağ paketlerinin başlıklarının değiştirilmesi işlemini ortadan kaldırarak CPU yükünü azaltır ve izlenebilirliği artırır.

**3. CNI ve Sanal Kablolama**&#x20;

CNI (Container Network Interface), bu mantıksal yapıyı fiziksel (sanal) dünyaya bağlayan arayüzdür.

* Veth Pair: CNI eklentisi, Pod oluşturulduğunda bir ucu Pod'un namespace'inde (`eth0` olarak görünür), diğer ucu ise Node'un (Host) namespace'inde (`veth...` veya `cali...` gibi bir isimle) bulunan sanal bir kablo oluşturur.
* Pod'dan çıkan bir veri paketi, `eth0` üzerinden bu sanal tünelden geçer ve Node'un ağ namespace'ine ulaşır. Buradan sonra yönlendirme (routing) kuralları devreye girer ve paket hedefe iletilir.

**4. Tanılama ve İnceleme**

Sistem yöneticileri, Kubernetes katmanında bir sorun olduğunda (örneğin `kubectl` yanıt vermediğinde), Linux araçlarıyla sorunu kök seviyede analiz edebilir.

* `lsns` ve `ip netns`: Linux çekirdeğindeki izole alanları listeler. Hangi işlemin hangi ağ alanında çalıştığını doğrulamak için kullanılır.
* Eğer Pod'un IP'si ile `ip addr` komutunda görülen sanal arayüz eşleşiyorsa, CNI eklentisi görevini doğru yapmış demektir.

***
