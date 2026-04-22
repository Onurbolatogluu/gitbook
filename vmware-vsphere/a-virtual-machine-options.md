---
icon: magnifying-glass-arrows-rotate
---

# A Virtual Machine Options

vSphere Client veya ESXi Web Arayüzü üzerinden bir sanal makinenin üzerine tıklandığında, sistem yöneticisini karşılayan kokpit ekranı, makinenin tüm hayati verilerini ve yönetim araçlarını barındırır.

Henüz "Power On" tuşuna basıp kurulum ekranına geçmeden önce, makine ayarlarını nasıl yöneteceğimizi ve fiziksel bir CD-ROM olmadan işletim sistemini nasıl kuracağımızı inceleyelim.

#### 🕵️‍♂️ Thin Provisioning'in Canlı Kanıtı

Makinenin özet (Dashboard) ekranına bakıldığında göze çarpan ilk detaylardan biri depolama (Storage) alanıdır. Bir önceki aşamada makineye 14 GB kapasiteli ve Thin Provisioned (Kullandıkça Büyüyen) bir disk tanımladığımızı varsayalım.

Özet ekranında, "Used Space" (Kullanılan Alan) değerinin yalnızca birkaç kilobayt (örneğin 2 KB) olduğu görülür. Henüz içine bir işletim sistemi kurulmadığı ve hiçbir veri yazılmadığı için makine fiziksel Datastore üzerinde adeta bir hayalet gibidir. Ayrıca, içine bir işletim sistemi yüklenmediği ve ağa dahil olmadığı için "Host Name" ve "IP Address" gibi alanlar da doğal olarak boş görünecektir.

#### 🛠️ "Edit Settings" Esnekliği

Sanallaştırma dünyasının en büyük avantajı, Yeni oluşturulmuş bir sanal makinenin kaynaklarını değiştirmek isterseniz, kasayı açıp donanım takmanıza gerek kalmaz.

Yönetim panelindeki "Edit Settings" menüsü, makinenin donanım kalbidir. Bu menü üzerinden:

* RAM miktarını anında artırabilir veya azaltabilirsiniz.
* Yeni bir sanal disk (.vmdk) ekleyebilir veya mevcut diskin kapasitesini genişletebilirsiniz.
* Ağ kartı (Network Adapter) ayarlarını değiştirebilirsiniz.

Tüm bu işlemler, fiziksel dünyada saatler sürecek operasyonları saniyelere indirger.

#### 💿 İşletim Sistemini Boot Etmek: CD/DVD Sürücüsü Çıkmazı

İçi tamamen boş olan bu yeni donanım şablonuna bir işletim sistemi (Örneğin Windows Server 2016) kurmak zorundayız. Fiziksel bir bilgisayarda olsaydık USB belleği veya DVD'yi kasaya takardık. Peki, kilometrelerce uzaktaki bir veri merkezinde çalışan sanal makineye Windows'u nasıl kuracağız?

"Edit Settings" menüsündeki CD/DVD Drive sekmesi tam olarak bu işe yarar. Burada karşımıza çıkan iki temel seçenek vardır:

1\. Host Device (Fiziksel Sunucu Sürücüsü)

Eğer bu seçenek tercih edilirse, sanal makine, ESXi'ın kurulu olduğu asıl fiziksel sunucunun üzerindeki gerçek CD-ROM'u okumaya çalışır. Kurumsal ortamlarda bu yöntem kesinlikle kullanışsızdır. Çünkü sunucular genellikle erişimi zor, yüksek güvenlikli veri merkezlerinde kilitli kabinler (Rack) içinde yer alır. Her format işlemi için sistem yöneticisinin veri merkezine gidip fiziksel diski takması beklenemez.

2\. Datastore ISO File (Sanal İmaj Bağlama)

Gerçek IT operasyonlarında kullanılan standart yöntem budur. İşletim sistemi kurulum medyasının dijital bir kopyası (ISO dosyası), ESXi sunucusunun fiziksel disk alanına (Datastore) yüklenir. Ardından, sanal makinenin CD/DVD sürücüsüne bu ISO dosyası bir "Sanal DVD" olarak bağlanır (Mount). Makine başlatıldığında, optik sürücüde fiziksel bir DVD varmış gibi doğrudan bu dosyadan boot eder.

#### 📁 Datastore Yönetimi ve Klasör Hiyerarşisi

ISO dosyasını sanal makineye bağlamadan önce Datastore'un iç yapısını düzenlemek, kurumsal mimarinin yazılı olmayan altın kurallarından biridir.

Sistem, oluşturulan her yeni sanal makine için Datastore içinde o makinenin adını taşıyan bir klasör (Örn: `Win2016_ActiveDirectory`) açar ve makineye ait disk/yapılandırma dosyalarını buraya koyar.

Önemli Kural: Kurulum ISO dosyalarını asla rastgele makine klasörlerinin içine veya Datastore'un kök dizinine atmayın.

Bunun yerine, "Datastore Browser" aracını kullanarak doğrudan "ISO\_Files" (veya benzeri) adında merkezi bir klasör oluşturun. Tüm şirketinizin işletim sistemi arşivini (Windows imajları, Linux dağıtımları, kurtarma araçları) bu klasöre yükleyin.

Böylece, yeni bir sanal makine kuracağınız zaman "Edit Settings -> CD/DVD Drive -> Datastore ISO File" yolunu izleyip, doğrudan bu merkezi arşiv klasöründen istediğiniz işletim sistemini saniyeler içinde makineye bağlayabilirsiniz.

Tüm donanım atamaları, disk stratejileri ve sanal DVD bağlama işlemleri tamamlandığına göre; altyapı hazırlıkları bitmiştir. Artık geriye kalan tek şey "Power On" tuşuna basmak ve işletim sisteminin kurulum adımlarını başlatmaktır.
