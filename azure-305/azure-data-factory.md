# 🚲 Azure Data Factory

Azure Data Factory, büyük veri işleme ve entegrasyon senaryoları için kullanılabilecek bulut tabanlı bir veri entegrasyon hizmetidir. Verileri farklı veri kaynaklarından toplayabilir, işleyebilir ve ardından bu verileri analiz ve raporlama için farklı hedeflere aktarabilir. Azure Data Factory'nin güçlü yönlerinden biri, karmaşık ETL (Extract, Transform, Load) işlemlerini görsel bir arayüz aracılığıyla kolayca tasarlamanıza ve yönetmenize olanak tanımasıdır.

<figure><img src="../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

* Farklı veri kaynaklarından verileri toplar ve bunları merkezi bir konumda birleştirir.
* Verileri işleyerek, analiz için uygun hale getirir. SQL, Python gibi dilleri kullanarak veri üzerinde dönüşümler ve hesaplamalar yapılabilir.
* Veri işleme işlemlerini planlama ve otomatize etme yeteneği sağlar, böylece manuel müdahaleye olan ihtiyacı azaltır.

**Azure Data Factory'nin ana bileşenleri**:

#### 1. **Pipeline**

Pipeline'lar, iş akışlarını tanımlayan ve yöneten temel yapı taşlarıdır. Bir pipeline, veri kopyalama, veri dönüştürme gibi bir veya daha fazla aktiviteyi içerebilir. Bu aktiviteler sıralı veya paralel olarak yürütülebilir. Pipeline'lar, belirli bir iş sürecini otomatize etmek için tasarlanmıştır, örneğin günlük veri yüklemeleri veya haftalık veri dönüştürme işlemleri gibi.

#### 2. **Data Flows**

Data Flows, veri dönüştürme aracıdır. Veri dönüştürme ve zenginleştirme işlemlerini kolayca tasarlamak için kullanılır. Data Flows, kaynak ve hedef arasında veri dönüştürme işlemlerini destekler. Örneğin, bir CSV dosyasındaki verileri temizleyebilir, dönüştürebilir ve ardından bir SQL veritabanına yükleyebilir.

#### 3. **Datasets**

Datasets, veri kaynaklarını temsil eder. Bir dataset, bir veri kaynağının yapısal tanımını içerir ve Azure Data Factory'nin, üzerinde işlem yapacağı verilerin nerede ve nasıl bir formatta olduğunu belirlemesine yardımcı olur. Örneğin, bir SQL veritabanı tablosu veya bir Azure Blob Storage'daki bir dosya bir dataset olarak tanımlanabilir.

#### 4. **Linked Services**

Linked Services, veri kaynakları ve hizmetleriyle olan bağlantıları tanımlar. Azure Data Factory'nin veri kaynaklarına nasıl erişeceğini belirleyen yapılandırma bilgilerini içerir. Örneğin, bir Azure SQL Database, Azure Blob Storage veya bir SaaS uygulaması bir Linked Service ile tanımlanabilir.

#### 5. **Triggers**

Triggers, pipeline çalıştırılmasını tetikleyen olaylardır. Bunlar, zaman tabanlı tetikleyiciler (belirli zamanlarda veya periyotlarda çalıştırma) veya olay tabanlı tetikleyiciler (örneğin, bir dosya bir depolama hesabına yüklendiğinde) olabilir.



### DEMO : Basit bir Pipeline oluşturulması,



1 - Azure portal'a giriş yapıp, Azure factory servisini bulup, yeni bir Azure factory servisi oluşturuyoruz.

<figure><img src="../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

2 - Servisimiz oluştuktan sonra, overview sekmesini kullanarak Azure Data Factory Studio giriş yapmalıyız.

<figure><img src="../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

3 - Yeni bir pipeline oluşturuyoruz.

<figure><img src="../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

4 - Copy data fonksiyonunu seçip, rest api metodunu kullanacağız.

url : [https://swapi.dev/api/people](https://swapi.dev/api/people.)

<figure><img src="../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

5 - Datayı saklayacağız yeri seçiyoruz. Bizim örneğimizde, table storage kullanacağız.&#x20;

<figure><img src="../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

6 - Datasets menüsü altında table storage'a gelip, seçili bir tablo olduğundan emin olmalıyız.

<figure><img src="../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

7 - Mappings kısmına gelip, "import schemes" seçeneğiyle rest api kullanarak bilgileri çekiyoruz.

<figure><img src="../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>

8 - Sadece 3 adet veriyi scrape edeceğimiz için, gereksiz kısımları exclude ediyoruz. Sadece, name-height-hair\_color parametrelerini seçiyoruz. Burada önemli olan kısım, string name "$\['results']\[0]\['name']" şeklinde değil "name" şeklinde olmalıdır. Ekteki gibi. Çünkü zaten, array olduğunu Collection reference parametresinde belirttik.

<figure><img src="../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

9 - Yapılandırma tamamlandı. Tüm ayarları validate etmek için, **validate** butonuna basıyoruz. ve sol üstte bulunan **"publish all"** butonuna basıp yapılandırmayı kaydediyoruz. Ardından testimizi başlatmak için, "debug" butonuna basıyoruz. Ardından aşağıda açılan task bilgilendirme satırı üzerinden gözlük ikonuna basıp, taskın durumunu kontrol etmeliyiz.

<figure><img src="../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

10 - Gördüğünüz üzere, tüm bilgileri restapi kullanarak çektik ve storage account altında tablomuza başarıyla yazdık.

<figure><img src="../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>



{% embed url="https://www.tevpro.com/blog/creating-your-first-azure-data-factory-pipeline" %}

{% embed url="https://learn.microsoft.com/en-us/azure/data-factory/" %}

{% embed url="https://k21academy.com/microsoft-azure/data-engineer/create-azure-data-factory-pipeline/" %}
