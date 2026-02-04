---
icon: hat-cowboy
---

# HashiCorp Packer Nedir?

#### 1. Packer Nedir?

HashiCorp Packer, modern altyapı yönetiminde Infrastructure as Code (IaC) ekosisteminin kritik bir parçasıdır. Temel amacı, tek bir kaynak konfigürasyonundan (source configuration) birden fazla platform için makine imajları (machine images) oluşturmaktır.

#### 2. Packer Core Concepts (Temel Kavramlar)

Packer'ın çalışma mantığını oluşturan 5 ana bileşen şunlardır:

* Templates: Yapının nasıl kurulacağını tarif eden, bir "yemek tarifi" gibi işlev gören dosyalardır. JSON veya HCL2 formatında yazılırlar.
* Builders: İmajın _hangi platformda_ (AWS, Azure, vSphere, Docker vb.) oluşturulacağını belirleyen modüllerdir. İlgili platformla konuşan yürütme motorudur.
* Provisioners: İmaj son halini almadan önce, build süreci esnasında (makine geçici olarak çalışırken) içine yazılım yüklemek veya konfigürasyon yapmak için kullanılır. (Örn: Ansible, Shell scriptleri, PowerShell).
* Post-processors: İmaj oluşturma işlemi bittikten _sonra_ devreye giren işlemlerdir. İmajı sıkıştırmak, buluta upload etmek veya bir artifact manifest dosyası oluşturmak bu aşamada yapılır.
* Artifacts: Build süreci sonunda ortaya çıkan çıktıdır. (Örn: AWS için bir AMI ID, vSphere için bir dosya dizini).

#### 3. Immutable Infrastructure (Değişmez Altyapı) Bağlamı

Packer, modern mimarilerin temeli olan Immutable Infrastructure yaklaşımını destekler. Avantajları şunlardır:

* Configuration Drift Engelleme: Uygulama kodu ve sistem paketleri imajın içine en baştan "pişirildiği" (baked) için, sunucular başlatıldıktan sonra manuel güncellemeye ihtiyaç duymaz.
* Hızlı Deployment: Tamamen konfigüre edilmiş bir imaj kullanıldığı için sunucuların ayağa kalkması saniyeler sürer.
* Multi-provider Portability: Tek bir Template üzerinden AWS, Google Cloud veya VMware için özdeş imajlar üretilebilir. (Örn: Prod'da AWS, Dev ortamında Docker kullanımı).
* CI/CD Entegrasyonu: Pipeline süreçlerine kolayca entegre edilir. Her kod değişikliğinde yeni bir imaj build edilip test edilebilir.

#### 4. Operasyonel Süreç ve Debugging

Packer kullanımı sırasındaki iş akışı şu şekildedir:

* Workflow: Süreç genellikle `packer init` ile başlar, ardından şablon doğrulanır ve `packer build` komutuyla imaj oluşturulur.
* Debugging: Sorun gidermek için `packer build -debug` komutu kullanılır. Bu modda Builders adımlar arasında durur ve kullanıcıya SSH ile bağlanıp inceleme şansı verir.
* Logging: Detaylı logları görmek için `PACKER_LOG=1` ortam değişkeni kullanılır.

