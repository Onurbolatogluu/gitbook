---
icon: cable-car
---

# VMware, VirtualBox

* Packer, aynı şablon dosyasını kullanarak üretim ortamı için bir AWS AMI üretirken, eş zamanlı olarak Dev ortamı için bir VirtualBox/VMware imajı derleyebilir. Bu, "Localhost" ile "Cloud" arasındaki davranışsal farklılıkları minimize eder.

#### 2. Artifact Yönetimi ve Dağıtım&#x20;

Bulut platformlarında (örn. AWS) süreç sonunda sadece soyut bir imaj kimliği (AMI ID) oluşurken; yerel sanallaştırma süreçlerinde üretilen çıktılar (Artifacts) fiziksel ve dosya tabanlıdır.

Platform Çıktıları (Builder Artifacts):

* VMware: Genellikle bir sanal makine dizini, disk dosyaları (VMDK) veya taşınabilir OVF/OVA şablonları üretir.
* VirtualBox: `virtualbox-iso` oluşturucusu, ham işletim sistemi medyasından (ISO) başlayarak .vbox konfigürasyon dosyası ve .vdi sanal disk dosyalarını oluşturur.

#### 3. Ekosistem  ve Destek Seviyesi

* VMware ve VirtualBox; AWS, Azure ve GCP gibi sağlayıcılarla aynı destek seviyesine sahiptir. HashiCorp mühendisliği tarafından doğrudan yönetilir.
* Hyper-V, QEMU ve Parallels gibi diğer hypervisorler de desteklenerek, farklı işletim sistemi ve donanım altyapılarına uyum sağlanır.

#### 4. Hata Ayıklama ve Erişim Yöntemleri

Bulut ortamlarındaki "Ephemeral SSH Key" (Geçici Anahtar) mantığı, yerel sanallaştırmada yerini statik kimlik doğrulamaya bırakır.

* Yerel sanal makineler başlatılırken, işletim sistemi kurulum dosyalarında (`kickstart` veya `preseed`) tanımlanan kullanıcı adı ve şifreler kullanılır.
* `PACKER_LOG=1` parametresi, SSH bağlantı detaylarını ve provizyon sürecini izlemek için birincil araçtır. Buluttaki gibi geçici anahtar üretimi yerine, tanımlı statik şifreler üzerinden bağlantı sağlanır.



