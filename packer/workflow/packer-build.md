---
icon: building
---

# packer build

#### 1. İnşa Süreci ve Yaşam Döngüsü (Execution Engine Lifecycle)

`packer build` komutu tetiklendiğinde, arka planda katı bir disipline sahip operasyonel bir süreç (Pipeline) başlatılır:

* Instantiation: Hedef platformun (AWS, Azure, vSphere vb.) API'leri kullanılarak geçici bir sanal sunucu ayağa kaldırılır.
* Provisioning: Oluşturulan sunucuya güvenli kanallar (SSH veya WinRM) üzerinden erişim sağlanır. Şablonda tanımlanan komutlar veya konfigürasyon yönetim araçları (Ansible, Shell vb.) çalıştırılarak sunucu özelleştirilir.
* Registration & Cleanup: İşlem tamamlandığında, sunucunun o anki durumu bir Makine İmajı (AMI, VM Template) olarak tescil edilir. Ardından, kullanılan geçici sunucu ve güvenlik anahtarları kalıcı olarak silinir.

#### 2. Immutable Infrastructure Paradigm

* Golden Image Konsepti: Uygulama kodu, kütüphaneler ve sistem ayarları imajın içine eklenir (Baking).
* Drift Önleme: Sunucular canlıya alındıktan sonra üzerlerinde manuel değişiklik yapılmaz. Bu sayede, "Configuration Drift" riski minimize edilir.
* Bir güncelleme gerektiğinde mevcut sunucu değiştirilmez `packer build` ile `v2` versiyonlu yeni bir imaj üretilir ve eski sunucular yenileriyle değiştirilir (Rolling Update / Blue-Green Deployment).

#### 3. Otomasyon ve CI/CD Entegrasyonu

`packer build`,  CI/CD pipeline'larının tetikleyici unsurudur.

* GitHub Actions veya Jenkins gibi servisler, kod deposundaki bir değişikliği algıladığında bu komutu insan müdahalesi olmadan çalıştırır.
* Üretilen imajlar, yayına alınmadan önce otomatik testlerden geçirilerek operasyonel kararlılık sağlanır.

#### 4. Troubleshooting

* (`-debug`): Süreci adım adım ilerleterek, çalışanlara çalışan geçici sunucuya erişim ve yerinde inceleme imkanı tanır.
* (`PACKER_LOG=1`): Arka plandaki API çağrılarını ve sistem olaylarını detaylı olarak raporlayarak kök neden analizini kolaylaştırır.
* (`-on-error`): Hata durumunda sistemin temizlik yapmasını engelleyerek (`ask`), adli inceleme (Forensics) yapılmasına olanak sağlar.

#### 5. Mimari ve Kaynak Yönetimi

* Packer'ın çoklu işlem (Multi-process) mimarisi nedeniyle, işletim sistemi seviyesindeki dosya tanımlayıcı (File Descriptor) limitlerine dikkat edilmelidir.
* Bulut ortamlarında sunucunun tam olarak hazır hale gelmesi (Cloud-init sürecinin tamamlanması) ile Provisioning adımının başlaması arasındaki zamanlama farkları yönetilmelidir.

***

Özetle: `packer build`; tanımlanan altyapı kodunu, üretim ortamına (Production) hazır, versiyonlanmış ve standartlaştırılmış sistem yapıtlarına dönüştüren temel üretim fonksiyonudur.
