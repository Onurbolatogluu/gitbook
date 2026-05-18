---
icon: person-running
---

# Run and Complete The Process of Cloning a VM

Bir önceki aşamada "Customization Specification" (Özelleştirme Şablonu) mimarisini kusursuz bir şekilde kurguladıktan sonra, sıra klonlama sihirbazını tamamlayıp vCenter'a "İşi Başlat" (Finish) komutunu vermeye gelir.

Bu makalede, klonlama işleminin son adımında şablonun nasıl devreye girdiğini, laboratuvar ortamlarında sık karşılaşılan "Özelleştirme Hatasının" (Customization Error) kaputun altındaki nedenini ve üretim (Production) ortamlarında felakete yol açabilecek performans tuzaklarını inceleyeceğiz.

**1. Dinamik "Cevap Anahtarı"nın Devreye Girmesi**

Klonlama sihirbazında donanım (Hardware) ayarlarını geçip şablon seçimi ekranına geldiğinizde, vCenter'ın mimari zekası kendini gösterir.

Daha önce şablonu oluştururken ağ ve isim ayarlarını "Prompt" (Kullanıcıya Sor) olarak yapılandırdığımızı hatırlayın. Sihirbazda "Next" butonuna tıkladığınızda sistem doğrudan son özet ekranına geçmez. Şablonun içindeki o esnek kural devreye girer ve vCenter arayüzü sizi durdurarak şu bilgileri manuel olarak girmenizi ister:

* Computer Name (Bilgisayar Adı): Örneğin, `DB_New_Service`.
* IP Address: Örneğin, `192.168.1.144`.

Aynı şablonu bir sonraki klonlamada kullandığınızda, sistem sizden yine farklı bir IP isteyecektir. Bu yapı, tek bir şablonla IP çakışması (Conflict) riskini sıfıra indirerek sonsuz sayıda yeni sunucu dağıtımı (Deploy) yapmanızı sağlayan sistem mühendisliği pratiğidir.

**2. Kritik Bir Hata: "Customization Not Supported" Uyarısı Neden Çıkar?**

Özet ekranını (Summary) onaylayıp işlemi başlattığınızda, vCenter alt kısımdaki görev çubuğunda (Recent Tasks) kopyalama işlemini başlatır. Ancak bazen, özellikle test ve laboratuvar ortamlarında, işlem biter bitmez kırmızı bir hata fırlatır: "Customization of the guest operating system is not supported in this configuration."

Bu hatanın çıkması sanal makinenin klonlanamadığı anlamına gelmez; makine arka planda başarıyla kopyalanmış ve Datastore'a yazılmıştır. Hata, Sysprep (Özelleştirme) işleminin yapılamadığını belirtir.

Bunun kaputun altındaki yegane sebebi şudur:

Kaynak sanal makinenin içi boştur (İşletim sistemi yoktur) veya VMware Tools yüklü değildir.

vCenter, diski kopyaladıktan sonra "Şimdi şu IP'yi ve Ismi içeri enjekte edeyim" diyerek makinenin içine girmeye çalışır. Ancak karşıda komutu alacak bir Windows veya VMware Tools bulamadığı için bu özelleştirme işlemini iptal eder ve uyarı verir. Bu durum, önceki yazılarımızda bahsettiğimiz "VMware Tools bağımlılığı" kuralının canlı bir kanıtıdır.

**3. Ağ (Network) ve I/O Yükü**

Klonlama işlemi, kâğıt üzerinde basit bir kopyala-yapıştır gibi görünse de, fiziksel altyapı (Infrastructure) üzerinde devasa bir strese neden olur.

Örneğin, içinde aktif bir veritabanı barındıran ve diski 60 GB dolu olan bir sanal makineyi, Host-A'dan Host-B'ye klonladığınızı düşünelim. vCenter bu komutu verdiğinde, arka planda Storage (Depolama) ağı üzerinden 60 GB'lık devasa bir veri bloğu bir Host'tan diğerine akmaya başlar.

* Storage I/O Tüketimi: Disklerin okuma/yazma kapasiteleri limitlere dayanır.
* Ağ (Network) Satürasyonu: Veri kopyalama trafiği, sanal makinelerin haberleştiği fiziksel Switch'leri (özellikle vMotion/Provisioning ağları izole edilmemişse) felç edebilir.

Eğer bu 60 GB'lık (veya daha büyük) klonlama işlemini Mesai Saatleri içinde yaparsanız, ağdaki tüm bant genişliğini (Bandwidth) sömürürsünüz. Ortamdaki diğer ERP yazılımları, dosya sunucuları veya kullanıcı veritabanı sorguları anında yavaşlamaya, gecikme (Latency) süreleri fırlamaya başlar.

**4. Ufuk Açıcı Çözüm: Scheduled Tasks (Zamanlanmış Görevler)**

İşte bu performans dar boğazını engellemek için kurumsal altyapılarda klonlama operasyonları asla manuel olarak mesai saatlerinde tetiklenmez. VMware, bu operasyonel yükü yönetebilmek için "Scheduled Tasks" (Zamanlanmış Görevler) mimarisini sunar.

Sistem yöneticisi gündüz mesaisinde klonlama sihirbazını baştan sona hazırlar; kaynak makineyi, hedef Host'u, Datastore'u ve Customization Spec şablonunu seçer. Ancak "Finish" demek yerine bu işlemi bir Scheduled Task olarak kaydeder ve zamanlayıcıyı Cumartesi gecesi saat 02:00'ye ayarlar.

Hafta sonu geldiğinde, ortamda hiçbir kullanıcı yokken ve ağ trafiği sıfırken, vCenter otomatik olarak uyanır. O devasa 60 GB'lık kopyalama işlemini rahat rahat, ağa hiçbir zarar vermeden tamamlar, yeni makinenin Sysprep işlemlerini bitirir ve Pazartesi sabahı sistem yöneticisine hazır, yepyeni bir sunucu teslim eder.

Özetle; Klonlama, teknik olarak birkaç tıklama ile başlasa da; ağ yükünü planlamak, kimliklendirme şablonlarını doğru anlarda devreye sokmak ve operasyonu mesai dışına kaydırmak gerçek bir sistem yöneticisinin vizyonunu belirler.
