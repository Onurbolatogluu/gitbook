---
icon: folder-tree
---

# Files That Make Up A Virtual Machine

Sanallaştırma dünyasına adım attığımızda, oluşturduğumuz sanal makineleri (VM) fiziksel birer bilgisayar kasası gibi hayal etme eğiliminde oluruz. Ekranda çalışan bir işletim sistemi, dönen fanlar ve yanan ışıklar varmış gibi hissederiz. Ancak ESXi (Hypervisor) mimarisinin kaputunu kaldırıp depolama alanına (Datastore) baktığımızda, fiziksel gerçekliğin çok daha farklı olduğunu görürüz.

Bir sanal makine, özünde Datastore üzerinde oluşturulmuş bir klasörden ve bu klasörün içindeki belirli uzantılara sahip bir grup dosyadan ibarettir.

Bir sistem mühendisi olarak bu dosyaların ne işe yaradığını bilmek; yedekleme (Backup), felaket kurtarma (Disaster Recovery), performans analizi ve sorun giderme (Troubleshooting) süreçlerinde size "karanlıkta görebilme" yeteneği kazandırır. Gelin, bir sanal makinenin klasörüne girdiğinizde karşınıza çıkacak olan tüm o karmaşık dosyaları, görevlerine göre kategorize ederek deşifre edelim.

***

#### 🏛️ 1. Temel Yapıtaşları

Bir sanal makinenin var olabilmesi ve verilerini tutabilmesi için bu klasörde mutlaka bulunması gereken temel dosyalar şunlardır:

* `.vmx` (Yapılandırma Dosyası): Sanal makinenin "Anakartı" ve "Beyni"dir. Tamamen metin tabanlı olan bu dosya; makineye kaç vCPU atandığı, ne kadar RAM verildiği, ağ kartının MAC adresi ve diğer tüm sanal donanım limitlerini barındırır. Makine her açıldığında ESXi bu dosyayı okur ve sistemi ona göre inşa eder.
* `.vmdk` (Disk Descriptor / Üst Veri Dosyası): Genellikle çok küçük boyutlu bir metin dosyasıdır. Sanal diskin geometrisi (Thick/Thin olduğu) ve gerçek verinin nerede durduğu hakkında donanıma yol gösteren bir harita (metadata) görevi görür.
* `-flat.vmdk` (Asıl Disk Verisi): İşte gerçek verilerinizin, işletim sisteminizin ve o devasa "1 ve 0'ların" yazıldığı asıl dosya budur. vSphere arayüzü kafa karışıklığını önlemek için `.vmdk` ve `-flat.vmdk` dosyalarını birleştirip tek bir disk gibi gösterse de, arka planda bu ayrım mevcuttur.

***

#### ⚡ 2. Çalışma Zamanı ve Runtime Dosyaları

Sanal makine kapalıyken (Power Off) klasör oldukça sakindir. Ancak makineyi başlattığınız anda (Power On), ESXi klasörün içine dinamik olarak bazı yeni dosyalar yaratır.

* `.vswp` (Misafir İşletim Sistemi Takas Dosyası): Makine her açıldığında, o makineye atanan RAM miktarı kadar (Örn: 8 GB) Datastore üzerinde devasa bir takas dosyası oluşturulur. Eğer ESXi sunucusunun fiziksel RAM'i tamamen dolarsa, içerideki işletim sisteminin çökmemesi için veriler acil durum olarak bu diske yazılır.
* `vmx-*.vswp` (Hypervisor Ek Yük Takas Dosyası): ESXi, sanal makineyi (klavye, fare, ekran kartı vb.) yönetebilmek için kendisine küçük bir "Yönetim Odası" (VMX Süreci) kurar. Bu dosya, ana işletim sisteminin değil, doğrudan ESXi'ın o makineyi yönetirken kullandığı sürecin (\~50-100 MB) acil durum takas dosyasıdır.
* `.nvram` (Sanal BIOS/UEFI): Fiziksel bilgisayarlardaki BIOS pilinin görevini üstlenir. Sanal makinenin boot (ön yükleme) sırası ve BIOS/UEFI ayarları bu dosyanın içinde tutulur.
* `.vmss` (Suspend / Bekletme Dosyası): Makineyi kapatmak yerine "Suspend" (Duraklat) derseniz, o an RAM'de çalışan tüm veriler dondurularak bu dosyaya yazılır. Makine tekrar başlatıldığında (Resume), sistem bu dosyayı okuyarak kaldığı milisaniyeden çalışmaya devam eder.

***

#### 🛡️ 3. Güvenlik, Yedeklilik ve Loglama

ESXi, olası çökmelere ve aynı anda birden fazla yöneticinin sisteme müdahale etmesi riskine karşı kendi emniyet kemerlerini üretir.

* `.vmx.lck` (Kilit Dosyası): Boyutu genellikle `0 KB` olan bu dosya, makine çalıştığında veya ayarları düzenlenirken ortaya çıkar. İçinde veri yoktur, sadece bir "girilemez" tabelası görevi görür. Amacı, ağdaki diğer yöneticilerin bu makineye aynı anda müdahale edip sistemi bozmasını (Split-Brain) engellemektir.
* `.vmx~` (Yedek Yapılandırma Dosyası): Makinenin ayarlarını değiştirdiğinizde, ESXi mevcut sağlıklı `.vmx` dosyasının bir kopyasını anında `.vmx~` (Tilde) olarak yedekler. Ayar yapılırken sunucu çökerse, sistem bu yedekten hayatına sorunsuz devam eder.
* `.vmxf` (Ek XML Yapılandırması): `.vmx` dosyasının küçük kardeşidir. Makineniz daha büyük bir grubun (vApp) parçasıysa veya VMware Workstation gibi platformlar arası taşınacaksa, uyumluluk ve UUID bilgilerini saklar.
* `.log` (`vmware.log`): Sanal makinede meydana gelen tüm arka plan olaylarının yazıldığı seyir defteridir. Dosya boyutu diski şişirmesin diye genellikle 2 MB'a ulaştığında sistem tarafından dondurulur, numaralandırılır (`vmware-1.log`, `vmware-2.log` vb.) ve tertemiz yeni bir log dosyası açılır (Log Rotation).

***

#### 📸 4. Snapshot ve Yedekleme (Backup) Dosyaları

Sistemde bir değişiklik yapmadan önce Snapshot (Anlık Görüntü) aldığınızda veya profesyonel bir yedekleme yazılımı çalıştırdığınızda şu dosyalar devreye girer:

* `-delta.vmdk` (Değişim Diski): Snapshot alındığında asıl disk (`-flat.vmdk`) kilitlenir/okunur duruma geçer. O andan itibaren makineye yazılan tüm "yeni" veriler bu Delta dosyasına yazılır. `Snapshot hayatta kaldığı sürece asıl disk kış uykusundadır, tüm yükü ve değişimi Delta disk çeker.`
* `.vmsd` (Snapshot Veritabanı): Makineye ait kaç tane Snapshot olduğu, aralarındaki ilişki ve hangi dosyaların kime ait olduğunu tutan dizin (index) dosyasıdır.
* `.vmsn` (Snapshot State): Eğer Snapshot alırken "RAM içeriğini de kaydet" derseniz, makinenin o anki aktif bellek durumu bu dosyanın içine kopyalanır.
* `-ctk.vmdk` (Değişen Blok İzleme / CBT): Yedekleme yazılımlarının (Örn: Veeam) kahramanıdır. Bu "gece bekçisi" dosya, diske yazılan her yeni verinin blok adresini not alır. Yedekleme saati geldiğinde, sistem tüm diski taramak yerine sadece bu dosyadaki notlara bakar ve yalnızca değişen blokları kopyalayarak saatler sürecek işlemi saniyelere indirir (Incremental Backup). `CTK dosyası kesinlikle bir veri dosyası DEĞİLDİR. Sadece değişen verinin yerini (adresini) gösteren bir "Harita" dosyasıdır.`

***

#### 🚀 5. Yeni Nesil ve İleri Düzey Dosyalar

* `.scoreboard`: Makine çalışırken arka plandaki VMM (Virtual Machine Monitor) süreci ile işletim sistemi arasındaki RAM tüketimi ve işlemci iletişimini anlık olarak takip eden skor tabelasıdır.
* `.hlog` (vMotion Günlüğü): Sanal makineyi çalışır vaziyetteyken bir fiziksel sunucudan diğerine (vMotion) taşıdığınızda ortaya çıkar. Sistem, bu taşıma esnasında saniyelik veri kayıplarını önlemek için geçici olarak bu günlüğü kullanır.

Sanal makinenin Datastore üzerindeki bu dosyalarını tanımak, ekranda gördüğünüz arayüzlerin ötesine geçmenizi sağlar. Artık kilitli kalan bir makineyi nasıl açacağınızı (`.lck` dosyasını silerek) veya yedekleme mimarinizin neden hızlı çalıştığını (`-ctk.vmdk` sayesinde) kaputun altındaki mühendislik seviyesinde biliyorsunuz.
