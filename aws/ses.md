---
description: Simple Email Services
---

# 📧 SES

E posta gönderim servisidir. Uygulamalar arayıcılığıyla mail göndermemizi sağlıyor. Örneğin EC2 üzerinde çalışan bir uygulamamız var, ve bir web mağazamız var. Bu web mağazasından alışveriş yapan her kullanıcıya bu alışveriş bilgilerini, bunun yanında da teslimat bilgilerini mail ile göndermek istiyoruz. Bunun SES ile yapabiliriz. Ek olarak pazarlama ve kampanya mailleri gönderebiliriz.

#### Servis Oluşturmak,

* identity management : Mail adreslerimizi ya da domainlerimizi ses servisine tanıtabiliriz. domain.com gibi. Örneğin buraya  (email adresini) onur@onurbolatoglu.com yazarak, ekleyerek from kısmı bu kullanıcı olur. Ayrıca domain olarak da ekleyebiliriz.
* Email statistics : Mail istatislikleri, üste bulunan sending limit incrase kısmını seçiyoruz. Böylelikle AWS spamı önlemek için mailleri incelemeye alıyor.
* Reputation Dashboard : Domain email gönderimi sağlığını görüntüleyebiliriz.
* Dedicated IP : Normal de maillerimiz paylaşımlı bir mail sunucudan gönderilir. Kendimize ait bir IP adresi talep edebiliriz.
* Configuration Sets : Gönderdiğimiz maillerin durumunu görmek için burayı kullanabiliriz. Cloudwatch,sns,kinesis gibi servisler de tutmak ve buna göre aksiyon alabiliriz.
* SMTP Settings : Uygulamalarımız mail gönderimi yapmak için, bu ayarlamaları yapabiliriz. Böylelikle (SES) servisini SMTP sunucu olarak da kullanabiliriz.
* suppression list Removal : Bounce (hata) olarak geri dönen mailleri gönderdiğimiz kişileri buraya ekler ve tekrar mail göndermemizi (hata aldığımızı kişi ve domainler) engeller. Bu ekran, bu adresleri listeden çıkarmamızı sağlar.
* Cross Account : Başka bir hesap adına gönderdiğimiz maillerle ilgili bilgi almak için kullanırız.
* Email Temp : Göndereceğimiz mailler standart aynı ise, sadece gönderdiğimiz kişilerin adı değişiyorsa bunları değişken olarak atıyoruz ve template oluşturuyoruz. Json tarzında bir template dosyası yaratıyoruz. Bu template cli arayıcılığıyla ses(upload) edebiliriz. Göndereceğimiz mailin json template oluşturulup gerekli bilgilerini girerek mail gönderebiliriz. ve bunu cli kullanarak yapabiliriz. From,to gibi gerekli bilgiler girmemiz şartıyla.

{% embed url="https://www.youtube.com/watch?ab_channel=KGPTalkie&v=qPSj9VPfd5c" %}
