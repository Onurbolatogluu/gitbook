---
description: AWS Kayıt Defteri
---

# 👀 CloudTrail

{% hint style="info" %}
API : Uygulamaların ve ya servislerin diğer uygulamalar ya da servisler ile konuşmasına\
olanak sağlayan giriş-çıkış kapısı olarak özetleyebiliriz.
{% endhint %}

AWS servislerini, hesabımızı yönetmek için, Web console, aws-cli, aws development kit gibi araçları kullanırız. 3 aracında arka tarafta yaptığı aslında api çağrılarıdır. Örnek bir sunucu kurduğumuzda finish dediğimizde, EC2 servisine API çağrısı yapılır.

![](../.gitbook/assets/product-page-diagram\_AWS-CloudTrail\_HIW@2x.d314033178a16dbbd99111038789685e42f23278.png)

#### CloudTrail,

* Hesabımızda yapılan her işlemin hangi kullanıcı tarafından, hangi tarihte, hangi IP adresinden yapıldığının kaydını tutar.
* Çoklu AWS hesabı olan X firmasında, hangi işin, kimin tarafından yapıldığını öğrenmek için çok kullanışlı bir servistir.
* AWS hesabımızın, yönetim, uyumluluk ve operasyonel risk denetimi sağlar.

Cloudtrail dashboard bize 90 günlük işlemleri gösterir. Event history geçmiş işlemleri görüntülememizi sağlar. Daha uzun süre loglar oluşturmak ve görüntülemek için yeni bir trail oluşturabiliriz.

* Trails > Create Trail
* Trail Name: Deneme
* Apply Trail to All regions : Bu trail tüm regionlara iletilsin mi? Yes. Böylelikle tüm regionlarda bulunan işlemlerin kaydını tutacak.
* Management Events : Yönetimsel olayların nasıl tutulmasını istiyoruz? All.
* Data Events : S3, bucket da bulunan, objelerin değişikliklerinin kaydı tutulsun mu?
* Storage Location : Yaratılan loglar nerede tutulsun? Yeni bucket yaratabiliriz. Dilersek mevcut bir bucket a gönderebiliriz.
* Create.

Loglar oluştukça, seçtiğimiz bucket'a yazılacaktır. Logları S3 üzerinden görebiliriz.
