# 💰 Environment Variables

Konuya bir örnekle başlayalım. Misal, biz bir web uygulaması geliştiriyoruz. Ve bu web uygulaması, bir veritabanı uygulamasına bağlanarak verileri burada saklıyor. Bu uygulamanın, bağlandığı veri tabanının, sunucu adresi, kullanıcı adı ve şifre bilgilerini de bu uygulamanın içerisine gömdük diyelim. Daha sonra, bu uygulamayı container imajı haline getirdik ve artık istediğimiz container platformunda bu uygulamayı çalıştırmaya hazırız.

Bu senaryo da 2 sıkıntımız bulunmaktadır.

1 - Biz veritabanı bağlantı bilgilerini, container imajı içerisine hardcoded olarak yazdık. Yani bir şekilde, container imajı istemediğimiz insanların eline geçerse, bu imajın içerisinde bulunan bu bilgiler expose olur. Bu durumu asla istemeyiz.

2 - Bu(web) imajdan, container oluşturmak istediğimiz zaman, bu container her defasında, aynı veritabanına, aynı kullanıcı adı ve şifre ile bağlanmaya çalışacak. Veritabanı adresimiz, test ortamlarında ayrı, prod ortamlarında ayrı olabilir. Biz bu kullanıcı adı ve şifre bilgilerini zaman içerisinde güncellemiş olabiliriz. Bu senaryo da ne yapacağız? Her ortam için, her seferinde, yeni bilgilerle imaj oluşturacağız. yada her seferinde, container oluşturulduktan sonra, container 'a bağlanıp, bu bilgileri güncelleyeceğiz. 2. seçeneği seçtiğimizde bu sefer de, şöyle bir sorunumuz olacak. Kubernetes bir sorun anında, containerı silip, yeniden oluşturabilir. Scale ediyoruz, geri alıyoruz vb. Her seferinde oluşturulan, tüm containerlara bağlanıp, bunu manuel mi düzelteceğiz?

Gördüğünüz gibi, bu tarz ortama göre değişebilen, ve hassas bilgileri container imajı içerisine gömmek, oldukça fazla sorun çıkartır. Fakat bununda bir çözümü var.

Bu tarz, çalıştırdığımız ortama göre değişebilecek bilgileri container imajına hardcoded olarak eklemeyiz. Bunun yerine, uygulamalarımızda bu bilgileri, çalıştırıldıkları sistemden okuyacakları şekilde, bir değişken olarak tanımlarız. Yani Environment variables _'dan okunması için tanım yaparız._

Yukarıdaki örnekte uygulamamızın bağlanacağı, veritabanının adres ve kullanıcı bilgilerini direkt uygulamaya gömmek yerine veritabanı adresini $database . Kullanıcı adı bilgisini $username . Parola bilgisini de, $password isimli değişkenlerin değerlerine bakacak ve oradan bu değerleri okuyacak şekilde ayarlamasını söyleriz.

Bu sayede, bu hassas bilgileri, bu imajlara gömmemiş oluruz. Ne zaman bu imajdan, container oluşturmak istersek, o an hangi değerlerin atanması gerekiyorsa, bu değerlerin Container da Environment variable olarak tanımlarız. Böylecek Environment variables nedir ve Container ile ne alakası var sorularının cevaplarının üstünden geçmiş olduk.

Şimdi bunları pod tanımına nasıl ekleyeceğimize bakalım.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envpod
  labels:
    app: frontend
spec:
  containers:
  - name: envpod
    image: nginx
    ports:
    - containerPort: 80
    env:
      - name: USER
        value: "Onur"
      - name: database
        value: "testdb.example.com"
```

Pod tanımları içerisinde, Environment variable'la&#x72;_, container tanımı altında, env parametresi ile tanımlanıyor. env parametresi altında list şeklinde, tüm_ Environment variable 'larımızı tanımlıyoruz.

Öncelikle tanımlamak istediğimiz Environment variable 'ın ismini giriyor ve ardından da, value opsiyonu ile buna atamak istediğimiz değeri giriyoruz.

Yukarıdaki örnek pod tanımında, "USER" isimli Environment variable tanımlayıp, ve buna da, "Onur" değerini atadım.&#x20;

Bir altındaki Environment variable 'da,  "database" isimli Environment variable tanımlayıp, buna da "testdb.example.com" değerini atadım.&#x20;

Bu container oluşturulduğunda, bu 2 Environment variable bu containerda tanımlanacak, Aynen az önceki örnekte belirttiğim gibi bir web uygulaması oluşturdum. Web uygulaması içerisinde, Selamlama yapacağı kişinin kim olacağını, hardcoded olarak eklemek yerine, çalıştığı ortamdaki "USER" isimli  Environment variable değeri ne ise, ondan bu değeri alıp, selamlayacak.

```bash
root@medellin-master:/home/onur# curl 127.0.0.1:8080

<html>
<body style="background-color:gray;">

<center><h1 style="color:magenta;">
    <p>K8S Fundamentals</p>
    <p>Pod: envpod</p>
    <p>Hello Onur</p>
        </h1>
</center>

</body>
```

envpod isimli pod içerisinde bulunan Environment variable'ları listelemek,

```bash
kubectl exec envpod -- printenv
```

Bu bir web uygulaması podu, normal bu pod'a dış dünyadan erişmek için, ya loadbalancer yada nodeport şeklinde bir servis tanımlamamız gerekir. Fakat kubernetes bize, şu imkanıda sunuyor. Biz kubectl kullanarak, herhangi bir pod 'a, deployment'a, service. vb. Kendi bilgisayarımızdan tünel açarak ilgili objenin portuna trafiği yönlendirebiliyoruz. Testlerde hızlı şekilde, objelerimize bağlanmak için, kullanabildiğimiz bu özelliğe "port forward" diyoruz.

Syntax:

```bash
kubectl port-forward {obje tipi/obje adı} hostport:podport
```

Örnek:

Kubectl 'i çalıştırdığımız cihazın, 8080 portuna gelen tüm istekleri, envpod isimli podun 80 portuna gönder/yönlendir.

```bash
kubectl port-forward pod/podenv 8080:80
```

{% hint style="info" %}
Container tanımı altında girdiğimiz, _Environment variable'ları, Bu container içerisinde,_

_farklı yerlerde dilediğimiz şekilde kullanabiliriz._
{% endhint %}
