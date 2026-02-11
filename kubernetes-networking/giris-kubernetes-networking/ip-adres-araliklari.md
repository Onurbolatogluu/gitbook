---
icon: arrow-up-from-water-pump
---

# IP Adres Aralıkları

#### 🗺️ CIDR ve IP Yönetimi

Kubernetes'te IP adresleri rastgele dağıtılmaz. Herkesin "Mahallesi" (CIDR Bloğu) bellidir. Metin, bu planlamanın 3 ana bölgeye ayrıldığını söylüyor:

**1. Üç Farklı Mahalle (Resource Types)**

Kubernetes, karışıklığı önlemek için adres defterini 3'e böler:

1. Node Mahallesi (Fiziksel):
   * _Nedir:_ Sunucuların kendi IP adresleridir.
   * _Kim Verir:_ Bunu Kubernetes vermez.  Ağ yöneticisi, Cloud sağlayıcın (AWS, Azure) veya modemindeki DHCP verir. Bunlar "Arsa"nın tapusudur.
2. Pod Mahallesi (Nüfus):
   * _Nedir:_ Uygulamaların (Pod'ların) yaşadığı yerdir. Her Pod buradan bir IP alır.
   * _Kim Verir:_ CNI Eklentisi (Calico, Cilium vb.) verir.
   * _Örnek:_ `10.244.0.0/16` aralığı Pod'lar içindir.
3. Service Mahallesi:
   * _Nedir:_ Pod'lara ulaşmak için kullanılan sabit "Danışma Hattı" numaralarıdır (ClusterIP).
   * _Kim Verir:_ Kubernetes API Server verir. Bu IP'ler sanaldır, fiziksel bir kablosu yoktur.
   * _Örnek:_ `10.96.0.0/12` aralığı Servisler içindir.

**2. No Overlap**

* Routing Sorunu: Eğer Pod'lara ayırdığın IP aralığı ile fiziksel ağındaki (Node) IP aralığı çakışırsa, postacı paketi nereye götüreceğini bilemez. Paket kaybolur.
* Cluster kurarken seçtiğin CIDR aralığı, şirketindeki diğer sunucuların IP'leriyle çakışmamalıdır. Yoksa o Pod, şirket veritabanına erişemez.

**3. Scalability**

* Pod CIDR bloğunu baştan geniş tutmalısın ki sistem büyüdüğünde IP bitmesin.
* IP-per-Pod: Her Pod bir IP yiyeceği için, havuzun Pod sayısına yetecek kadar büyük olması şarttır.

**4. Uygulayıcı: CNI**

* Sen kurulumda dersin ki: _"Pod'lar `192.168.0.0/16` bloğunu kullansın."_
* CNI Eklentisi (Usta), bu emri alır ve her yeni Pod oluşturulduğunda bu havuzdan bir numara seçip Pod'a yapıştırır. Pod silindiğinde numarayı geri alır ve havuza koyar.
