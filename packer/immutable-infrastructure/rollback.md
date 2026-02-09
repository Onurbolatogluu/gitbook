---
icon: webhook
---

# Rollback

Geleneksel In-place Fix yaklaşımının aksine, Packer mimarisinde geri dönüş işlemi Versiyonlanmış Yapıtlar üzerinden yürütülür.

* Revert vs. Redeploy: Hatalı bir sürüm (`v2`) tespit edildiğinde, sunucu içine girilip ayarlar geri alınmaz veya paketler kaldırılmaz.
* Altyapı Orkestrasyon aracı (Terraform, CloudFormation), bir önceki kararlı sürümün (`v1`) İmaj ID'sini (AMI ID) referans alacak şekilde güncellenir. Sistem, bilinen son duruma anında geri döner.

#### 2. Yan Yana Deploy Stratejisi

* Yeni sürüm (`v2`) sunucuları, mevcut çalışan (`v1`) sunucuların yanına kurulur. Trafik hemen kesilmez.
* `v2` sunucuları üzerinde otomatik Health Checks ve Smoke Test süreçleri işletilir.
* Eğer `v2` testleri geçemezse, trafik `v1` üzerinde kalmaya devam eder ve `v2` sunucuları imha edilir. Bu sayede Production ortamı herhangi bir kesinti yaşamaz.

#### 3. Sapma Yönetimi

* Sunucular üzerinde manuel müdahale önerilmediği için, `v1` imajına dönüldüğünde sistemin durumu %100 öngörülebilirdir.
* "Acaba eski versiyona dönerken veritabanı bağlantı kütüphanesi uyumsuzluk yaratır mı?" gibi endişeler taşınmaz. Çünkü İmaj, tüm bağımlılıklarıyla birlikte paketlenmiş statik bir bütündür.

***

* `packer build -debug` ve `PACKER_LOG=1` gibi araçlar, imajın içerisine hatalı bir konfigürasyonun girmesini engeller.
* Yazılım hataları (Bugs) veya konfigürasyon eksiklikleri, sunucu canlıya alındıktan sonra (Run-Time) değil, imaj derlenirken tespit edilir.
