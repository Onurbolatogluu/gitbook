---
icon: image-landscape
---

# Versiyonlanmış İmajlar

#### 1. "Baking" ve Versiyon Stratejisi

Versiyonlanmış imajlar, işletim sistemi ve uygulama katmanlarının birleştirildiği birimlerdir. Packer, bu birimleri "Baking"yöntemiyle oluşturur.

* Bütünleşik Yapı: Kaynak kod, sistem kütüphaneleri, güvenlik yamaları ve konfigürasyon dosyaları derleme aşamasında imajın içine gömülür.
* Değiştirme Prensibi:
  * v1: İlk dağıtım için `v1` etiketli bir imaj üretilir ve sunucular bu "Altın İmaj" referans alınarak başlatılır.
  * v2: Bir güncelleme gerektiğinde (Örn: Güvenlik yaması veya yeni özellik), mevcut sunuculara müdahale edilmez. Şablon güncellenir ve tamamen yeni bir `v2` imajı derlenir. Eski sunucular imha edilir, yerlerine `v2` tabanlı sunucular geçer.

#### 2. Operasyonel İstikrar ve Sapma Yönetimi

Bu model, sistemlerin zamanla kararsızlaşmasını engelleyen en güçlü mekanizmadır.

* Sunucular yayına alındıktan sonra Read-Only gibi davranıldığı için, manuel müdahalelerden kaynaklanan "Configuration Drift" riski ortadan kalkar. Her sunucu, imajın matematiksel olarak kesin bir kopyasıdır.
* Yeni sürümde (`v2`) kritik bir hata tespit edilirse, sistem yöneticileri karmaşık geri alma senaryolarıyla uğraşmaz. Altyapı orkestrasyon aracı (Terraform vb.) önceki kararlı sürüm olan `v1` imaj ID'sine yönlendirilerek saniyeler içinde eski sürüme dönülür.

#### 3. CI/CD ve Otomasyon Entegrasyonu

* Continuous Delivery: Kodda veya ayarlarda en ufak bir değişiklik yaptığınızda, sistem otomatik olarak çalışmaya başlar. Sonuçta elinizde her zaman test edilmiş ve kullanıma hazır taze bir sunucu imajı olur.
* Terraform veya CloudFormation gibi araçlar, Packer'ın ürettiği o yeni Kimlik Numarasını (AMI ID) otomatik olarak algılar ve kullanır. Sizin elle kopyala-yapıştır yapmanıza gerek kalmaz.

#### 4. Çoklu Platform Desteği ve Ortam Eşitliği

* Platform Bağımsızlığı: Tek bir kaynak şablondan AWS (AMI), Azure (VHD) ve VMware (OVA) gibi farklı formatlarda çıktılar üretilebilir.
* Dev/Prod Parity: Geliştiricilerin yerel ortamda (Docker/Vagrant) kullandığı imaj ile Production ortamında çalışan imajın aynı kaynaktan üretilmesi, ortam farklılıklarından kaynaklanan hataları minimize eder.
