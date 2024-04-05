# 🚲 Azure Data Factory

Azure Data Factory, büyük veri işleme ve entegrasyon senaryoları için kullanılabilecek bulut tabanlı bir veri entegrasyon hizmetidir. Verileri farklı veri kaynaklarından toplayabilir, işleyebilir ve ardından bu verileri analiz ve raporlama için farklı hedeflere aktarabilir. Azure Data Factory'nin güçlü yönlerinden biri, karmaşık ETL (Extract, Transform, Load) işlemlerini görsel bir arayüz aracılığıyla kolayca tasarlamanıza ve yönetmenize olanak tanımasıdır.

<figure><img src="../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

* Farklı veri kaynaklarından verileri toplar ve bunları merkezi bir konumda birleştirir.
* Verileri işleyerek, analiz için uygun hale getirir. SQL, Python gibi dilleri kullanarak veri üzerinde dönüşümler ve hesaplamalar yapılabilir.
* Veri işleme işlemlerini planlama ve otomatize etme yeteneği sağlar, böylece manuel müdahaleye olan ihtiyacı azaltır.





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
