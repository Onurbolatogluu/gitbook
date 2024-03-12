# 📠 Azure Functions Demostration

<figure><img src="../.gitbook/assets/6.webp" alt=""><figcaption></figcaption></figure>

Azure Functions, Microsoft'un sunmuş olduğu bir sunucusuz (serverless) computing hizmetidir. Bu hizmet, geliştiricilerin sunucu yönetimi ile uğraşmadan, kod parçalarını bulutta çalıştırmalarına olanak tanır. Azure Functions, yalnızca kodun çalıştırılması gerektiğinde çalışır ve kullanılan kaynak miktarına göre ödeme yapılmasını sağlar.&#x20;

#### SSS:

1. Azure Functions, belirli olaylar gerçekleştiğinde tetiklenebilir. Bu, gerçek zamanlı veri işleme, dosya işleme, otomatik yedekleme ve daha fazlası gibi senaryolar için idealdir.
2. Azure Functions, gereksinimlere göre otomatik olarak ölçeklenir, böylece uygulama daha fazla istek aldığında daha fazla kaynak ayrılabilir ve trafik azaldığında kaynaklar azaltılabilir.
3. Yalnızca kodun çalıştırıldığı süre için ödeme yapılır, bu da kullanılmayan kapasite için ödeme yapılmadığı anlamına gelir.
4. Azure Functions, birçok popüler programlama dilini destekler, böylece geliştiriciler mevcut becerilerini kullanabilirler.
5. Azure Functions, Azure hizmetleriyle ve diğer harici hizmetlerle kolayca entegre edilebilir.

#### Trigger Kaynakları:

Azure Functions, çeşitli kaynaklardan tetiklenebilir. Bunlar arasında:

* **HTTP İstekleri:** Webhooks veya RESTful API'lar aracılığıyla doğrudan HTTP istekleri.
* **Zamanlayıcı:** Belirlenen zaman aralıklarında otomatik olarak çalışan işlemler.
* **Storage Account Olayları:** Azure Blob Storage, Queue Storage ve Table Storage'da gerçekleşen değişiklikler.
* **Azure Cosmos DB:** Veritabanındaki değişikliklerle tetiklenen işlemler.
* **Service Bus Sıraları ve Konuları:** Mesaj sıralarına ve konularına gönderilen mesajlar.
* **Event Grid:** Azure ve üçüncü taraf hizmetlerden gelen olaylar.
* **Daha Fazlası:** GitHub olayları, IoT Hub mesajları vb.



Azure Functions, esnek, maliyet etkin ve güçlü bir sunucusuz platform sunarak, modern bulut tabanlı uygulama geliştirmeyi kolaylaştırır.

***

### DEMO:

Azure Functions kullanarak bir web API'si oluşturup. Bu API adresine HTTP istekleri gönderip, Azure Blob Storage'da bir container oluşturacağız. Oluşturmak istediğimiz container ismini de yaptığımız isteğin içerisinde parametre olarak verebileceğiz.

***

* Proje hazırlığının yapılması gerekmektedir. Biz örneğimizde, vscode kullanarak bu projeyi deploy edeceğiz.
* İlk olarak, aşağıdaki adresten Azure Functions Core Tools paketini çalışmayı yapacağımız cihaza kurmalıyız:&#x20;

{% embed url="https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?pivots=programming-language-python&tabs=windows,isolated-process,node-v4,python-v2,http-trigger,container-apps" %}

* Kurulum tamamlandıktan sonra, Microsoft'un dökümanlarında da bahsettiği gibi gerekli vscode eklentilerini kurmalıyız. Aşağıda kurulması gereken extensionlar hakkında kaynak paylaşıyorum.

{% embed url="https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-python#configure-your-environment" %}

* Ekran görüntüsünde de görüleceği üzere, Azurite,Azure Functions ve python extensionlarını vscode üzerine kurmamız gerekiyor.

<figure><img src="../.gitbook/assets/image (255).png" alt=""><figcaption></figcaption></figure>

* Extension kurulumlarını yapıp, işlemlerimize devam ediyoruz. Öncelikle boş bir dizin oluşturup, vscode üzerinde bu boş dizini import ediyoruz.

<figure><img src="../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

* Ardından, sol tarafta azure extensionuna tıklayıp, workspace içerisinde azure functions template'i oluşturuyoruz.

<figure><img src="../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

* Create functions butonuna tıklayıp, istenilen bilgileri girmeliyiz. Burada, çalışacağımız dizin, kullanacağımız programlama dili ve versiyonu ve trigger seçenekleri gibi bir çok değeri belirtiyoruz.

<figure><img src="../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

* Tüm bilgileri girdikten sonra, projemize ait bir template oluşmuş oldu. Şimdi bu bilgileri düzenlememiz gerekiyor.

<figure><img src="../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

* Uygulama kodumuzu, function\_app.py dosyası içerisini temizleyip, yapıştırıyoruz.&#x20;

```python
import azure.functions as func
from azure.storage.blob import BlobServiceClient
import os

app = func.FunctionApp()

@app.route(route="create_container", methods=["GET", "POST"], auth_level=func.AuthLevel.ANONYMOUS)
def main(req: func.HttpRequest) -> func.HttpResponse:
    connection_str = os.getenv('AZURE_STORAGE_CONNECTION_STRING')  
    container_name = req.params.get('containername')  

    if not container_name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            container_name = req_body.get('containername')

    if not container_name:
        return func.HttpResponse(
             "Please pass a container name on the query string or in the request body.",
             status_code=400
        )

    try:
        blob_service_client = BlobServiceClient.from_connection_string(connection_str)
        blob_service_client.create_container(container_name)
        return func.HttpResponse(f"Container '{container_name}' created successfully.", status_code=201)
    except Exception as e:
        return func.HttpResponse(
             f"Error creating the container: {str(e)}",
             status_code=500
        )

```

<figure><img src="../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>

* Ardından, requirements.txt dosyamızın içerisine aşağıdaki modülleri ekliyoruz.

```
azure-storage-blob
azure-functions
```

<figure><img src="../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

* Ardından, function.json adında bir dosya oluşturup, uygulamamıza ait diğer spesifik bilgilerin yer aldığı bazı parametreleri ekliyoruz. (Opsiyonel)

```json
{
    "bindings": [
      {
        "authLevel": "anonymous",
        "type": "httpTrigger",
        "direction": "in",
        "name": "req",
        "methods": ["get", "post"]
      },
      {
        "type": "http",
        "direction": "out",
        "name": "$return"
      }
    ]
  }
```

<figure><img src="../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

* Ardından, değişkenlerimizi ve diğer başlatma seçeneklerimizi ayarladığımız **local.settings.json** dosyasını düzenliyoruz. Burada `AZURE_STORAGE_CONNECTION_STRING` değişkeninin değerini işlem yapmak istediğimiz storage hesabı altında, access keys kısmından alıyoruz. Kodumuz, storage connection string'i buradan alacak.

<figure><img src="../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsFeatureFlags": "EnableWorkerIndexing",
    "AZURE_STORAGE_CONNECTION_STRING": "<AZURE_STORAGE_CONNECTION_STRING>"
  }
}
```

* Şu anda yapılacak tüm değişiklikler bu kadar, şimdi azurite extension'u çalıştırıp, lokal development ortamımızı ayağa kaldırmaya hazırız. vscode ekranında, klavyeden F1 tuşuna basıp azurite extensionunu ayağa kaldırıyoruz.

<figure><img src="../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

* Şimdi sıra geldi projemizi çalıştırmaya, Azure extensionu üzerine tıklayıp, "start debugging to update..." butonuna tıklayıp, gelen uyarıya da "debug anyway" butonuna basıp, uygulamayı ayağa kaldırıyoruz.

<figure><img src="../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

* Görüldüğü gibi, uygulama start oldu ve web üzerinden istekleri almaya hazır.&#x20;

<figure><img src="../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

* Uygulamamız web'den cevap verdi. Şimdi parametre vererek, storage account üzerinde container oluşturmaya çalışalım.

<figure><img src="../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

* Yukarıda gördüğünüz gibi, uygulamamız başarıyla çalıştı ve containerımız oluştu.
* Fakat bu uygulama şu anda lokalimde çalışıyor, yani bunu henüz azure üzerine deploy etmedik. Bu uygulamayı azure üzerine deploy etmek için Azure portal üzerinden Function App oluşturuyoruz.

<figure><img src="../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

* Azure hesabımız altında bir Function App oluşturduk. Şimdi vscode uygulamamızı tekrar açıp, projemizi azure da oluşturduğumuz "functiondemo0001" Function App 'ine push ediyoruz.

```powershell
func azure functionapp publish functiondemo0001
```

<figure><img src="../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

* Uygulamamız push edildi ve functiondemo0001 Function App içerisinde uygulamamız gözüküyor.

<figure><img src="../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>

* Şimdi bir önemli kısımda şu; hatırlarsanız biz storage account bilgisini, **local.settings.json** dosyası içerisine eklemiştik. Fakat oraya eklediğimiz bilgi sadece lokal development ortamları için geçerli. Biz bu storage account connection string bilgisini, Function App altında, Environment Variables kısmından eklemeliyiz.

<figure><img src="../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

* Değişkenimizi ekledik ve Apply butonuna bastık. Şimdi testlerimizi yapabiliriz.
* İlk olarak, Function App Overview kısmından, "main" adındaki uygulamamıza tıklayıp, Code + Test menüsüne gelip, "Get function URL" butonuna basıp, istek yapacağımız adresi kopyalıyoruz.

<figure><img src="../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

* Ardından kopyaladığımız url'e istek yaptığımız da, container 'ın başarılı bir şekilde oluştuğunu görüyoruz.

<figure><img src="../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

***

{% embed url="https://learn.microsoft.com/en-us/azure/azure-functions/" %}
