---
icon: bezier-curve
---

# Introduction To Virtual Network Standard Switches

Sanallaştırma dünyasına adım atan birçok sistem yöneticisinin en çok çekindiği konuların başında Virtual Networking (Sanal Ağlar) gelir. Çünkü kabloları, donanımları veya switch cihazlarını gözümüzle göremeyiz; her şey bir ekranın içindedir.

Bu makalede, VMware ESXi üzerindeki Standard Switch mimarisine giriş yapacak, sanal ağların Hypervisor üzerinde nasıl kurgulandığını adım adım ve en temel haliyle inceleyeceğiz.

**1. Fiziksel Dünyadan Sanal Dünyaya Geçiş**

Kafanızdaki karmaşayı gidermenin tek bir yolu var: Sanal ağların, fiziksel ağ donanımlarının sadece "yazılıma dönüştürülmüş" hali olduğunu kabul etmek. Mimari tamamen aynıdır.

Fiziksel bir sistem odasında bir sunucuyu ağa bağlamak için ne yaparsınız?

Sunucunun arkasındaki ağ kartına (NIC) bir kablo takar, o kablonun diğer ucunu da kabin içindeki donanımsal bir Switch cihazına bağlarsınız.

VMware ESXi üzerinde bu fiziksel parçaların yazılımsal karşılıkları şunlardır:

* Fiziksel Sunucu 👉 Virtual Machine olur.
* Fiziksel Ağ Kartı (NIC) 👉 vNIC (Virtual Network Adapter) olur.
* Fiziksel Switch Cihazı 👉 Standard Switch olur.

Yani ESXi arayüzünde bir Virtual Machine'i bir ağa bağladığınızda, aslında yaptığınız şey sanal bir ağ kartını (vNIC), ESXi içindeki yazılımsal bir switch'e (Standard Switch) bağlamaktan ibarettir.

**2. Virtual Machine Ağla Nasıl Konuşur?**

Bir Virtual Machine, donanımsız bir sanal dünyada yaşadığını bilmez. İşletim sistemi (örneğin Windows), kendisine gerçekten bir ağ kartı takıldığını sanır ve internete çıkmak için verilerini bu karta (vNIC) gönderir. İşin asıl büyüsü ESXi (Hypervisor) işletim sisteminde başlar.

Hypervisor, Virtual Machine'den çıkan bu veriyi alır ve ESXi'ın hafızasında (RAM) çalışan yazılımsal Standard Switch'e iletir.

Aynı Host İçindeki İletişim:

Eğer o Virtual Machine'in veri göndermek istediği hedef, aynı ESXi Host içinde ve aynı Standard Switch'e bağlı başka bir Virtual Machine ise; Hypervisor veriyi dışarıdaki fiziksel kablolara hiç yollamaz. Bunun yerine, veriyi birinci makinenin RAM adresinden okur ve doğrudan ikinci makinenin RAM adresine kopyalar. Sistem fiziksel donanımları ve kablo limitlerini beklemediği için bu aktarım inanılmaz hızlı gerçekleşir.

**3. Standard Switch'in İki Kritik Parçası: Port Group ve Uplink**

Sanal makinelerin ESXi içinde kendi aralarında nasıl konuştuğunu anladık. Peki bu makineler dış dünyaya (İnternete veya şirketinizin gerçek fiziksel ağına) nasıl çıkacak? İşte bu noktada karşımıza iki temel kavram çıkar:

*   Port Group (Sanal Port Grupları):

    Fiziksel bir switch üzerinde yan yana dizilmiş onlarca port vardır ve siz kabloyu Port-1'e mi yoksa Port-5'e mi takacağınızı seçersiniz. Standard Switch'te ise numaralı portlar yerine Port Group oluştururuz. Örneğin "Muhasebe Ağı (VLAN 10)" adında bir Port Group açar ve muhasebe sunucularını bu gruba bağlarsınız. Port Group, sanal makinelerin virtual Standart Switch üzerinde bağlandığı yazılımsal yuvalardır.
*   Uplink (Dışarı Çıkış Kapısı):

    Standard Switch, ESXi sunucusunun içinde hapis kalmış yazılımsal bir cihazdır. İnternete çıkabilmesi için gerçek, fiziksel bir ağ kartına ihtiyacı vardır. İşte Standard Switch'in, ESXi sunucusunun arkasında takılı olan gerçek fiziksel ağ kartlarına bağlanmasına Uplink denir.

    Virtual Machine'den çıkan veri Standard Switch'e gelir ve Uplink (fiziksel ağ kartı) üzerinden dışarıdaki gerçek fiziksel switch'e iletilir.

**4. Standard Switch'in Sınırları**&#x20;

Standard Switch kusursuz çalışır, ancak çok büyük bir zayıflığı vardır: Sadece içinde bulunduğu ESXi Host'a özeldir.

Ortamınızda 1 veya 2 adet ESXi Host varsa Standard Switch kullanmak mükemmel ve basittir. Ancak ortamınızda 50 adet Host varsa, "VLAN 10 - Muhasebe" adında bir Port Group oluşturmak için 50 farklı ESXi Host'a tek tek girip aynı ayarı 50 kez yapmanız gerekir. Sistem yöneticilerini yoran şey bu tekrarlayan operasyonel yüktür.

Bu sorunu çözmek için VMware, Distributed Switch (vDS) mimarisini geliştirmiştir. vDS sayesinde vCenter üzerinden tek bir devasa sanal switch yaratır ve bunu tüm Host'lara tek bir tıkla dağıtırsınız.

Özetle; Virtual Networking mimarisi, bildiğimiz donanımsal ağ cihazlarının ESXi içinde yazılıma dönüşmüş halidir. Tüm yapıyı kafanızda netleştirmek için şu 3 adımlık basit köprüyü kurmanız yeterlidir:

1. Virtual Machine, Standard Switch üzerinde kendisi için ayrılan yuvaya (Port Group) bağlanır.
2. Standard Switch, sunucu içindeki sanal ağ trafiğini toplar ve yönetir.
3. Uplink (fiziksel ağ kartı), bu trafiği Standard Switch'ten devralıp dış dünyaya, yani gerçek fiziksel ağa çıkarır.

Bu basit işleyişi anladığınızda, Virtual Networking karmaşık bir sistem olmaktan çıkıp son derece anlaşılır ve yönetilebilir bir yapıya dönüşür.



