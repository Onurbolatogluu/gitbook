---
icon: h
---

# Hardware Virtualization

Fiziksel bir sunucuyu ( physical server ) alıp sanal bir altyapıya ( virtual architecture ) taşıdığımızda kaputun altında tam olarak ne olur? Bir sistemi ilk defa sanallaştıran herkesin aklına şu soru gelir: _"İyi de benim elimde 1 tane anakart, kısıtlı sayıda işlemci var; birden fazla işletim sistemi bu donanımı aynı anda nasıl kullanıyor?"_

#### ⚙️ Kaputun Altındaki Sihir: Hypervisor ve Sanal Donanım

Geleneksel yapıda donanım (CPU, RAM, Disk) doğrudan üzerine kurulan tek bir işletim sistemine ve onun üzerindeki uygulamalara hizmet eder.

Sanal mimaride ise hypervisor devreye girer. Hypervisor, fiziksel host üzerindeki donanımı alır ve bunları "sanal donanımlara" ( virtual hardware ) dönüştürür. Artık yarattığınız her bir virtual machine (VM), kendine ait izole bir CPU'su, RAM'i ve network adapter'ı olduğuna inanır.

***

#### 🧠 CPU Virtualization ve Sınırları Aşmak

Sanallaştırmanın en vurucu noktası CPU yönetimidir. Normal şartlarda 4 fiziksel CPU'nuz varsa ve sadece 1 CPU kullanan eski nesil bir uygulama çalıştırıyorsanız, geriye kalan 3 CPU boşta yatar ve israf olur.

Ancak sanal dünyada vSphere, elinizdeki fiziksel işlemcileri bir "havuz" (pool) gibi yönetir. Hatta öyle bir yönetir ki, elinizdeki fiziksel kapasiteden daha fazlasını sanal makinelere dağıtabilirsiniz. Sektörde buna CPU Overcommitment diyoruz.

Nasıl Mümkün Oluyor? (Time-Slicing)

Diyelim ki 4 fiziksel CPU'ya sahip bir host'unuz var. Siz bu sistemde şu sanal makineleri (VM) oluşturdunuz:

* VM 1: 1 vCPU (Sanal CPU)
* VM 2: 2 vCPU
* VM 3: 4 vCPU
* Toplam Dağıtılan: 7 vCPU.

Sistem çökmez, çünkü vSphere Scheduler inanılmaz hızlı bir trafik polisi gibi çalışır. Hiçbir sunucu 7/24 tam kapasite çalışmaz. VM 2 o an boşta beklerken, vSphere onun hakkı olan fiziksel işlemci gücünü alır ve ağır işlem yapan VM 3'e kaydırır. Bu işlem mikrosaniyeler içinde gerçekleştiği için (Time-Slicing), sanal makineler işlemcilerin tamamen kendilerine ait olduğunu sanır.

> ⚠️ Altın Kural: Bir sanal makineye, host'un sahip olduğu toplam fiziksel CPU'dan daha fazlasını veremezsiniz (Yani 4 fiziksel CPU'nuz varsa, tek bir makineye 5 vCPU veremezsiniz). Ayrıca tüm VM'ler aynı anda %100 yük altına girerse sistemde darboğaz başlar; bu tehlikeli bekleme süresine CPU Ready Time denir.

***

#### 🌐 Network Virtualization

Donanımdan bağımsızlaşmanın ikinci büyük adımı ağ (network) tarafındadır. Eski dünyada, iki sunucunun konuşması için fiziksel ağ kartlarından kablo çıkması ve dışarıdaki bir fiziksel switch'e bağlanması gerekirdi.

Sanal dünyada hypervisor, kendi içinde sanal bir switch ( vSwitch ) oluşturur. İçerideki VM'ler, sanal ağ kartları ve sanal kablolar ile bu vSwitch'e bağlanır.

Sonuç: Aynı host üzerindeki bir Web Sunucusu ve Veritabanı Sunucusu, fiziksel hiçbir ağ kartını veya dışarıdaki kabloyu meşgul etmeden, hypervisor'ün vSwitch'i üzerinden bellek hızında haberleşir. Fiziksel network adapter sadece sanal makineler dış dünyaya (internete) çıkmak istediğinde kullanılır.

```
[Network Virtualization Topolojisi]

+-------------------------------------------------------------+
| FİZİKSEL HOST                                               |
|                                                             |
|  +--------------+               +--------------+            |
|  |     VM 1     |               |     VM 2     |            |
|  | (Web Server) |               |  (Database)  |            |
|  +------|-------+               +-------|------+            |
|         | (Virtual Adapter)             | (Virtual Adapter) |
|         +----------[ vSwitch ]----------+                   |
|         (Hypervisor içindeki Sanal Switch)                  |
|                         |                                   |
|                 Physical Network                            |
|                     Adapter                                 |
+-------------------------|-----------------------------------+
                          |
                   [ Dış Dünyaya (İnternet) ]
```

#### 🎯 Özetle

Sanallaştırma sadece bir yazılım illüzyonu değil; fiziksel sınırların yazılımla aşılmasıdır. Donanım bağımsızlığı, esnek kaynak dağıtımı ve sanal ağ mimarisi sayesinde, modern veri merkezleri maksimum verimlilikle çalışır.
