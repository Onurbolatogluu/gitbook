---
icon: box-isometric
---

# Docker

#### 1. Çift Yönlü Mimari Rolü

* Packer, Docker altyapısını bir "Oluşturucu" olarak kullanır. Temel bir imajdan (Base Image) başlayarak, tanımlanan provizyon adımlarını (Shell, Ansible vb.) uygular ve nihai bir Hedef İmaj üretir.
* Post-Processor: Üretilen Docker imajı (Artifact) üzerinde `docker-tag` , `docker-push` (Registry'ye yükleme) veya `docker-save` gibi operasyonlar bu aşamada yönetilir.

#### 2. Ortam Eşitliği ve Stratejik Uyumluluk

* Production için AWS AMI derlenirken, eş zamanlı olarak Development ortamı için aynı konfigürasyon setine sahip bir Docker imajı üretilebilir.
* Bu yaklaşım, altyapı kodunun (IaC) her iki ortamda da aynı davranışsal bütünlüğü sergilemesini garanti altına alır.

#### 3. Standartlaştırılmış İş Akışı

* Docker imajları, sadece `Dockerfile` komutlarıyla sınırlı kalmaz; Packer'ın sunduğu Ansible, Chef veya Shell provizyonerleri ile zenginleştirilebilir.
* Üretilen konteyner imajları, runtime'da değiştirilemez statik birimler olarak kabul edilir. Her deploy, yeni bir imaj sürümü gerektirir.

#### 4. Operasyonel Kısıtlamalar ve Sorun Giderme

Docker ile çalışırken, işletim sistemi seviyesindeki bazı teknik limitlere dikkat edilmelidir:

* Unix tabanlı sistemlerde, Docker ile iletişim kurulan soket dosyasının yolu (Path) belirli bir karakter sınırını aşarsa "Failed to initialize build" hatası alınabilir.
  * _Çözüm:_ `TMPDIR` ortam değişkeni, daha kısa bir dizine (Örn: `/tmp`) yönlendirilerek soket yolunun kısalması sağlanır.
* Packer'ın her eklenti (Plugin) için ayrı bir süreç (Process) başlatması, yoğun paralel işlemlerde dosya tanımlayıcı (File Descriptor) limitlerinin (`ulimit -Sn`) aşılmasına neden olabilir. Sistem kaynaklarının bu yükü karşılayacak şekilde yapılandırılması gerekir.

***
