---
icon: user-group
---

# introduction to User Permissions

Sanallaştırma altyapısının ilk kurulum aşamasında, genellikle her şeyi yapabilme gücüne sahip tek bir süper kullanıcı hesabı yaratılır. ESXi sunucuları için bu efsanevi hesap `root`, vCenter mimarisi için ise `administrator` (genellikle _administrator@vsphere.local_) hesabıdır.

Sistem yöneticisi tek kişi olduğunda bu hesaplarla çalışmak kolaydır. Ancak BT altyapısı büyüdüğünde, ekibe yeni sistem uzmanları, ağ yöneticileri veya yedekleme operatörleri katıldığında işin rengi değişir. Herkesin aynı "God Mode" (Sınırsız Yetki) parolasını paylaşarak sisteme girmesi, kurumsal ortamlar için saatli bir bombadır.

Bu makalede, VMware ortamlarında "Rol Bazlı Erişim Kontrolü" (RBAC - Role-Based Access Control) mimarisinin neden zorunlu olduğunu, yetkilendirme katmanlarını ve kurumsal güvenlik standartlarını inceleyeceğiz.

**1. Neden Ortak "Root / Administrator" Hesabı Kullanmamalıyız?**

Bir veri merkezini yöneten 4 farklı uzman olduğunu düşünün. Hepsi vCenter'a `administrator` hesabı ile giriyorsa, altyapınızda şu iki ölümcül güvenlik açığı oluşur:

* Hesap Verilebilirlik (Accountability) Kaybı: Gece saat 03:00'te kritik bir SQL sunucusu yanlışlıkla silindi (Delete from Disk). Loglara (Görev ve Olaylar menüsüne) baktığınızda işlemi yapan kişi olarak sadece "Administrator" yazar. O an o parolaya sahip 4 kişiden hangisinin bu hatayı yaptığını (veya dışarıdan bir siber saldırganın mı sızdığını) asla tespit edemezsiniz.
* Yetki Aşımı: Sadece sanal makinelerin yedeklerini (Snapshot/Clone) almakla görevli bir operatörün, yanlışlıkla tüm veri depolama (Datastore) LUN'larını formatlama veya silme yetkisine sahip olması mimari bir faciadır.

**2. Görev Ayrılığı ve RBAC Mimarisi**

VMware vSphere, bu kaosu çözmek için son derece gelişmiş bir İzin ve Ayrıcalık (Permissions & Privileges) yapısı sunar. Temel mantık, yönetimi spesifik parçalara bölmektir:

* Sanal Makine Yöneticisi: Sadece yeni sanal makine oluşturma, açma/kapatma ve donanım (RAM/CPU) ekleme yetkisine sahiptir. Ağ ayarlarına veya Host donanımlarına dokunamaz.
* Ağ Yöneticisi: Sadece vSwitch'leri, Port Gruplarını ve VLAN taglerini yönetebilir. Sanal makinelerin içini göremez veya onları silemez.
* Depolama Yöneticisi: Sadece Datastore ekleme, iSCSI/NFS bağlantıları kurma yetkilerine sahiptir.

Bu sayede her uzman, sadece kendi işini yapmaya yetecek kadar bir arayüz ve menü yetkisiyle sistemi yönetir.

**3. İki Farklı Yönetim Katmanı: Host vs. vCenter**

Yetkilendirme mimarisi tasarlanırken altyapının hangi katmanında işlem yapıldığı çok önemlidir:

* ESXi Seviyesi Yetkilendirme: Doğrudan fiziksel sunucunun IP adresi üzerinden yapılan yönetimdir. Burada oluşturulan kullanıcılar sadece o spesifik fiziksel sunucu içinde yaşarlar.
* vCenter Seviyesi Yetkilendirme: Tüm veri merkezinin beyni olan vCenter üzerinde yapılan yetkilendirmedir. Burada bir kullanıcıya yetki verdiğinizde, bu yetki hiyerarşik olarak altındaki tüm Datacenter, Cluster ve Host'lara (sizin belirlediğiniz sınırlarda) uygulanır. Kurumsal yönetim daima bu katmandan yapılır.

> 💡 İpucu: "En Az Yetki Prensibi" ve Active Directory Entegrasyonu
>
> Kurumsal sahadaki en iyi pratik, vCenter üzerinde manuel olarak `Ahmet`, `Mehmet` gibi yerel (Local) kullanıcılar açmamaktır. Gerçek bir kurumsal mimaride vCenter, doğrudan Microsoft Active Directory (AD) veya LDAP sunucunuza entegre edilir.
>
> 1. Active Directory üzerinde `vCenter_Network_Admins` veya `vCenter_Read_Only` gibi Güvenlik Grupları (Security Groups) oluşturulur.
> 2. Bu gruplar vCenter'a tanıtılır ve ilgili VMware rolleri bu gruplara atanır.
> 3. BT ekibinden biri işten ayrıldığında, vCenter arayüzüne hiç girmeden sadece Active Directory'den hesabını kapatmanız, o kişinin sanallaştırma altyapısına olan tüm erişimini saniyeler içinde keser.
>
> Ayrıca, bir rol atanırken daima "En Az Yetki Prensibi" uygulanmalıdır. Bir kullanıcıya sadece görevini ifa etmesi için gereken asgari yetki verilmeli; "İleride lazım olur" mantığıyla gereksiz ayrıcalıklar kesinlikle tanımlanmamalıdır.
