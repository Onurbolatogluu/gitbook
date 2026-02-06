---
icon: memo
---

# Templates

<figure><img src="../.gitbook/assets/Screenshot 2026-02-04 at 17.24.10.png" alt=""><figcaption></figcaption></figure>

Packer'ın en temel bileşeni olan Template'ler, bir makine imajının nasıl üretileceğini belirleyen talimat kılavuzları'dır. Otomasyon sürecini yöneten ve diğer tüm bileşenleri bir araya getiren merkezi yapıdır.

**1. Yapı ve Format**

Template'ler, imaj inşa sürecini detaylandıran kod dosyalarıdır.

* Dil Standartları: Packer, yapılandırma için JSON veya HCL (HashiCorp Configuration Language) kullanır. Ancak, versiyon 1.7.0 itibarıyla HCL2'nin fiili standart olduğunu ve tercih edilmesi önerilir.
* İçerik Blokları: Bir Template genellikle şu bölümlerden oluşur:
  * Packer Block: Gerekli eklentileri (plugins) ve versiyonları tanımlar.
  * Locals Block: Yapılandırma içinde kullanılacak yerel değişkenleri tutar.
  * Source Block: Builder'ın hangi platformda (AWS, vSphere vb.) çalışacağını ve hangi temel imajı kullanacağını belirtir.
  * Build Block: Provisioner'ları ve Post-processor'ları bir araya getirerek gerçek inşa sürecini tanımlar.

**2. Temel Kavramlarla İlişkisi**

Template, Packer'ın diğer ana bileşenlerini organize eden bir orkestrasyon dosyasıdır:

* Builders ile Bağlantı: Template, belirli bir platform (örn. Amazon EBS veya vSphere-iso) için Builder'ı tetikler. Kimlik bilgileri ve bölge seçimleri buradan yönetilir.
* Provisioners ile Bağlantı: Template içinde tanımlanan Provisioner'lar (örn. Shell, Ansible, PowerShell), imaj statik hale gelmeden önce gerekli yazılım kurulumlarını ve yapılandırmaları yapar.
* Artifacts ve Post-processors: İnşa süreci bittiğinde, Template talimatlarına göre bir manifest dosyası oluşturulabilir veya Artifact bir bulut sağlayıcısına yüklenebilir.

**3. Stratejik Önem ve Kullanım Alanları**

Modern altyapı yönetiminde Template'ler kritik bir rol oynar:

* Immutable Infrastructure (Değişmez Altyapı): Uygulama kodu ve bağımlılıklar imaj oluşturulurken dahil edilir (baked). Sunucular açıldıktan sonra manuel SSH müdahalesi gerekmez, bu da yapılandırma kaymasını (configuration drift) önler.
* CI/CD Entegrasyonu: GitHub Runners gibi araçlarla entegre edilir. Kod/Paket değişikliği olduğunda Template kullanılarak otomatik yeni bir imaj sürümü üretilir.
* Debugging (Hata Ayıklama):
  * `packer validate` komutu ile kontrol edilebilir.
  * Hata durumunda `-on-error=ask` veya `-debug` modları kullanılabilir.
  * _Not:_ İnşa süreci yeniden denendiğinde (retry), Template dosyadan tekrar yüklenmez; sadece Builder kaynakları yenilenir.

Özetle: Packer Template'leri, sistem kurulumunu, güvenliğini ve ölçeklenebilirliğini kod olarak tanımlayan (IaC) ve bunu farklı bulut sağlayıcılarında tutarlı şekilde uygulayan temel araçlardır.
