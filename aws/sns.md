---
description: Simple Notification Service
---

# 🔕 SNS

AWS Duyuru servisi, SNS 3 temel yapıdan oluşur.

![](../.gitbook/assets/Product-page-diagram-Amazon-SNS\_event-driven-SNS-compute@2X\_.4b9c0a75aa40bda9cdb12f0176930a12da2872bf.png)

Publisher : Mesaj gönderen ya da mesaj gönderimini tetikleyen kaynaktır. Örn: cloudwatch\
Publisher yayınlamak istediği mesajı TOPIC 'e iletir.

Topic : Yayınlamak istediğimiz mesajın hedef kitlesini bir arada topladığımız basit klasörlerdir. Publisher mesajı yazar, bu mesajı topic üzerinden publish eder yani yayınlar. Daha sonra SNS mesaj bildirimlerine abone olmuş kişilere mesajı iletir. Topic 'lere abone olmuş çeşitli kaynaklar vardır. Örn, Bir email adresi ile topic 'e abone olunabilir. Bu mesaj e-mail adresine iletilebilir.

Subscriber : SQS Amazon - Lambda - SMS - Email - HTTP/HTTPS vb duyuru servis üyeleri olabilir. Örneğin kullanıcılar telefonlara yaptığımız uygulamayı yüklerse bildirim gönderebiliriz.

#### SNS Oluşturmak,

* Topics : Create topic
* Name : Alarm
* Display Name : Mail de, bildirimde buraya yazdığımız isim gözükecek.
* Encryption : Bu topic'i şifrelemek istiyor muyuz?
* Access Policy :&#x20;
* Advanced : Topic mesajlarını json dilinde kendimiz yazabiliyoruz.
* Basic

Topic Bildirimini kim gönderecek?

* Only the topic owner : Sadece topic'i yaratan kişiler, bu topic'e mesaj gönderebilir.
* Everyone : Herhangi bir kimse mesaj gönderebilir.
* Only the specifed AWS accounts : Sadece seçtiğimiz AWS hesapları mesaj gönderebilir.

Topic kime mesaj iletecek?

* Only the topic owner
* Everyone
* Only the specified AWS accounts
* Only requester with certain endpoints : @domain.com > domainine sahip kullanıcılar.

Delivery retry policy (HTTPs) : HTTPs endpointlere kaç defa bildirim göndermeyi denemesi gerekmektedir?

Delivery status logging : Bildirimleri karşı tarafa ulaşıp, ulaşmadığını izleyip, izlemediğimiz sorulmaktadır. "SMS ve Email mesajları buradan takip edilmiyor." Lambda-SQS-HTTPs-Platform Endpoint (app)

#### Topic oluşturmak,

* Create Topic
* Topics > Alarm > Create subscription
* Protocol : Hangi yöntem ile abone olmak istiyoruz. Email vb. Abone olacak mail adresine öncelikle onay maili gitmektedir ve bu mail onaylanmalıdır.
* Publish Message : Test mesajı gönderebiliriz.
* TTL : 100 yazarsak, 100 saniye boyunca mesaj iletilmediyse mesajı iptal et.
* Message Attributes : Genelde application bildirimleri için kullanılır. Api key vs.
* Publish Messages

Böylelikle cloudwatch'da rule yaratıp, örneğin sunucuların durumu değişirse (reboot vb) hedef (target) olarak SNS Topic kullanıp, oluşturduğumuz Topic'e üye olan kişilere bildirim ilet diyebiliriz.



#### Mobile,

Push Notification : Google,Apple,Microsoft vb push notification servisleri gibidir. Misal gönderinizi 5 kişi beğendi bildirimi gibi. Uygulamamızı yükleyen insanlara topic'imize üye yapabiliriz ve bildirim gönderebiliriz.

Text Messaging :&#x20;

* Prometonel : Promosyon mesajları için.
* Trasactional : Onay mesajlarıdır. (Gelen kodu gir vb)
* Phone number : SMS'in iletileceği numara.
* Sender ID : Karşı tarafın, yani SMS'in iletileceği kişinin gördüğü isim. (From kısmında gözükür)

Buraya eklediğimiz numaraları da topic'e üye yapabiliriz ve toplu mesaj gönderebiliriz.
