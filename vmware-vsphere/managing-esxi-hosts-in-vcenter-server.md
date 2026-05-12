---
icon: people-roof
---

# Managing ESXi Hosts in vCenter Server

Sanallaştırma ortamına vCenter'ı kurup sunucularımızı içeri aldıktan sonra, sistem yöneticisi olarak mesaimizin neredeyse tamamını geçireceğimiz o devasa arayüzünü tanıma vakti gelir.

vCenter arayüzünün en önemli özelliği, sol taraftaki ağaç yapısında (Inventory Tree) nereye tıklarsanız, sağ taraftaki ekranın o objenin boyutuna göre şekil değiştirmesidir. Bu, sistemin "Bağlama Duyarlı" çalışması demektir.

* Datacenter Seviyesi: Ağacın en tepesindeki Datacenter'a tıkladığınızda, sağ taraftaki _Summary (Özet)_ sekmesi size o veri merkezindeki tüm donanımların (tüm host'ların) birleşik gücünü ve ortamdaki tüm makinelerin genel özetini verir.
* Host Seviyesi: Ağaçta bir alt dala inip spesifik bir ESXi Host'a (Örn: Host-10) tıkladığınızda, ekran anında daralır. Artık sadece o fiziksel sunucunun işlemcisi, donanım sağlığı, takılı olan fiziksel ağ kartları (vmnic) ve sadece onun üzerinde koşan sanal makineler listelenir.
* VM Seviyesi: Bir adım daha inip doğrudan bir sanal makineye (VM) tıkladığınızda, ekran tamamen o işletim sistemine odaklanır. O makinenin anlık CPU/RAM tüketimi, alınan Snapshot'ları ve konsol ekranı karşınıza çıkar.

Menü isimleri hep aynıdır (_Summary, Monitor, Configure, Permissions_), ancak içleri bulunduğunuz hiyerarşik konuma (Datacenter > Host > VM) göre tamamen farklı verilerle dolar.

**2. Enhanced Linked Mode**

Diyelim ki şirketinizin İstanbul, Ankara ve İzmir'de 3 ayrı veri merkezi ve 3 ayrı vCenter'ı var. Bunları ELM mimarisiyle kurarsanız, İstanbul'daki vCenter arayüzüne giriş yaptığınızda, sol taraftaki ağaç yapısında Ankara ve İzmir'i de görürsünüz. Tek bir tıkla, arayüz değiştirmeden veya yeniden şifre girmeden, İstanbul'dan İzmir'deki bir sunucuya müdahale edebilirsiniz. Global Inventory List, işte bu devasa birleşik ortamlardaki tüm makineleri tek bir listede görebilmek için tasarlanmıştır.

**3. Farklı Bakış Açıları**

* Hosts and Clusters: Donanım odaklı bakış açısıdır. Sunucular, işlemci güçleri ve Resource Pool buradan yönetilir.
* VMs and Templates: Sanal makine odaklı bakış açısıdır. Folder yapısı tam olarak burada inşa edilir.
* Storage & Network: Datastore'ların ve sanal switch'lerin topluca yönetildiği izole görünümlerdir.

Bu farklı pencereler, "Aynı ortama farklı gözlüklerle bakmanızı" sağlar. Donanım arızası ararken _Hosts and Clusters_ gözlüğünü takarsınız; makineleri departmanlara göre klasörlerken _VMs and Templates_ gözlüğünü takarsınız.

**🎯 Temel Mesaj**

"vCenter'ı kurup sunucularımızı içeri aldıktan sonra, artık ESXi sunucularına doğrudan bağlanmaya ihtiyacımız yoktur."

Bir makine yaratılacaksa, silinecekse, ağ ayarı değiştirilecekse veya snapshot alınacaksa; bunların tamamı artık vCenter üzerinden yapılır. vCenter, ortamın tek hakimi (Single Source of Truth) olmuştur. Eski usul doğrudan Host'a bağlanma işlemi, sadece vCenter'ın tamamen çöktüğü acil felaket durumlarında (Disaster Recovery) kullanılan bir arka kapı olarak kalmalıdır.
