---
icon: smoke
---

# Public Clouds

Packer, endüstri standardı olan üç büyük bulut sağlayıcısını Official Plugins statüsünde destekler. Bu statü, eklentilerin doğrudan HashiCorp mühendisliği tarafından geliştirildiği ve bakımının yapıldığı anlamına gelir.

* Amazon Web Services: Packer, AWS üzerinde AMI (Amazon Machine Image) üretimi gerçekleştirir. Bu süreçte `ec2:RunInstances` ve `ec2:CreateImage` gibi spesifik IAM izinlerine ihtiyaç duyar.
* Microsoft Azure: Packer, bu platform için Azure Managed Images veya VHD artifact üretir.
* Google Cloud Platform: Packer, burada Google Compute Engine (GCE) imajları oluşturur.

#### 2. Platform ve Hibrit Yapı

* Sanallaştırma: Veri merkezlerinde çalışan VMware (vSphere), Hyper-V ve Dev ortamları için VirtualBox/Parallels desteklenir.
* Konteynerizasyon: Docker için imajlar üretilerek, mikroservis mimarilerine uyum sağlanır.
* Alternatif: DigitalOcean, OpenStack gibi platformlar da ekosisteme dahildir.

#### 3. Stratejik Yönetim

* Packer, tek bir kaynak şablonundan eş zamanlı olarak hem AWS hem de Azure için imaj üretebilir. Bu, organizasyonun tek bir sağlayıcıya teknik olarak bağımlı kalmasını engeller.
* Geliştiricilerin lokal ortamda (Docker/VMware) kullandığı imaj ile Production ortamındaki AWS imajının aynı kaynaktan üretilmesi, "Configuration Drift" riskini minimize eder ve ortamlar arası tutarlılığı garanti altına alır.

#### 4. Operasyonel Farklılıklar

* AWS gibi bulut ortamlarında Packer, iletişim için standart olarak geçici bir SSH anahtarı üretir. `-debug` modu aktif edildiğinde ise bu anahtar, kullanıcının sunucuya bağlanıp manuel inceleme yapabilmesi için yerel diske (.pem) kaydedilir; işlem bittiğinde silinir.
* VMware veya VirtualBox gibi yerel ortamlarda ise süreç, `PACKER_LOG=1` ile elde edilen detaylı loglar ve `Kickstart/Preseed` dosyalarındaki statik kimlik bilgileri üzerinden yürütülür.

