---
icon: layer-group
---

# Desteklenen Platformlar

#### 1. Platform Çeşitliliği ve Hibrit Yapı

* Public Cloud: AWS (AMI), Azure (Managed Image), Google Cloud (GCP Image) gibi sağlayıcılar için imajlar üretir.
* Virtualization: Şirket içi (On-premise) veri merkezleri için VMware (vSphere, ISO), Hyper-V, QEMU ve VirtualBox formatlarını destekler.
* Containerization: Docker için imajlar oluşturabilir ve bunları Docker Hub veya Private Registry yükleyebilir.

#### 2. Genişletilebilir Eklenti Mimarisi

* Official:HashiCorp mühendisleri tarafından geliştirilen ve bakımı yapılan, en yüksek güvenilirlik seviyesindeki eklentilerdir (Örn: AWS, Azure).
* Verified: Teknoloji ortakları (Örn: DigitalOcean, CloudStack) tarafından geliştirilen ve HashiCorp tarafından onaylanan eklentilerdir.
* Community: Açık kaynak topluluğu tarafından geliştirilen, niş veya özel ihtiyaçlara yönelik eklentilerdir. Bu yapı, Packer’ın her türlü özel altyapıya adapte edilebilmesini sağlar.

#### 3. Stratejik Avantaj

Çoklu Bulut (Multi-Cloud) stratejisi izleyen organizasyonlar için Packer kritik bir stratejik araçtır.

* İşletim sistemi ve uygulama yapılandırması tek bir Packer şablonunda tanımlanır.
* Aynı şablon kullanılarak eş zamanlı hem AWS AMI hem de Azure Image üretilebilir. Bu, organizasyonun tek bir bulut sağlayıcısına teknik olarak bağımlı kalmasını engeller ve Migration maliyetlerini düşürür.

#### 4. Dev/Prod Parity

* Senaryo: Geliştirici, kendi bilgisayarında (Localhost) çalışırken Docker veya VirtualBox kullanır. Prod ortamı ise AWS üzerindedir.
* Çözüm: Packer, aynı kaynak kodunu kullanarak geliştiriciye bir Docker imajı, Prod ortamına ise bir AWS AMI üretir.
* Sonuç: "Lokalde çalışıyordu ama sunucuda patladı" sorunu elimine edilir. Dev ve Prod ortamları, aynı konfigürasyon DNA'sını taşır.
