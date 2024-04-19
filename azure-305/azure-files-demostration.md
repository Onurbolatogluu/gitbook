# 🥌 Azure Files Demostration



<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Azure Files, sanal makineler (VM'ler) ve bulut tabanlı uygulamalar gibi çeşitli senaryolarda dosya depolama ihtiyaçlarını karşılamak için kullanılır. Temel olarak, Azure Files, dosyaları bulutta depolamanıza, **paylaşmanıza** ve erişmenize olanak tanır. Azure Files, birçok senaryoda kullanılabilir, örneğin dosya paylaşımı, uygulama veri depolama, yedekleme depolama alanı, web içeriği paylaşımı ve daha fazlası.&#x20;

#### Bağlantı Metotları:

* **SMB (Server Message Block) Protokolü**: SMB, Windows, macOS ve Linux dahil çoğu işletim sisteminde desteklenen bir ağ dosya paylaşım protokolüdür. Azure Files, SMB 2.1, 3.0 ve 3.1.1 sürümlerini destekler, bu da neredeyse her modern işletim sisteminden Azure Files'a bağlanabileceğiniz anlamına gelir.
* **NFS (Network File System) Protokolü**: NFS, özellikle Linux ve UNIX sistemlerinde yaygın olarak kullanılan bir dosya paylaşım protokolüdür. Azure Files, NFS v4.1 protokolünü destekler, ancak bu destek yalnızca **premium** depolama hesapları ile sınırlıdır.
* **REST API'leri**: Uygulama geliştiricileri ve otomasyon araçları, Azure Files'a programatik olarak erişmek için Azure Storage REST API'lerini kullanabilir. Bu metod, dosya yükleme, indirme, listeleme ve yönetme işlemlerini gerçekleştirmek için geniş bir esneklik sunar.
* **Azure Storage SDK'ları**: Microsoft, .NET, Java, Python, Node.js ve daha fazlası gibi popüler programlama dilleri için Azure Storage SDK'ları sunar. Bu SDK'lar, uygulamalarınızın Azure Files ile etkileşimini kolaylaştırmak için kullanılabilir.

Azure Files'a bağlanma yöntemi, kullanım senaryonuza ve kullandığınız platforma bağlı olarak değişir. Windows kullanıcıları genellikle SMB protokolünü tercih ederken, Linux kullanıcıları NFS'yi tercih edebilir. Uygulama geliştiricileri ve sistem yöneticileri, Azure Files ile etkileşimde bulunmak için REST API'leri veya Azure Storage SDK'larını kullanabilir.

\
Azure Files'ta **Standard** ve **Premium** dosya paylaşımı seçenekleri arasında birkaç temel fark bulunmaktadır:

1. **Depolama Türü**:
   * **Standard**: HDD (Hard Disk Drive) kullanır. Bu, daha düşük performans ve daha yüksek gecikme süreleri anlamına gelir.
   * **Premium**: SSD (Solid State Drive) kullanır. SSD'lerin daha hızlı erişim süreleri ve daha düşük gecikme süreleri vardır.
2. **Günlük İşlem Hacmi (IOPS) ve Bant Genişliği**:
   * **Standard**: IOPS 10.000'e kadar.
   * **Premium**: IOPS 100.000'e kadar.
3. **Maliyet**:
   * **Standard**: Daha düşük performans ölçütleri nedeniyle maliyeti daha düşüktür.
   * **Premium**: Daha yüksek performans ve SSD depolama kullanımı nedeniyle maliyeti daha yüksektir. Premium, genellikle yalnızca ZRS (Zone-Redundant Storage) olarak sunulur ve bu da belirli bölgelerde mevcut olabilir.
4. **Bölgesel Kısıtlamalar**:
   * **Standard**: Genellikle tüm bölgelerde mevcuttur.
   * **Premium**: Genellikle yalnızca belirli bölgelerde ve yalnızca ZRS seçeneğiyle mevcuttur.



Azure Files, çeşitli seçenekler sunarak farklı kullanım senaryolarına uygun depolama hizmetleri sağlar.&#x20;

1. **Premium**: Premium dosya paylaşımları, düşük gecikme süresi ve yüksek performans sunan SSD sürücülerini kullanır. Bu seviye, G/Ç yoğun iş yükleri için önemlidir. SMB ve NFS protokolleri aracılığıyla Premium dosya paylaşımları bağlanılabilir.
2. **Transaction Optimized**: Bu seviye, yoğun iş yüklerinin olduğu ve aynı zamanda Premium dosya paylaşımlarının sağladığı düzeyde gecikmeye ihtiyaç duymadığı senaryolarda kullanılır. Bu seviye, verileri depolamak için HDD'leri kullanır.
3. **Hot**: Hot seviyesi, genel amaçlı dosya paylaşımları için idealdir. Takımların dosya paylaşımı için kullandığı küçük ve orta ölçekli dosya paylaşımları için uygundur. Hot seviyesi, verileri depolamak için standart performans seviyesini kullanır.
4. **Cool**: Cool seviyesi, arşivleme senaryoları için idealdir ve Hot seviyesine göre maliyet açısından daha ekonomiktir.  Verileri depolamak için standart performans seviyesini kullanır.



#### Comparing Blob Storage, Azure Files, and Azure NetApp files:

* **Azure Blob Storage**: Bu depolama çözümü, büyük ölçekli, okuma ağırlıklı ardışık erişim yükleri için ideal. Verilerin bir kez sisteme alınıp daha sonra değiştirildiği senaryolar için düşük maliyetli bir çözüm olarak ön plana çıkar. Desteklenen protokoller arasında NFS 3.0, REST API ve ADLS GEN2 yer alıyor.
* **Azure Files**: Yüksek kullanılabilirlik sunan bir dosya paylaşım hizmetidir ve rastgele erişim yükleri için en uygun olanıdır. NFS, tam POSIX dosya sistemi desteği sunar ve ACI ve AKS'de (Azure Container Instances ve Azure Kubernetes Service) persistent storage alanı olarak kullanılabilir. Desteklenen protokoller SMB ve NFS 4.1'i içerir.
* **Azure NetApp Files**: NetApp tarafından sağlanan ve gelişmiş yönetim özellikleri ile donatılmış, tam yönetilen bir bulut dosya paylaşımı hizmetidir. Çoklu protokol desteği ve veri koruma özellikleri gerektiren rastgele erişim yükleri için idealdir. Desteklenen protokoller NFS 3.0, NFS 4.1 ve SMB'dir.



### Demo 1:

**Azure File Share kullanarak,  NFS Paylaşımları Linux sistemlere bağlamak.**



**Adım 1: Storage Account oluşturma**

Aşağıdaki komutu kullanarak, Azure'da "demoaccountforshare" adında bir Premium dosya depolama hesabı oluşturuyoruz. Premium olması önemli çünkü, NFS paylaşımı da test edeceğiz. Standard sürüm sadece SMB desteklemektedir.

```powershell
az storage account create --name demoaccountforshare1 --resource-group rg-fileshare-demo-001 --location westeurope --sku Premium_LRS --kind FileStorage
```

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Adım 2: Paylaşım oluşturma

Aşağıdaki komut, "demoaccountforshare1" adlı depolama hesabı üzerinde "mynfsshare1" adında bir NFS dosya paylaşımı oluşturur. Dosya paylaşımı kaynağı, 100 GB'lık bir depolama kotasıyla ve "AllSquash" ayarına sahip olarak oluşturulur.

`--root-squash` seçeneği, NFS (Network File System) üzerinde dosya paylaşımı yaparken kök kullanıcının davranışını kontrol etmek için kullanılır.

```powershell
az storage share-rm create -g rg-fileshare-demo-001 --storage-account demoaccountforshare1 --name mynfsshare1 --enabled-protocols NFS --quota 100 --root-squash AllSquash
```

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Adım 3: NFS network ayarlarının yapılandırılması

Sunucular private endpoint kullanarak bağlanacağı için, Network menüsü altından, yeni bir private endpoint oluşturmalıyız. (Private endpoint kurulumda seçtiğimiz vnet ile, nfs paylaşımına bağlancak sunucular aynı vnet içerisinde olmalı)

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

Secure transfer required özelliği desteklenmediği için devre dışı bırakmalıyız.

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

#### Adım 4: Ubuntu 22.04 Paylaşımı mount etme

Aşağıdaki komut ile, storage account ile iletişimimizi kontrol ediyoruz.

```bash
nslookup demoaccountforshare1.file.core.windows.net
```

<figure><img src="../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Gördüğünüz gibi, iletişim private endpoint kullanarak, aynı vnet üzerinden sağlanılıyor.



NFS Kurulumu ve mount komutları,

```
sudo apt-get -y update
sudo apt-get install nfs-common
sudo mkdir -p /mount/demoaccountforshare1/mynfsshare1
sudo mount -t nfs demoaccountforshare1.file.core.windows.net:/demoaccountforshare1/mynfsshare1 /mount/demoaccountforshare1/mynfsshare1 -o vers=4,minorversion=1,sec=sys,nconnect=4
```

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

Paylaşımı başarılı bir şekilde bağladık. Eğer paylaşıma on-prem sunucularımızdan ve/ya Azure dışında bulunan sunuculardan bağlanmak istersek,

* Azure Express route
* Site to site VPN&#x20;
* Point to site VPN

seçeneklerinden birini tercih etmeliyiz. Misal, [https://learn.microsoft.com/tr-tr/azure/storage/files/storage-files-configure-p2s-vpn-linux?WT.mc\_id=Portal-Microsoft\_Azure\_FileStorage](https://learn.microsoft.com/tr-tr/azure/storage/files/storage-files-configure-p2s-vpn-linux?WT.mc\_id=Portal-Microsoft\_Azure\_FileStorage)

<figure><img src="../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>



***

### **Demo 2:**

**Azure File Share kullanarak,  SMB Paylaşımları Windows/Linux sistemlere bağlamak.**



**Adım 1: SMB paylaşımı oluşturmak.**

smbshare1 adında, smb paylaşımı oluşturuyoruz.

<figure><img src="../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

#### Adım 2: Paylaşımı mount etmek

Paylaşımı mount edeceğimiz sunucuya bağlanıp, computer'e gelip, "Map network drive" seçeneğini seçiyoruz.

<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

Ardından, Storage account ismimizi ve paylaşım ismimizi girip, "connect using different credentials" seçeneğini seçip, ardından çıkan ekranda, "Use a different account" seçeneğini giriyoruz.

Folder: `\\anexampleaccountname.file.core.windows.net\file-share-name`

Username kısmına: storage account ismini,

Password kısmına: storage account key bilgisini giriyoruz (key1 veya key2)

<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

İşlemler bu kadar, paylaşımımız artık windows sunucumuza bağlandı. fdsf klasörünü test için oluşturdum.&#x20;

<figure><img src="../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

Linux (ubuntu 22.04) sunucuya bağlamak için ise, Azure portal 'a gelip,  Paylaşımımızın üzerine tıklayıp, Connect butonuna bastığımızda bize bir script verecek, bu scripti kopyalıyoruz.



<figure><img src="../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

Sunucuya gelip, mount.sh adında bir dosya oluşturup, bu scripti içerisine yapıştırıyoruz.

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

scripti çalıştırmadan önce, `chmod +x mount.sh` komutuyla scriptimize çalıştırma yetkisi tanımlıyoruz. Ardından, sistemimizde smb paketi yüklü değilse, aşağıdaki komut ile yüklüyoruz.

```bash
sudo apt install cifs-utils
```

Ardınan scriptimizi çalıştırıyoruz,

```bash
./mount.sh
```

Tüm işlemler tamamlandı. Artık smb paylaşımımız sunucuya mount oldu. df -h komutu ile kontrol edebiliriz.

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Unutmayın, siz scripti istediğiniz gibi değiştirebilir ve istediğiniz path'e mount edebilirsiniz. (Credentials bilgileri elbette azure 'ın verdiği gibi olmalı).



Kaynaklar:

{% embed url="https://learn.microsoft.com/en-us/azure/storage/files/storage-files-quick-create-use-linux" %}

{% embed url="https://learn.microsoft.com/tr-tr/azure/storage/files/storage-how-to-use-files-linux?tabs=Ubuntu%2Csmb311" %}

