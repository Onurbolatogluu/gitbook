# 🚀 Lambda - Api Gateway

Lambda, Sunucu yönetmek zahmetine girmeden, kod çalıştırmanıza izin verir. Yalnızca tükettiğiniz hesaplama zamanı için ödeme yapılır. Kod çalışmadığında ücret alınmaz.&#x20;

![](../.gitbook/assets/0\_hxjzck3fCwXMy83L.png)

FAAS (serverless) : Developer 'lar için yarattıkları kod bloğunu, bloklarını atarak çeşitli yöntemlerle bu kodları tetikleleterek, kodun çalışması sağlanarak, sonucun alındığı bir konsept. Geliştiriciler için bir çok sorunu ortadan kaldırır. Ortamda bir sunucu olmadığından, sunucu yönetim problemi kalmıyor. Ayrıca kod çalışıp sonuç alındığında faturalandırma yapılıyor. Kullandığımız kadar ödeme yapıyoruz. Servis arka planda, ihtiyaca göre büyüyebiliyor. Yani EC2 'da autoscaling ile sağladığımız durumu burada otomatik olarak sağlıyoruz.

Kullanıcılar bir sunucu yönetmeden, uygulama geliştirmelerine ve işlevleri dağıtmalarına olanak sağlayan ve işlem verimliliğini arttıran bir bulut bilişim modelidir.  FAAS(Function as a Service) temelindeki konsept serverless(sunucusuz) olarak bilinen konsepttir. Bu konsept tüm bilişim alt yapısı bulut bilişim sağlayıcısı tarafından yönetilir ve bizler sadece kod ile ilgileniriz. Kodun çalışması ve sonucun gönderilmesi için gereken tüm alt yapı servis sağlayıcının görevidir.

Lambda, ile hemen hemen her türlü uygulama ve ya arka uç (backend) hizmeti için kod çalıştırabiliriz. Sadece kodumuzu yükleriz ve gerisini lambda'ya bırakırız. Kodumuzun diğer AWS hizmetlerinden otomatik olarak tetikletmek ve ya doğrudan her hangi bir web ve ya mobil uygulamadan çağırmak için ayarlayabiliriz. Python,Java,C#,NodeJS,Go dillerinde kodları çalıştırarak bize işlem sonucunu çıkaran AWS'nin FAAS servisidir.

Api Gateway, Ortamda birden fazla mikro servis bulunan ortamlarda tüm bu servislere erişimin, servislerin birbiriyle konuşmalarının dış dünyadan diğer uygulamaların bu servise hizmetlerine erişebilecek bir ortam oluşturulmasını sağlar. Tüm bu karışık servislerin giriş noktasıdır. Lambda ile mikro servisler hatta tek bir fonksiyon çalıştıran nano servisler yaratabiliriz. Ve bu servislere bir giriş noktası Auth alt yapısı kurmamız gerekebilir. İşte bize bunu Api GW servisi sağlıyor.&#x20;

Api Gateway, Geliştiricilerin, her hangi bir ölçekte API oluşturmasını,yayınlamasını,sürdürmesini ve izlemesini ve güvenliğini sağlamasını kolaylaştıran tamamen yönetilen servistir. AWS yönetim konusunda bir kaç tıklamayla Amazon Elastic Compute Cloud üzerinde çalışan iş yükleri AWS lambda üzerinde çalışan kod blokları ve ya herhangi bir web uygulması gibi arka uç hizmetlere erişmek için ön kapı olarak işlev gören bir API oluşturulabilir.

#### Lambda Fonksiyonu oluşturmak,

* Create Lambda function
* Autor from scratch : Sıfırdan kodu kendimiz mi yazıp mı kullanacağız?
* Use a blueprint : Blueprint kullanabiliriz. Blueprint aws 'nin hazırladığı, hazır lambda fonksiyonlarıdır.
* Browse serverless app repository : Marketplace olarak düşünebiliriz. İnsanlar yazdıkları kodu buraya yükler. Biz buradan kullanabiliyoruz.
* autor from scratch seçeneği ile devam edebiliriz.
* Function Name: Fonksiyon adı
* Runtime : Kodumuz hangi dil'de yazıldıysa buradan seçmeliyiz.
* Permission : Bu fonksiyonu hangi permission (aws haklarıyla) çalıştırmak istediğimizi soruyor.
* Function code : Bu ekrana kodumuzu yapıştırabiliriz.&#x20;
* Policy Template : Hazır bir template'den  permission seçiyoruz.&#x20;
  * Role Name giriyoruz.
* Create Function

Dilersek bu kodu tetikletebiliriz, örneğin s3'e dosya atıldığında bu kodu çalıştır vb. Dilersek manuel olarak çalıştırabiliriz.

* \=> Test
* \=> Create new test event
* \=> Event Name
* \=> Diğer ayarlar varsayılan kalabilir.
* \=> Save
* \=> Test

Böylelikle oluşturduğumuz kodumuz çalışmış olacak.

\=> Basic Settings, kodumuzun çalışması için gerekli memory miktarını seçebiliriz.

\=> Timeout : Kod çalışırken kaç saniye içerisinde timeout 'a düşsün. Uzun sonuçlar çıkaran, fonksiyonlar için gerekli timeout seçeneğini ayarlayabiliriz.

\=> Execution Role : Daha önceden atadığımız rolleri değiştirebiliyoruz.&#x20;

\=> Network : Bir VPC içerisinden erişilebilir hale getirebiliyoruz.

\=> Debug - Error Handling :  Logları SNS ve ya SQS e gönderebiliriz.&#x20;

\=> Concurrency : Kodun arka arkaya çalışma limitini belirleyebiliriz.

Kodumuzu tetikletmek ve dış dünyadan erişmek için API GW servisine geçiyoruz.

Api GW, rest-api yaratmamzı sağlar. Bu servis sayesinde HTTP protokolü üzerinden çalışan API yaratabiliriz ve bu API yarattığımız, mikro servislere bağlayabiliriz. Ayrıca lambda üzerinde yarattığımız birer mikro servis görevi gördüğünden yaratacağımız API bu fonksiyona bağlanabilir.

API ile her /get isteği gönderildiğinde bunun lambda servisinde az önceki oluşturduğumuz fonksiyonu tetikletecek şekilde ayarlayacağız. Ve sonucu geri döndüreceğiz.

* Choose the Protocol : Hangi tip API yaratacağız?
  * Rest
* Create New API : Yeni API oluşturabilir ve ya örnek bir API kullanabiliriz.
  * New API
* Name : ilkapi
* Endpoint : Regional (Lambda fonksiyonu bu region'da)
* Create API



* Resources => Actions : Fonksiyonumuzu tetikletmek için metot yaratalım.
  * Create Method => Get => Okey
* Bu metot çağrıldığında ne tetiklenecek?
  * Integration Type : Lambda Function
* Lambda Region : Lamdayı oluşturduğumuz region?
* Lambda Function : Fonksiyonumuzun adı?
* Use Default Timeout : Seçili.
* Böylelikle işlemler tamamlandı.

Deploy API => Stage Name : Misal, " Prod " yazıyoruz.  İlk api yaratmış olduk.
