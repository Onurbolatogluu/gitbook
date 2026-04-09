---
icon: autoprefixer
---

# ESXi Hardware Requirements

Şu ana kadar sanallaştırmanın ne olduğunu, kaynakların (CPU, RAM) nasıl dağıtıldığını ve vCenter ile büyük resmi nasıl yöneteceğimizi teorik olarak öğrendik. Şimdi sıra bu mimariyi kendi bilgisayarımızda inşa etmeye geldi.

Gerçek bir veri merkezinde VMware ESXi doğrudan çıplak donanıma (fiziksel sunucuya) kurulur. Ancak bizim evde veya ofiste devasa fiziksel sunucularımız olmadığı için, "İç İçe Sanallaştırma" (Nested Virtualization) denilen bir yöntem kullanacağız. Yani laptop'ımızdaki VMware Workstation'ın (Type 2) içine, bir sanal makine (VM) açıp onun içine ESXi (Type 1) kuracağız.

#### 💿 1. ISO Seçimi ve Otomatik Gereksinim Algılama

Sanal makine oluşturma sihirbazına başladığınızda, sistem size işletim sistemini nasıl kuracağınızı sorar.&#x20;

_Sistemi "İşletim sistemini sonra kuracağım" diyerek boş geçmeyin._ Başlangıçta indirdiğiniz ESXi ISO dosyasını gösterin.

Workstation bu ISO dosyasını okuduğunda, kuracağınız sistemin bir ESXi (Hypervisor) olduğunu anlar ve sizin için arka planda minimum donanım gereksinimlerini otomatik olarak şablona yazar (Örneğin minimum 4 GB RAM ve 2 vCPU). Eğer bunu yapmaz ve manuel ayarlarda RAM'i 2 GB bırakırsanız, kurulum ekranında "Yetersiz donanım" hatası alır ve başa dönersiniz.

#### 💾 2. Disk Yapılandırması

ESXi'ın kendisi çok küçük bir işletim sistemidir (kurulumu ortalama 14-15 GB alan kaplar). Ancak unutmayın, ESXi'ın amacı kendi içinde başka sanal makineler barındırmaktır.

Bu yüzden disk kapasitesi belirlerken:

* ESXi'ın kendisi için gereken minimum alan (\~15 GB).
*   İçine kurmayı planladığınız VM'ler (örneğin 2 tane 5 GB'lık test Linux makinesi kuracaksanız +10 GB).

    Bu hesaplamayla diski başlangıçta 25-30 GB olarak ayarlamak, ileride sorun yaşamanızı engeller.

Disk Bölme Seçeneği: "Split virtual disk into multiple files" seçeneği, sanal diskinizi 2 GB'lık küçük parçalara böler. Bu, ileride bu Lab ortamını bir USB belleğe kopyalayıp başka bir bilgisayara taşımak isterseniz işinizi kolaylaştırır. Performans testleri yapmıyorsanız Lab ortamları için bu seçenek tavsiye edilir.

#### 🧠 3. CPU ve RAM Limitleri

* RAM Sınırı: ESXi'ın çalışması için tavsiye edilen minimum değer 4 GB'tır. Ancak Workstation size bir maksimum limit gösterir (Örneğin laptop'ınızda 16 GB RAM varsa maksimum limit 16 GB'tır). Sistemin tamamen çökmemesi (kendi Windows'unuza da RAM kalması) için test ortamında 4 GB ile 8 GB arası bir değer vermek en mantıklısıdır.
* CPU Sınırı: Laptop'ınızda 4 fiziksel çekirdek varsa, ESXi sanal makinesine 5 çekirdek veremezsiniz. Sistem anında hata verir. Kurulum için 2 çekirdek yeterli olacaktır.
* BIOS Virtualization (VT-x / AMD-V): Sanal makinenin içinde sanal makine çalıştıracağımız (Nested) için, host makinenizin (laptop'ınızın) BIOS/UEFI ayarlarından Intel VT-x veya AMD-V sanallaştırma teknolojisinin "Enabled" (Açık) konumda olduğundan kesinlikle emin olmalısınız.

#### 🌐 4. Ağ Bağlantısı (Network Adapter) Seçimi: Neden "Bridge"?

* NAT: Seçerseniz, Workstation kendi içinde yalıtılmış bir ağ yaratır. ESXi'ınız sizin ev ağınızdan farklı (Örn: 192.168.100.x) bir IP alır. Dışarıdan ona ulaşmak veya port yönlendirmek işkenceye dönüşebilir.
* Bridge: Bu modu seçtiğinizde, sanal ESXi makineniz laptop'ınızın Wi-Fi veya Ethernet kartını bir "köprü" olarak kullanarak doğrudan evinizdeki ADSL/Fiber modeminize (router'a) bağlanır. Evdeki modeminiz ona sıradan bir cihazmış gibi (Örn: 192.168.1.50) temiz bir IP adresi verir.

