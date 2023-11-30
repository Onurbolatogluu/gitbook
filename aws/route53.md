# ❤ Route53

AWS tarafından sunulan, yönetilebilen DNS hizmeti. Aynı zaman da domain register edebiliyoruz.

Tüm AWS servisleri ile entegre durumdadır. Api desteğine sahip, uygulamalarımız otomatik olarak ihtiyacı olan kayıtları oluşturabilir.

Route policy ve healthcheck servisi arayıcılığıyla load balancing işlemi yapabiliyoruz. Gelen istekler coğrafi olarak yönlendirilebilir. Healthcheck servise gelen istek'de sorun varsa failover senaryoları kurabiliriz.

#### Route53 de iki çeşit zone mevcuttur;

Public Zone,

* Tüm dünyadan erişim yapılabilir.
* Alan adının gerçekten var olması gerekmektedir.
* Böyleye tüm dünyadan domainimizin IP adresini öğrenmek isteyenlere bu zone cevap verecektir.

Private Zone,

* Lokalde sadece VPC üzerinden erişim yapılır.
* Alan adı gerekmez, herhangi bir isimlendirme seçilebilir.
* VPC de kaynakların IP adresleri yerine, isimlerle haberleşme imkanı verir.

#### Private zone oluşturmak için;

* Hosted zone
* Create hosted zone
* onur.local > Kullanacağımız lokal domainimizi yazıyoruz.
* Type, Private
* Hangi VPC için bu zone oluşturulacaksa seçmeliyiz.

Böylelikle VPC içerisinde bulunan sunucular private kayıtları çözerler.

Private zone sadece seçtiğimiz VPC içerisinde bulunan sunucular erişebilecekler.&#x20;

SOA : Domain için temel kayıtları barındırır.

NS : DNS sunucuların listesidir. Domainimiz burada yazan NS'ler tarafından yönetilecektir.

TTL : Kayıtlar kaç saniye, kaç dakika boyunca tekrar istemcilere iletildiğini belirtiyoruz. Örneğin 60 dakika TTL süresi olan bir kayıt değiştirilmişse bile 60 dakika sonra kullanıcılar bunu yeni kayda göre işleyebileceklerdir.

Alias : Alias kaydı ile bir kaydı, AWS servislerine / kaynaklarına bağlayabiliyoruz.

#### Health Check Yaratmak,

* Create Health check
* Name, misal irlweb1-abdweb1
* What to Monitor : Endpoint
* IP address : IP adresi giriyoruz. Misal irlweb1 IP adresini. ve abdweb1 IP adresini.
* Port : 80
* Next : Böylelikle irlanda ve ABD de de bulunan sunucumuz için healthcheck yarattık.

#### Public Zone Eklemek,

* websitesi - domain.com
* Create record set
* name  : domain.com
* Type : A - IPv4
* Value : 1.2.3.4 - 5.6.7.8 > Birden çok IP eklenebilir.

Route Policy : DNS üzerinde basit bir load balancing yapabiliriz.\
Weighted  : Gelen istekleri böler.&#x20;

Örnek olarak, domain.com için farklı IPlere sahip 2 kayıt yaratalım, Kayıtları oluştururken weight kısmını birini 10 diğerine 90 yazalım. Böylelikle 10 isteğin 9'u 90 yazdığımız host'a gidecek. 1 istek istek ise 10 yazdığımız host'a gidecek. (Değerler %liktir)

Letancy : Route53 dns sorgusu atan kişi, hangi lokasyona daha kısa sürede eriştiğine karar verip, ona göre uygun sunucuya yönlendirdiği routing şeklidir.

#### Letancy yapılandırma,

* Create record set
* Name : domain.com
* value   : 1.1.1.1 > avrupada ki sunucumuzun IP adresi.
* Routing policy : latency
* Region : Sunucu hangi region da bunu seçiyoruz. Çoğu durumda bunu otomatik seçer.
* Set ID : webserverEU
* Create

#### ABD için,

* Name : domain.com
* Value : 2.2.2.2 > ABDde de bulunan sunucunun IP adresi.
* Routing Policy : Latency&#x20;
* Set ID : webserverUS
* Region : us-east-1 > kaydını eklediğimiz sunucunun region'nunu belirtiyoruz.

Örnek olarak, bir kullanıcı Belçika'dan istek yaparsa Route53 bu isteği Avrupa da bulunan sunucumuza iletecek.



#### Geolocation Oluşturmak,

Geolocation : Örneğin ABD den, istek gönderen kişi bu sunucuya.. Avrupa'dan istek gönderen kişi şu sunucuya gitsin diyebiliyoruz.

#### 1.Kayıt

* Create record set
* Name : domainx.com
* Value : 1.1.1.1 > Avrupa da bulunan sunucumuzun IP adresi.
* Routing Policy : Geolocation
* Location : Europe, Eklediğimiz  kayıt hangi ülkede ki ve kıtadaki kullanıcılara etki edecekse bunu belirtiyoruz.
* SET ID : webserverEU

#### 2. Kayıt

* Name : domainx.com
* Value : 2.2.2.2 > ABD'de de bulunan sunucunun IP adresi.
* Routing Policy : geolocation
* Location : North America
* SET ID : webserverUS

#### 3.Kayıt

* Name : domainx.com
* Value : 1.2.3.5 > İrlanda da bulunan sunucunun IP adresi.
* Routing Policy : Geolocation
* Location : Default
* SET ID  : webserverEU2

1. kayıt da kullanıcılar  Avrupa'dan istek atarsa 1.1.1.1 'e gidecek.
2. kayıt da kullanıcılar Amerika 'dan istek gelirse, 2.2.2.2'ye gidecek.
3. kayıt da diğer tüm bölgelerden gelen istekleri 1.2.3.5'e gidecek.

Failover, birden fazla IP girerek eğer 1. sunucuya erişilemezse 2.IP adresine yönlendir diyebiliriz. Burada ilk yarattığımız Health check'ler işe yarıyor.

#### Failover oluşturma,

* Name : domaina.com
* Value : 1.1.1.1 > İrlanda da bulunan sunucunun IP adresini yazıyoruz.
* Routing Policy : Failover
* Failover Record Type : Bu kayıt primary kayıt mı olacak? secondary kayıt mı olacak? Bunu yazıyoruz.
* SET ID : domaina-primary
* Associate With health check : Yes > Primary sunucuyu seçiyoruz. Burada isteklerde bu sunucuyu kontrol et sunucuda problem yoksa, ilk IP yani bu IP adresine istekleri ilet. Eğer problem varsa, secondary sunucuya yönlendir.

#### 2.Kayıt

* Name : domaina.com
* Value : 2.2.2.2 > Secondary sunucu IP adresi
* Routing Policy : Failover
* Failover Record Type : Secondary > Bu kayıt secondary sunucuya ait.
* SET ID : domaina-secondary
* Associate with health check : yes > Secondary sunucuyu seç.
* Create

Failover oluşturmuş olduk.

#### Trafik Policy Oluşturmak,

* Policy name : deneme
* Version desc : boş bırakabiliriz.
* Ardından hangi tip dns kaydı yaratmak istediğimizi seçiyoruz.
* A kaydı
* Connect to : A kaydını nereye bağlamak istiyoruz.
* Geolocation seçiyoruz.  Location : Europe . Avrupa'dan gelenleri connect to 1.1.1.1 'e yönlendir. Location : North America connect to 2.2.2.2 'ye yönlendir.

#### Ve ya,

* Location  : Europe
* Connect to : weight
* 10 weight : connect to : 1.1.1.1 endpoint
* 90 weight : connect to : 2.2.2.2 endpoint
* Location : North America
* Connect to : New Endpoint
* Endpoint type : value 2.2.2.2

{% embed url="https://docs.gitlab.com/ee/administration/geo/replication/location_aware_git_url.html" %}

{% embed url="https://www.ictshore.com/cloud/dns-geolocation-tutorial-aws" %}

