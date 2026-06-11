---
icon: clipboard-list-check
---

# Other Scheduler Task Available

Önceki makalemizde, ağır ağ ve depolama (I/O) yükü getiren klonlama işlemlerini vCenter'ın "Zamanlanmış Görevler" (Scheduled Tasks) özelliği sayesinde mesai dışı saatlere nasıl kaydırabileceğimizi incelemiştik. Ancak vCenter'ın otomasyon yetenekleri sadece klonlama ile sınırlı değildir.

vCenter üzerinde zamanlayabileceğiniz görevlerin türü, hiyerarşik olarak tam olarak hangi nesnenin (Sanal Makine, Host, Datacenter veya vCenter Root) üzerinde bulunduğunuza göre tamamen değişir.

**1. Sanal Makine (VM) Seviyesi**

vCenter'da en zengin otomasyon seçenekleri doğal olarak en uç nokta olan Sanal Makineler seviyesinde bulunur. Bir sanal makineyi seçip `Configure > Scheduled Tasks` menüsüne girdiğinizde, doğrudan o işletim sistemine ve donanıma müdahale eden şu görevleri zamanlayabilirsiniz:

* Güç Yönetimi: Power On, Shut Down Guest OS, Restart, Power Off, Suspend ve Reset. Örneğin; hafıza sızıntısı (Memory Leak) sorunu yaşatan eski bir uygulamanın bulunduğu sunucuyu, kullanıcıları etkilememesi için her pazar gecesi saat 03:00'te otomatik olarak yeniden başlatmak (Restart Guest OS) için bu menü kullanılır.
* Kaynak Yönetimi: Mesai saatlerinde daha fazla RAM veya CPU gücüne ihtiyaç duyan bir makinenin kaynak limitlerini belirli bir saatte artırıp, mesai bitiminde tekrar kısmak için kullanılabilir.
* Taşıma ve Klonlama: Klonlama işlemini daha önce detaylandırmıştık. Migrate (vMotion) ise, donanım bakımı yapılacak bir ESXi host üzerindeki kritik makinelerin gece yarısı otomatik olarak başka bir host'a taşınması için zamanlanabilir.
* Anlık Snapshot: Sistem yöneticilerinin sık kullandığı bir fonksiyondur.

> 💡 Uzman İpucu: Snapshot Bir "Yedekleme" (Backup) Stratejisi Değildir
>
> Snapshot, veritabanı veya işletim sistemi yama (Patch) geçişleri öncesinde alınması gereken, işlem başarılı olursa derhal silinmesi (Consolidation) gereken geçici bir geri dönüş noktasıdır. Bir sanal makine üzerinde günlerce Snapshot tutmak, diskin Delta dosyalarını devasa boyutlara ulaştırır, Storage performansını ciddi şekilde düşürür ve makinenin donmasına neden olabilir. Veri güvenliği ve yedekleme (Backup) için vCenter Snapshot'ları değil; donanım seviyesinde çalışan profesyonel yedekleme yazılımları kullanılmalıdır.

**2. ESXi Host Seviyesi**

Hiyerarşide bir üst kademeye çıkıp bir ESXi Host'u seçtiğinizde, Scheduled Tasks menüsünün dramatik bir şekilde daraldığını görürsünüz. Bir Host'u "klonlayamazsınız" veya "taşıyamazsınız".

Burada karşınıza çıkan yegane temel seçenek "New Virtual Machine" (Yeni Sanal Makine Oluştur) görevidir. Eğer ortamınıza yeni bir sunucu eklemeniz gerekiyorsa ve bu makinenin kurulum süreçlerinin (örneğin işletim sisteminin ISO'dan boot edilip ağa dahil olması) mesai saati sonunda, sizin belirlediğiniz bir saatte başlamasını istiyorsanız, bu görevi doğrudan ilgili Host üzerinden zamanlayabilirsiniz.

**3. Datacenter Seviyesi**

Datacenter seviyesine çıktığınızda, vCenter size doğrudan fiziksel altyapıyı büyütmeye yönelik iki temel otomasyon seçeneği sunar:

* Yeni Sanal Makine Oluştur (New Virtual Machine)
* Host Ekle (Add Host): Ortama yeni bir fiziksel sunucu (ESXi) ekleneceği zaman bu işlem zamanlanabilir.

Ancak burada vCenter'ın koruma mimarisi devreye girer: Sistem, doğrulamadığı hiçbir işi zamanlamaz. Yeni bir Host ekleme görevini zamanlamaya çalıştığınızda vCenter sizden IP adresi, kullanıcı adı ve parola (Root credentials) ister ve arka planda o sunucuya erişip erişemediğini test eder (Pre-validation). Eğer fiziksel sunucu ağda yoksa veya şifre yanlışsa, vCenter bu görevi takvime kaydetmeyi kesinlikle reddeder. Bu, gece yarısı çalışacak bir otomasyon scriptinin "Bağlantı Hatası" verip tüm süreci kilitlemesini önleyen harika bir kontrol mekanizmasıdır.

**4. vCenter (Root) Seviyesi: Merkezi İzleme Paneli (Dashboard)**

Envanterin en üst seviyesi olan vCenter Root dizininde Scheduled Tasks menüsüne girdiğinizde, yeni bir görev oluşturma butonunun (Create Task) olmadığını fark edersiniz.

Bunun mimari nedeni, vCenter'ın kendisinin bir işlem nesnesi olmamasıdır. Bu root seviyesindeki ekran, alt seviyelerdeki (Makineler, Hostlar, Datacenterlar) tüm zamanlanmış görevlerin toplandığı, sistem yöneticisine ortamdaki tüm otomasyon takvimini tek bir listede sunan merkezi bir izleme paneli (Dashboard) olarak işlev görür.

Özetle; vCenter üzerinde bir görevi zamanlamak (Schedule) istediğinizde, işleme doğru hiyerarşik seviyeden (Doğru nesneye sağ tıklayarak) başlamak zorundasınız. Zamanlanmış görevler, kurumsal standartlara ve doğru operasyonel pratiklere (Snapshot yönetimi gibi) sadık kalındığı sürece, BT operasyonlarını kusursuzlaştıran en güçlü araçlardan biridir.

***

#### 1. "Edit Resource Settings" Görevi Aslında Neyi Değiştirir?

vCenter arayüzünde bir makineye sağ tıklayıp _Scheduled Tasks > Edit Resource Settings_ dediğinizde, vCenter size makinenin o anki donanım ekranını açar ve "Hangi ayarları değiştirmek istiyorsun, saat kaçta uygulayayım?" diye sorar.

Burada teorik olarak CPU'yu 4'ten 8'e, RAM'i 8GB'dan 16GB'a çıkarabilirsiniz. Ve evet, bunun için tek bir Job (Görev) tanımlarsınız. Ancak burada devasa bir "AMA" vardır.

#### 2. Kritik Engel: Makine Açıkken Kaynak Değişir mi? (Hot-Add/Hot-Plug Kısıtlaması)

Bir sanal makinenin kaynaklarını (CPU veya RAM), makine çalışırken (Powered On) değiştirebilmeniz için o makinenin konfigürasyonunda (VM Options) "CPU Hot Plug" ve "Memory Hot Plug" özelliklerinin önceden aktif edilmiş (Enable) olması şarttır.

* Eğer Hot-Plug Kapalıysa (Varsayılan Durum): vCenter zamanlanmış görev saatinde çalışır, makinenin RAM'ini artırmaya çalışır, ancak makine açık olduğu için hata verir ve görev başarısız (Failed) olur.
* Eğer Hot-Plug Açıksa: vCenter zamanlanmış görev saatinde çalışır, makine açıkken RAM'i 8GB'dan 16GB'a anında (kesintisiz) yükseltir. Ancak...

#### 3. Asıl Sorun: Geriye Dönüş (Hot-Remove Desteklenmez!)

Senaryomuz şuydu: _"Mesai saatlerinde kaynakları artırıp, mesai bitiminde tekrar kısmak."_

* Sabah 08:00 için bir Görev (Job) yazdınız: RAM'i 16GB yap. (Hot-Plug açık olduğu için başarıyla çalıştı, RAM yükseldi).
* Akşam 18:00 için ikinci bir Görev (Job) yazdınız: RAM'i tekrar 8GB'a düşür.

İşte tam bu noktada sistem çöker! Çünkü VMware mimarisinde ve modern işletim sistemlerinde (Windows/Linux) "Hot-Add" (Canlı canlı ekleme) desteklenirken, "Hot-Remove" (Canlı canlı RAM veya CPU sökme) D-E-S-T-E-K-L-E-N-M-E-Z. İşletim sistemi RAM'i kullanmaya başlamıştır, o RAM'i çalışırken geri alamazsınız.

#### Peki Kurumsal Dünyada Bu Senaryo Nasıl Otomatize Ediliyor?

Madem canlı canlı kaynak düşüremiyoruz, o zaman bu döngüyü (Sabah yükselt, akşam düşür) tam otomatik hale getirmek için zincirleme görevler (Chained Jobs) veya PowerCLI scriptleri yazmamız gerekir. Süreç tam olarak şöyle planlanır:

Senaryo: Gece RAM'i Düşür, Sabah Tekrar Yükselt

1. Görev 1 (Akşam 18:00 - Kapatma): Makineyi güvenli bir şekilde kapat (Shut Down Guest OS).
2. Görev 2 (Akşam 18:05 - Kaynak Kısma): Makine kapalı olduğu için _Edit Resource Settings_ görevi çalışır ve RAM'i 8GB'a düşürür.
3. Görev 3 (Akşam 18:10 - Tekrar Açma): Makineyi gece çalışması için düşük kaynakla aç (Power On).
4. _(Gece bitti, sabah oldu)_
5. Görev 4 (Sabah 07:50 - Kapatma): Makineyi kapat.
6. Görev 5 (Sabah 07:55 - Kaynak Artırma): _Edit Resource Settings_ görevi ile RAM'i 16GB'a çıkar.
7. Görev 6 (Sabah 08:00 - Tekrar Açma): Makineyi mesai için yüksek kaynakla aç (Power On).

#### Özetle;

Evet, kaynak düzenlemek için ayrı bir Job (Zamanlanmış Görev) yazmanız gerekir. vCenter bu işlemleri arayüz üzerinden yapmanıza izin verir ancak sanal makinenin çalışma prensipleri gereği kaynak azaltma işlemi makine açıkken yapılamaz. Bu nedenle gerçek bir otomasyon kurmak için makineyi kapatma, kaynağı değiştirme ve tekrar açma işlemlerini bir silsile (zincir) halinde planlamanız veya tüm bunları tek bir PowerShell (PowerCLI) betiği yazıp Windows Görev Zamanlayıcısına (Task Scheduler) bırakmanız gerekir.
