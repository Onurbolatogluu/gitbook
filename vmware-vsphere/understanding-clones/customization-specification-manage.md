---
icon: user-tie-hair
---

# Customization Specification Manage

Sanallaştırma ortamlarında otomasyonun kalbi olan şablonlar (Customization Specifications), sadece klonlama sihirbazı çalışırken aceleyle oluşturulup bırakılacak geçici dosyalar değildir. Kurumsal bir altyapıda bu "Cevap Anahtarlarının" merkezi bir noktadan yönetilmesi, yedeklenmesi ve farklı projeler için hızla çoğaltılması gerekir.

Klonlama sihirbazından tamamen bağımsız olarak çalışan, vCenter'ın gizli kahramanlarından biri de Customization Specification Manager ekranıdır.

**1. Klonlama Sihirbazı vs. Merkezi Yönetim Ekranı**

Sanal makine klonlanırken sihirbazın içinden yeni bir şablon oluşturmanın en büyük kısıtlaması şudur: Hangi makineyi klonluyorsanız (Örn: Windows), sistem hedef işletim sistemini ona kilitler.

vSphere arayüzünün ana menüsünden (Policies and Profiles altından) erişilen Customization Specification Manager ise tamamen özgür bir alandır. Herhangi bir klonlama işlemi başlatmadan, dilediğiniz zaman buraya girip yepyeni şablonlar hazırlayabilirsiniz. Üstelik bu ekranda "Target OS" (Hedef İşletim Sistemi) kilitli değildir; tek bir tıkla ister Linux ister Windows şablonu oluşturmaya başlayabilirsiniz. Bu yapı, altyapı hazırlıklarını ihtiyaç doğmadan önce yapmanıza olanak tanır.

**2. Yöneticinin Takım Çantası: Düzenle, Çoğalt, Dışa Aktar**

Merkezi yönetim ekranı, oluşturduğunuz tüm Windows ve Linux şablonlarını tek bir listede toplar ve sistem yöneticisine üç hayati yetenek sunar:

* Edit: Mevcut bir şablonda küçük bir değişiklik yapmanız gerektiğinde (örneğin Active Directory'ye makine alma yetkisine sahip servisin şifresi değiştiğinde), yeni bir şablon oluşturmak yerine mevcudu anında güncelleyebilirsiniz.
* Duplicate: Şablon mimarisinin en çok zaman kazandıran özelliğidir. Elinizde her ayarı kusursuz olan bir "Web\_Server\_Sablonu" var ama size bunun sadece saat dilimi veya farklı bir Local Admin şifresi kullanan versiyonu lazım. Sıfırdan şablon yaratmak yerine, mevcut olanı `Duplicate` ile kopyalar, adını "Database\_Server\_Sablonu" yapar ve sadece gerekli yeri değiştirerek saniyeler içinde yeni bir standart oluşturursunuz.
* Export: Mimarinin en çarpıcı özelliklerinden biridir. Hazırladığınız bu değerli kural setlerini bir XML dosyası olarak bilgisayarınıza indirebilirsiniz.

**3. Ufuk Açıcı Mimari: XML İhracatı Neden Bu Kadar Kritik?**

Export özelliği, aslında çok merkezli veri merkezleri ve Disaster Recovery - DR senaryoları için tasarlanmış stratejik bir fonksiyondur.

Bu XML dosyaları bize şu ufuk açıcı mimari yetenekleri sağlar:

* Fiziksel İzolasyonları Aşmak: Diyelim ki şirketinizin birbiriyle hiçbir ağ bağlantısı olmayan (tamamen izole/Air-gapped) iki farklı vCenter ortamı var. Bir tarafta saatlerce uğraşıp mükemmel şablonlar hazırladınız. Bunları XML olarak dışa aktarıp, diğer izole vCenter'a `Import` ederek (İçeri Aktararak) tüm standartlarınızı anında o ortama taşıyabilirsiniz.
* Altyapı Yedekliliği ve Versiyon Kontrolü: Şablonlar, sunucuların ağa nasıl çıkacağını belirleyen kritik konfigürasyonlardır. Bu XML dosyalarını düzenli olarak dışa aktarıp kurumunuzun kod depolarında saklayabilirsiniz. Böylece vCenter'da biri yanlışlıkla bir şablonu siler veya bozarsa, elinizdeki XML dosyasından anında eski haline döndürebilirsiniz.
* IAC Hazırlığı: VMware PowerCLI veya Terraform gibi otomasyon araçları arka planda vCenter ile konuşurken tam olarak bu XML veri yapılarını kullanır. Şablonlarınızı XML'e dökmek, gelecekteki otomasyon scriptlerinizin referans dosyalarını oluşturmak anlamına gelir.

Özetle; Customization Specification Manager, sadece bir liste ekranı değil; sanallaştırma ortamınızdaki dağıtım kurallarının yazıldığı, yedeklendiği ve farklı lokasyonlara taşınabildiği "Merkezi Dağıtım Beyni"dir. Kurumsal standartların korunması için bu ekranın efektif kullanımı şarttır.
