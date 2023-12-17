# 🔧 Managing Storage

### import & export service **(WAImportExport tool);**

Azure Storage Import/Export hizmeti, müşterilerin büyük miktarda veriyi Azure Blob Storage ve Azure Files'a aktarmalarını ve bu hizmetlerden veri çıkarmalarını sağlayan bir Azure hizmetidir. Fiziksel olarak taşınabilir depolama aygıtlarını kullanarak Azure veri merkezlerine veri aktarımını kolaylaştırır. Bu, özellikle büyük miktarda verinin ağ üzerinden taşınmasının pratik olmadığı durumlar için yararlıdır.

**İçe Aktarma(import) İş Akışı:**

1. Azure Portal üzerinde bir içe aktarma işi oluşturulur ve hedeflenen Azure Storage hesabı belirtilir.
2. Azure Import/Export hizmeti kullanılarak veri taşıyacak sabit diskler hazırlanır. Bu hazırlık işlemi, verilerin diskler üzerine kopyalanmasını ve dosyalarının oluşturulmasını içerir.
3. Hazırlanan sabit diskler, sağlanan takip kimliği ile birlikte Azure veri merkezine gönderilir. Disklerin Microsoft tarafından işlendikten sonra geri gönderilebilmesi için bir iade adresi de belirtilmelidir.
4. Azure veri merkezinde, veri sabit diskten hedef depolama hesabına kopyalanır.
5. Veri transferi tamamlandıktan sonra, sabit diskler boşaltılır ve belirtilen iade adresine geri gönderilir.

**Dışa Aktarma(export) İş Akışı:**

1. Azure Portal üzerinde bir dışa aktarma işi oluşturulur ve hangi verilerin taşınacağı belirlenir.
2. Sabit diskler Azure veri merkezine gönderilir.
3. Diskler veri merkezinde işlenir ve seçilen veriler depolama hesabından sabit diskler üzerine kopyalanır. Bu sırada diskler BitLocker ile şifrelenir ve şifre çözme anahtarları iş ile ilişkilendirilir.
4. Veri kopyalama işlemi tamamlandıktan sonra, diskler paketlenir ve gönderim için hazırlanır.
5. Şifrelenmiş diskler müşteriye gönderilir ve müşteri, Azure Portal üzerinden sağlanan BitLocker anahtarları kullanarak diskteki verinin şifresini çözebilir.

Bu iş akışları, özellikle yüksek hızda veri transferi gerektiren veya ağ üzerinden veri aktarımının pratik olmadığı durumlar için tasarlanmıştır.



### Azure Storage Explorer;

<figure><img src="../.gitbook/assets/image (206).png" alt="" width="563"><figcaption></figcaption></figure>

Azure Storage Explorer, Azure depolama kaynaklarını masaüstünüzden yönetmenize olanak tanıyan bir uygulamadır. Bu aracın ana özellikleri şunlardır:

1. Azure Storage Explorer, Windows, macOS ve Linux işletim sistemleri üzerinde çalışabilir.
2. Blob, File, Queue ve Table depolama hizmetleri ile uyumlu çalışır ve kullanıcıların bu hizmetlerdeki verileri kolayca gözden geçirmelerine, yüklemelerine, indirmelerine ve düzenlemelerine olanak tanır.
3. Kullanıcılar, Azure Storage Explorer aracılığıyla bulutta veri yükleyebilir, indirebilir, silme ve taşıma işlemleri yapabilir, ayrıca Blob, Queues ve Tables'daki verileri düzenleyebilir ve sorgulayabilir.
4. Kullanıcılar, aboneliklerine, depolama hesaplarına ve bu hesaplar altında yer alan tüm verilere tek bir arayüzden erişebilir.
5. Kullanıcılar, SAS token'ları (Shared Access Signature), Azure Active Directory kimlik doğrulaması ve depolama hesap anahtarları ile güvenli bir şekilde erişim sağlar.
6. Büyük veri kümelerinin yüklenmesi ve indirilmesi için optimize edilmiştir. Paralel yükleme ve indirme desteği bulunur.
7. Kullanıcılar, büyük miktarda veri arasında kolayca arama yapabilir ve sorgu dillerini kullanarak gelişmiş filtrelemeler gerçekleştirebilir.
8. Azure Cosmos DB hesaplarını yönetme ve veri keşfetme yeteneği de dahil olmak üzere, NoSQL veri tabanı hizmetleriyle de çalışabilir.

Bu özellikler, Azure Storage Explorer'ı, Azure depolama kaynaklarını yönetmek isteyen herhangi bir kişi veya kuruluş için kapsamlı ve esnek bir araç haline getirir.



### AZCopy;

<figure><img src="../.gitbook/assets/azure-command-line-tool-for-data-transfer.png" alt=""><figcaption></figcaption></figure>

AzCopy, Microsoft Azure tarafından sağlanan komut satırı bir araçtır ve verileri Azure Storage hizmetleri arasında, ya da on-premises (yerel) depolama ile Azure Storage hizmetleri arasında veri transfer etmek için kullanılır. İşte AzCopy'nin ana özellikleri ve işlevleri:

AzCopy, Azure Blobs, Azure Files, Amazon S3, Google Cloud Storage (GCP) ve Azure Data Lake Storage Gen2 (ADLS Gen2) gibi farklı bulut depolama platformları arasında veri transferini destekler. Her AzCopy işlemi, bir iş kimliği ve ilişkili bir log dosyası oluşturur. Bir iş başarısız olursa, kullanıcılar bu log dosyasını inceleyerek sorunun kaynağını anlayabilir ve işi yeniden başlatabilir. Kullanıcılar, belirli dosyaları dahil etmek veya hariç tutmak için `include` ve `exclude` flags kullanabilir, wildcard patterns ile çalışabilir ve `recursive` flag ile bir klasördeki tüm dosyaları kopyalayabilir. AzCopy, SAS tokens (Shared Access Signature) veya Azure Active Directory aracılığıyla kimlik doğrulaması kullanarak güvenli bir şekilde çalıştırılabilir ve Windows, Linux ve macOS işletim sistemlerinde çalışabilir.



`azcopy copy <kaynak> <hedef> [seçenekler]`&#x20;



```powershell
# Get help
azcopy /?

# Copy files
azcopy copy <source> <destination> [options]
azcopy copy ./myfiles/visio.png https://mystaccount.blob.core.windows.net/files/image.png?sv=2020-08-04&ss=bfqt&srt=sco&sp=rwdlacup&se=2022-05-19T14:31:40Z&st=2022-05-19T06:31:40Z&sip=168.11.12.13-168.11.12.19&spr=https&sig=66iXqzZSakarJ05J210%2ByoPRVXTeTo%2FTJcHHSEKUjHr0%3D

# Copy using AAD
azcopy login --tenant-id xxxx-xxxx-xxxx-xxxxxxxx-xxxx
azcopy copy ./myfiles/visio.png https://mystaccount.blob.core.windows.net/files/image.png
# Bu durumda, erişim için SAS token'ına gerek yoktur çünkü kullanıcı zaten AAD ile kimlik doğrulaması yapmıştır.
# Login olan kullanıcının container veya storage account altında IAM rolünün olması gerekiyor.
```



***

**Examples;**

```powershell
.\azcopy.exe copy .\Downloads\v2-cfdd802421715361ba6edb8ba864d70_720w.png "https://demoaz8890.blob.core.windows.net/images/image.png?sv=2020-08-04&ss=b&srt=sco&sp=rwdlacitfx&se=2022-06-07T01:32:19Z&st=2022-06-06T17:32:19Z&spr=https&sig=s%2FHdUmbIOBwQNOaHjz8w8p5dvSr99RJ9G0wPdPMMKIw%3D"
```

* `.\azcopy.exe`: AzCopy programını çalıştırır. Nokta ve ters eğik çizgi (`.\`), Windows'ta mevcut dizindeki bir uygulamayı çalıştırmak için kullanılır.
* `copy`: AzCopy'nin dosya kopyalama işlemi için kullanılan komutudur.
* `.\Downloads\v2-cfdd802421715361ba6edb8ba864d70_720w.png`: Bu kısım, AzCopy'nin kopyalayacağı yerel dosyanın yolunu ve adını belirtir. Bu örnekte, `Downloads` klasöründe `v2-cfdd802421715361ba6edb8ba864d70_720w.png` adında bir dosya kopyalanacaktır.
* `"https://demoaz8890.blob.core.windows.net/images/image.png?...`": Bu uzun URL, dosyanın kopyalanacağı Azure Blob Storage'daki hedefin yolunu ve adını içerir. URL'nin sonundaki sorgu dizesi (query string), erişim için kullanılacak Shared Access Signature (SAS) token'ını içerir.
* Özetle, bu komut bir kullanıcının yerel bilgisayarındaki bir PNG dosyasını, Azure Blob Storage'daki bir konuma kopyalamak için kullanılır.

{% hint style="info" %}
Hangi storage account altındaki container içerisine kopyalanacaksa, container "properties" kısmından container url alınmalıdır.

\
example  :  https://demoaz8890.blob.core.windows.net/images
{% endhint %}

<figure><img src="../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

```powershell
.\azcopy.exe copy .\Downloads\computers\ https://demoaz8890.blob.core.windows.net/images/ --recursive
```

Bu komut, `.\Downloads\computers\` dizinindeki tüm dosyaları, `--recursive` seçeneği sayesinde, yani klasör içindeki tüm alt klasörler ve dosyalar dahil olacak şekilde, `https://demoaz8890.blob.core.windows.net/images/` adresindeki Azure Blob Storage'a kopyalar. `--recursive` seçeneği, belirtilen kaynak yolundaki tüm klasörleri ve alt klasörlerdeki dosyaları da dahil ederek kapsamlı bir kopyalama işlemi gerçekleştirir.

