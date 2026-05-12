---
icon: readme
---

# Difference Between ESXi And vCenter Server User Interface

#### 🎛️ Yönetimden Orkestrasyona Geçiş: ESXi ve vCenter Arayüzleri Arasındaki Uçurum

Sanallaştırma ortamlarında tekil sunuculardan merkezi bir yapıya (vCenter) geçiş yapıldığında, sistem yöneticilerinin karşısına çıkan ilk somut değişiklik kullanıcı arayüzlerindeki (UI) farklılıklardır. ESXi arayüzü sadece o sunucunun donanımını yönetmenizi sağlarken, vCenter arayüzü tüm veri merkezini tek bir zeka altında birleştiren bir orkestrasyon katmanıdır.

İki arayüz arasındaki bu fark, sadece menülerin kalabalıklaşması değil, altyapının yeteneklerinin kurumsal seviyeye çıkmasının görsel bir kanıtıdır.

**1. "Managed by vCenter" Uyarısı ve Yönetim Devri**

vCenter'a eklenmiş bir ESXi Host'un IP adresini yazarak doğrudan kendi arayüzüne (Host Client) bağlandığınızda, sistem sizi sarı bir uyarı bandıyla karşılar: _"This host is being managed by vCenter Server..." (Bu host vCenter Server tarafından yönetilmektedir)._

* Bunun Anlamı Nedir? Bu sadece bir bilgilendirme mesajı değildir. Siz ESXi'ı vCenter'a eklediğiniz an, vCenter bu Host'un içine `vpxa` adında gizli bir agent kurar. Bu uyarı, Host'un kontrolü merkeze devrettiğini ve arkada sizin haberiniz olmadan DRS veya vSphere HA gibi servislerin bu makineye müdahale edebileceğini belirtir. Yönetim artık tekil değil, merkezidir.

**2. Sağ Tık Menülerindeki Büyük Fark**

ESXi Arayüzünde Durum:

Standalone (bağımsız) bir ESXi üzerinde sanal makineye sağ tıkladığınızda sadece temel operasyonları görürsünüz: Power On/Off, Snapshot alma, Edit Settings (Ayarları Düzenle) ve Console açma. Host seviyesinde ise sadece bakım modu (Maintenance Mode) ve temel ağ ayarları bulunur.

vCenter Arayüzünde Durum:

Aynı makineye vCenter üzerinden sağ tıkladığınızda menü inanılmaz derecede genişler. Karşınıza çıkan ve ESXi'ın tek başına asla yapamayacağı o gelişmiş özellikler şunlardır:

* Migrate (vMotion): Sanal makineyi kapatmadan (sıfır kesintiyle) bir Host'tan diğerine taşımak.
* Clone & Template: Makinenin birebir kopyasını almak veya ileride hızlıca yeni makineler oluşturmak için onu bir şablona (Template) dönüştürmek.
* Fault Tolerance (FT): Makinenin anlık bir kopyasını diğer Host üzerinde senkron çalıştırarak donanım arızalarında sıfır veri kaybı sağlamak.
* Resource Pool ve Host Profile: Kaynakları (CPU/RAM) belirli havuzlara bölmek ve Host'ların yapılandırmalarını tek bir profil üzerinden standartlaştırmak.

_(Kritik Not: vMotion veya Fault Tolerance gibi özelliklerin menüde aktif olabilmesi için ortamda mutlaka en az iki adet ESXi Host bulunması ve vCenter tarafından yönetiliyor olması gerekir.)_

**3. Datacenter Özeti: Kaynakların Birleşimi**

ESXi arayüzü sadece kendi donanımını (Örn: 4 GB RAM, 5 GHz CPU) görür. Ancak vCenter, bu Datacenter içerisindeki tüm Host'ları donanımsal bir "havuz" olarak toplar.

* İki adet 4 GB RAM'li Host'u vCenter'a eklediğinizde, Datacenter özet ekranında toplam kapasiteyi 8 GB RAM ve birleşik CPU gücü olarak görürsünüz.
* Datastore İsimlendirme Çakışması: ESXi kurulduğunda kendi yerel diskine varsayılan olarak `datastore1` adını verir. İki farklı ESXi'ı vCenter'a eklediğinizde, vCenter veritabanında isim çakışması olmaması için ikinci Host'un diskini arayüzde otomatik olarak `datastore1 (1)` şeklinde yeniden adlandırır. Bu, vCenter'ın ortamdaki benzersizliği (uniqueness) koruma yöntemidir.

**💡 Ekstra Bilgi: Tek Ekran (Single Pane of Glass) Kavramı**

Sanallaştırma literatüründe vCenter'ın sunduğu bu yapıya "Single Pane of Glass" denir. Sistem yöneticisi; Host & Clusters sekmesinden donanımları, VMs sekmesinden yüzlerce sanal makineyi, Datastore sekmesinden depolama alanlarını ve Network sekmesinden tüm sanal ağları (Port Group ve vSwitch yapılarını) tek bir ekranda filtreleyebilir ve yönetebilir.

Özetle; ESXi arayüzü bir arabanın motor kaputunun altıdır, donanımla birebir muhatap olursunuz. vCenter arayüzü ise o arabanın gelişmiş otonom sürüş konsoludur; filoyu yönetir, rotayı çizer ve kurumsal seviyede güvenliği sağlarsınız.



