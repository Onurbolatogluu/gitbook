---
icon: square-half-stroke
---

# Clone A Virtual Machine To A Template

Önceki yazılarımızda şablon (Template) mimarisinin sistem yöneticilerine nasıl saatler kazandırdığını ve kurumsal altyapılarda kurulum standartlarını nasıl koruduğunu teorik olarak incelemiştik. Şimdi bu mimariyi pratiğe dökme ve tüm temel ayarları yapılmış, içine işletim sistemi ve güncellemeleri kurulmuş bir sanal makineyi kalıcı bir "Altın İmaj"a (Master Image) dönüştürme sürecini adım adım inceleyeceğiz.

Bu makalede, `Clone to Template`  işleminin detaylarını, vCenter arayüzündeki kırılımlarını ve kaputun altındaki dosya yapısı değişikliklerini ele alacağız.

**1. İşlem Adımları: Klonlama Sihirbazı Nasıl Çalışır?**

İçerisinde sadece işletim sistemi (Örneğin: Windows Server 2012 R2), güvenlik duvarı ayarları, kurumsal yazılımlar ve güncellemeler bulunan, henüz hiçbir spesifik rol (SQL, IIS vb.) yüklenmemiş "temiz" bir sanal makineyi hazırladıktan sonra süreç şu şekilde ilerler:

1. İşlemi Başlatma: İlgili sanal makineye sağ tıklayıp "Clone" menüsüne gelindiğinde, karşımıza iki ana seçenek çıkar. Standart klonlama yerine "Clone to Template" (Şablona Klonla) seçeneği ile ilerlenir.
2. İsimlendirme ve Lokasyon: Şablona açıklayıcı bir isim verilir (Örn: `Template-Win2012R2-Standart`). Bu isimlendirme, gelecekte yüzlerce şablon arasında doğru imajı bulmak için kritik önem taşır. Ardından bu şablonun saklanacağı Datacenter seçilir.
3. Compute ve Storage Seçimi: Şablonun barınacağı ESXi Host ve Datastore belirlenir. Bu aşamada tıpkı normal bir makine klonlar gibi disk formatını (Thick veya Thin Provision) değiştirme şansınız vardır. Şablonlar genellikle depolama alanında yer kazanmak için Thin Provision olarak kaydedilir.
4. Sihirbaz tamamlandığında vCenter arka planda disk kopyalama işlemini başlatır.

**2. Kafa Karıştıran Detay: "Klonlanan Şablon Nereye Kayboldu?"**

Sihirbaz bittiğinde ve %100 "Completed" uyarısını gördüğünüzde, vCenter'ın klasik "Hosts and Clusters" (Ana Bilgisayarlar ve Kümeler) görünümünde hazırladığınız bu yeni şablonu göremezsiniz. Sanallaştırma dünyasına yeni giren birçok kişinin _"Acaba işlem başarısız mı oldu?"_ diye düşünmesine yol açan durum tam olarak budur.

Şablonlar, doğaları gereği "Power On" (Çalıştırılabilir) yapılabilen aktif bilgisayarlar değildir. Onlar mühürlenmiş kalıplardır. Bu kalıpları görmek ve yönetmek için vCenter'ın sol üst köşesindeki menüden "VMs and Templates" (Sanal Makineler ve Şablonlar) görünümüne geçmeniz gerekir.

Bu görünüme geçtiğinizde şablonunuzu bulacaksınız. Üstelik vCenter arayüzünde normal sanal makinelerden görsel olarak da ayrılırlar; yanlarındaki ikon standart bir ekran/kasa yerine, üst üste binmiş kağıt/kalıp (Template) şeklindedir.

**3. Kaputun Altında Ne Değişiyor? (`.vmx` vs `.vmtx`)**

Bir sanal makineyi şablona klonladığınızda (veya mevcut makineyi doğrudan şablona dönüştürdüğünüzde) Datastore seviyesinde fiziksel olarak ne değişir?

Standart bir sanal makinenin donanım ayarlarını ve kimliğini tutan dosyanın uzantısı `.vmx` (Virtual Machine Configuration) dosyasıdır. Siz bir makineyi şablon yaptığınızda, vCenter bu dosyanın uzantısını `.vmtx` (Virtual Machine Template Configuration) olarak değiştirir.

İşte bu araya eklenen "t" harfi, vCenter mimarisi için kesin bir mühürdür:

* Bir makinenin uzantısı `.vmtx` olduğunda, vCenter bu nesneyi ESXi sunucusunun RAM'ine ve CPU'suna gönderip çalıştıramaz.
* "Power On" butonu tamamen devre dışı (Gri) kalır.
* Bu dosya sadece okuma (Read-Only) moduna geçer ve dışarıdan gelebilecek (virüs dahil) hiçbir değişikliğe izin vermez.

Özetle; Bir makineyi `Clone to Template` yöntemiyle kalıplaştırmak, referans imajı güvenli bir şekilde dondurmak anlamına gelir. Artık bu şablon; saniyeler içinde yeni, birbirinin birebir kopyası ve hatasız sanal makineler üretmek (`Deploy from Template`) için hazır bir fabrikaya dönüşmüştür. Orijinal sanal makine ise yoluna devam edebilir, üzerine SQL veya Domain Controller rolleri kurularak prod ortama dahil edilebilir.

