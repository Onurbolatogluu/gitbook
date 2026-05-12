---
icon: cloudsmith
---

# Understanding Clones

Clone işlemi, halihazırda çalışan veya kapalı durumdaki bir Virtual Machine'in (VM) işletim sistemi, kurulu uygulamaları, sanal diskleri (VMDK) ve donanım konfigürasyonuyla birlikte birebir bağımsız bir kopyasının çıkarılmasıdır.

Bu mimari, sistem yöneticilerine manuel kurulum süreçlerini atlayarak dakikalar içinde yeni sunucular devreye alma imkanı sunar.

**1. Orkestrasyonun Şartı: vCenter Server Zorunluluğu**

Bilinmesi gereken ilk kural şudur: Bağımsız bir ESXi Host arayüzü üzerinden doğrudan bir Virtual Machine Clone'u alınamaz. ESXi, kendi başına sadece donanımı sanallaştıran ve makineleri çalıştıran bir hipervizördür. Bir makinenin disklerini başka bir alana kopyalamak, yeni donanım kimlikleri (MAC adresi, UUID) üretmek ve bunu ağda çakışma yaratmadan sisteme entegre etmek bir "orkestrasyon" işidir. Bu nedenle Clone mimarisi, ortamda yalnızca bir vCenter Server bulunduğunda kilidi açılan, Enterprise seviyesinde bir özelliktir.

**2. Stratejik Kullanım: Hızlı Dağıtım (Rapid Provisioning)**

Clone işleminin en yaygın ve en güçlü kullanım amacı, zamandan tasarruf etmektir.

Örneğin, altyapınıza yeni bir SQL Server eklemeniz gerektiğini düşünelim. Geleneksel yöntemde sıfırdan bir işletim sistemi kurmak, güncellemeleri (Update) çekmek, SQL Server uygulamasını yüklemek ve güvenlik ayarlarını yapmak saatler sürer.

Bunun yerine, ortamda zaten kusursuz çalışan bir SQL Virtual Machine'i hedef Host üzerine Clone'layarak, dakikalar içinde tamamen hazır bir sunucu elde edersiniz. Yeni makine üzerinde sadece IP adresi veya sunucu ismi (Hostname) gibi ufak konfigürasyon değişiklikleri yaparak sistemi anında canlıya (Production) alabilirsiniz.

**3. Alternatif Bir Strateji: Backup (Yedekleme) Olarak Clone**

Sistem yöneticileri, yüksek trafik alan ve kritik veriler barındıran sunucuları korumak için Clone özelliğini bir "felaket kurtarma" (Disaster Recovery) veya periyodik Backup stratejisi olarak da kullanabilirler.

Örneğin, her gün yoğun veri yazılan bir veritabanı sunucusunun her 3-4 günde bir tam bir Clone'u alınarak farklı bir fiziksel sunucuya veya harici bir Datastore'a aktarılabilir. Orijinal Virtual Machine'de kritik bir çökme veya veri kaybı yaşanırsa, saatler süren onarım süreçleriyle uğraşmak yerine, birkaç gün öncesine ait bu sağlam Clone makinesi anında "Power On" yapılarak hizmetin kesintisiz devam etmesi sağlanabilir.

**💡 Ekstra Mühendislik Perspektifi: Clone vs. Snapshot Farkı**

Clone işlemini bir Backup stratejisi olarak kullanırken, sistem mimarisindeki şu önemli farkı (Snapshot ile olan farkını) iyi kavramak gerekir:

* Bağımsızlık (Independence): Snapshot, orijinal makinenin o anki durumunu donduran ve orijinal diske (_Base Disk_) bağımlı olan geçici bir delta dosyasıdır. Orijinal disk bozulursa Snapshot çöp olur. Clone ise orijinal makineden tamamen bağımsız, kendi `.vmx` ve `.vmdk` dosyaları olan kalıcı bir makinedir. Orijinal makine tamamen silinse bile Clone makinesi kusursuz çalışmaya devam eder.
* Depolama Maliyeti (Storage Cost): Clone işlemi, orijinal makinenin diski ne kadarsa (Örneğin 100 GB), hedef Datastore'da da o kadar ekstra tam yer kaplar. Bu nedenle manuel Clone işlemleri güvenli bir Backup yöntemi olsa da, ortamdaki Storage (Depolama) kapasitesini çok hızlı tüketir. (Kurumsal yapılarda bu depolama maliyetini optimize etmek için genellikle Veeam gibi üçüncü parti, _Deduplication_ yapabilen özel Backup yazılımları tercih edilir).
