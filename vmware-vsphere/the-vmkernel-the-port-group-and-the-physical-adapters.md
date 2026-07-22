---
icon: ethereum
---

# The VMkernel, The Port Group and The Physical Adapters

VMware vSphere ortamlarında sağlam, güvenli ve performanslı bir altyapı kurmanın ilk kuralı, ağ mimarisini doğru tasarlamaktır. Bir ESXi sunucusu kurulduğunda varsayılan olarak bir "Sanal Switch" (Standard Switch) ve tek bir fiziksel ağ kartı bağlantısıyla gelir. Test ortamlarında işe yarayan bu basit yapı, kurumsal veri merkezlerinde ciddi performans ve güvenlik sorunlarına yol açabilir.

Peki, profesyonel bir ağ topolojisi nasıl kurgulanmalıdır? Bu makalede, ESXi ağ mimarisinin kalbini oluşturan üç temel bileşeni ve bunları en verimli şekilde nasıl kullanacağımızı inceleyeceğiz.

### 1. VM Port Group: Sanal Makinelerin Ağa Çıkış Kapısı

Sanal makinelerin (Virtual Machines) dış dünyayla iletişim kurabilmesi için bağlandıkları sanal yuvalara **VM Port Group** denir.

Bu bileşeni anlarken unutulmaması gereken en önemli kural şudur: **VM Port Group'ların kendilerine ait bir IP adresi, yönlendirme yeteneği veya bir işletim sistemi yoktur.** Onlar tıpkı duvardaki bir elektrik prizi veya ağ kablosu ucu gibi, sadece verinin üzerinden geçip gittiği pasif geçiş kanallarıdır.

Bir sanal makine bu gruba bağlandığında; IP adresi alma, ağ geçidini (Gateway) belirleme veya DHCP sunucusuyla konuşma işlemleri tamamen sanal makinenin içindeki işletim sisteminin (Windows, Linux vb.) sorumluluğundadır. ESXi, bu trafiğin içeriğine karışmaz, sadece iletilmesini sağlar.

### 2. VMkernel Port: ESXi Sunucusunun Kendi Ağ Kimliği

Sanal makinelerin trafiğini bir kenara bırakalım; ESXi sunucusunun bizzat kendisinin (Hypervisor) ağ üzerinde aktif işlemler yapması gerekir. Örneğin:

* Sizin sunucuyu vCenter üzerinden yönetmeniz (Management),
* Sunucunun harici bir veri depolama cihazına bağlanması (IP Storage - iSCSI/NFS),
* Diğer sunucularla sanal makine transferi yapması (vMotion).

ESXi'ın bu kritik altyapı operasyonlarını yürütebilmesi için, tıpkı standart bir bilgisayar gibi kendine ait bir IP adresine ihtiyacı vardır. İşte ESXi'ın kendi kullanımı için ayrılmış, üzerine doğrudan IP adresi ve ağ ayarları tanımlanabilen bu "akıllı" arayüzlere **VMkernel Port** adı verilir.

_(Özetle: Sanal makine trafiği VM Port Group'tan, ESXi'ın kendi yönetim ve altyapı trafiği ise VMkernel Port'tan geçer.)_

### 3. Trafik İzolasyonu Neden Kritik Bir Zorunluluktur?

Fiziksel bir sunucunun arkasında genellikle birden fazla fiziksel ağ kartı (Physical Adapter / Uplink) bulunur. Tüm trafiği (sanal makineler, yönetim, vMotion ve Storage) tek bir fiziksel karta yığmak, mimaride yapılabilecek en büyük hatalardan biridir.

Bunun iki temel nedeni vardır:

1. **Performans (Darboğaz):** vMotion veya harici bir veri depolama ünitesine doğru yapılan veri transferleri ağ üzerinde devasa bir yük oluşturur. Bu ağır yük, sanal makinelerin normal internet trafiğiyle aynı fiziksel kabloya sıkıştırılırsa, ağda ciddi yavaşlamalar (Latency) ve paket kayıpları yaşanır.
2. **Güvenlik:** Dış dünyaya veya internete açık olan sanal makinelerin trafiği ile, sunucuların kalbi olan Storage (depolama) veya yönetim trafiğinin aynı fiziksel hattan geçmesi büyük bir güvenlik zafiyeti yaratır.

**Best Practice:** Fiziksel sunucunun arkasındaki ağ kartları, görevlerine göre izole edilmelidir. Örneğin; birinci ve ikinci kartlar sadece Sanal Makine trafiğine, üçüncü kart sadece Storage trafiğine, dördüncü kart ise vMotion ve Management trafiğine atanmalıdır.

### 4. Birden Fazla Sanal Switch ile Tam İzolasyon

Güvenliğin en üst düzeyde tutulması gereken senaryolarda, tek bir ESXi host içerisinde **birden fazla bağımsız Sanal Switch** (Örn: vSwitch0 ve vSwitch1) oluşturabilirsiniz.

* **vSwitch0'a** birinci fiziksel kabloyu,
* **vSwitch1'e** ise ikinci fiziksel kabloyu bağladığınızı düşünelim.

Bu iki farklı switch üzerindeki sanal makineler, birbirleriyle kesinlikle haberleşemezler. İletişim kurabilmelerinin tek yolu, bu iki farklı fiziksel kablonun bağlandığı dışarıdaki donanımsal (fiziksel) switch üzerinde bir yönlendirme (Routing) yapılmasıdır. Bu yöntem, özellikle DMZ kurulumlarında dışa açık sunucularla iç ağ sunucularını birbirinden fiziksel olarak yalıtmak için mükemmel bir çözümdür.

### Sonuç

Sanallaştırma dünyasında ağ mimarisi kurgularken kafanız karıştığında, sistemin temel altın kuralını hatırlayın: **"Sanal switch'i, her zaman gerçek ve donanımsal bir fiziksel switch olarak hayal edin."**

Sanal dünyada yaptığınız her işlemin fiziksel dünyada mantıklı bir karşılığı vardır. Farklı trafik türlerini farklı fiziksel kablolara doğru paylaştırmak ve bileşenlerin (VM Port Group ile VMkernel) sınırlarını doğru çizmek; hızlı, hataya dayanıklı ve güvenli bir veri merkezi mimarisinin en temel anahtarıdır.
