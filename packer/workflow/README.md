---
icon: house-laptop
---

# Workflow

#### 1. Temel Operasyonel İş Akışı

Packer'ın çalışma prensibi, ham bir işletim sistemini alıp nihai bir ürün haline getiren üç aşamalı bir üretim bandından oluşur:

* Build (İnşa): Sürecin başlatılması ve kaynak tahsisidir. Hedef platformda (AWS, VMware vb.) temel işletim sistemine sahip geçici bir sanal sunucu (Instance) ayağa kaldırılır.
* Provision (Konfigürasyon): Çalışan sunucunun özelleştirildiği aşamadır. Sistem yöneticisi tarafından belirlenen kurallar (Shell scriptleri, Ansible playbook'ları vb.) işletilerek gerekli uygulamalar ve yamalar yüklenir.
* Post-Process (Son İşlem/Teslimat): Üretilen çıktının (Artifact) paketlenmesi ve dağıtımıdır. Oluşturulan imaj sıkıştırılabilir, etiketlenebilir veya ilgili kayıt defterine (Registry) yüklenebilir.

#### 2. Yönetim Komutları

Kod tabanlı altyapı (Infrastructure as Code) süreçlerinde kullanılan temel komut setleri şunlardır:

* `packer init`: Projenin ihtiyaç duyduğu bağımlılıkların (Plugin/Eklenti) ve kütüphanelerin yerel ortama indirilmesini sağlar. (Yazılım dünyasındaki `npm install` veya `pip install` mantığıdır).
* `packer validate`: Kodun sözdizimi (Syntax) ve mantıksal bütünlüğünü kontrol eder. Kod çalıştırılmadan önce yapılan statik bir analizdir.
* `packer build`: Tanımlanan şablonun icra edildiği ve imaj üretim sürecinin tetiklendiği asıl komuttur.

#### 3. Bulut Ortamında İcra Mantığı

Özellikle AWS gibi bulut ortamlarında süreç, "Geçici Kaynak Yönetimi" prensibiyle ilerler:

1. Kaynak Tahsisi: Packer, hedef platformda geçici bir sunucu ve bu sunucuya erişim için geçici kimlik bilgileri (SSH Key/Key Pair) oluşturur.
2. Konfigürasyon: Provisioner'lar sunucuya bağlanır ve tanımlı görevleri icra eder.
3. Snapshot ve Temizlik: İşlem tamamlandığında, sunucunun o anki durumu bir imaj (AMI) olarak kaydedilir. Ardından Packer, oluşturduğu geçici sunucuyu ve güvenlik anahtarlarını imha eder. Bu, güvenlik ve maliyet yönetimi açısından kritik bir adımdır.

#### 4. Değişmez Altyapı Stratejisi

Bu yaklaşım, mevcut sistemlerin güncellenmesi yerine, her güncellemede sistemin yeniden üretilmesini esas alır.

* Versiyonlama (Versioning): `v1` versiyonlu bir sunucu üzerinde yama (patch) yapılmaz.
* Yeniden Üretim (Replacement): Bir güncelleme gerektiğinde, kaynak kod revize edilir ve `v2` etiketiyle tamamen yeni bir imaj üretilir.
* Devreye Alma (Rollout): `v2` imajından üretilen sunucular canlıya alınır, `v1` sunucuları trafikten çekilerek imha edilir. Bu yöntem, sistem kararlılığını maksimize eder.

#### 5. CI/CD Entegrasyonu ve Standardizasyon

Packer, modern DevOps süreçlerinde Sürekli Entegrasyon/Sürekli Dağıtım (CI/CD) Pipeline'ların bir parçasıdır.

* Otomasyon: GitHub Actions veya Jenkins gibi araçlar, kod deposundaki bir değişikliği algılayarak `packer build` sürecini otomatik tetikler.
* Configuration Drift'in Önlenmesi: Manuel müdahale olmadığı için, Prod ortamındaki sunucuların konfigürasyonunun zamanla standarttan sapması (Drift) engellenmiş olur. Her sunucu, onaylanmış imajın birebir kopyasıdır.

***

Özetle: Packer İş Akışı; altyapı tanımlarının (Kod), standartlaştırılmış, test edilebilir ve versiyonlanabilir sistem imajlarına (Artifacts) dönüştürüldüğü, insan hatasını minimize eden endüstriyel bir otomasyon sürecidir.
