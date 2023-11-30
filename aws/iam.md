---
description: Identity and Access Management
---

# 🆔 IAM

IAM AWS servislerine ve kaynaklarına erişimi güvenli bir şekilde sağlar ve yönetmemize yarar. IAM kullanarak, AWS grup ve kullanıcılar oluşturabilir ve yönetebiliriz. AWS kaynaklarına izin verip,  ve ya reddedebiliriz.  IAM tüm regionları kapsar, her region için ayrı ayrı kural yazma, grup oluşturma, kullanıcı oluşturmak durumunda değiliz.

![](<../.gitbook/assets/intro-diagram \_policies\_800.png>)

#### IAM Objeleri,

* User
* Group
* Role
* Policy

{% hint style="info" %}
Root, en yetkili kullanıcıdır.
{% endhint %}

Gruplar kullanıcıları bağlandığı objelerdir. Gruplara policy bağlayarak kullanıcıları, bu grubu eklediğimizde kullanıcılara tek tek policy atamak zorunda kalmayız.

* Policy, Kimin nereye, hangi yetkilerle erişebileceğini belirttiğimiz kurallardır. Dilersek kendimiz özelleştirebiliriz. (json ve AWS editör kullanarak) Ve ya hazır kuralları (policy) seçebiliriz.
* Role, AWS kaynaklarımıza, diğer aws kaynaklarını ya da farklı AWS kullanıcılarının hesabımıza nerelere ve hangi yetkilerle erişebileceğini belirlediğimiz yetkilendirme sistemidir. Misal, EC2 sunucumuzun S3 bucket'ına erişebilmesi için role yaratabiliriz. Ve ya farklı AWS hesabına bizim kaynaklarımıza erişip işlem yapabilmesi için role oluşturabiliriz. Role 2'ye ayrılır;
  * Kim tarafından bu role kullanılacak? Bu rol nereye atanacak?
  * Bu rolün atandığı kaynağın, hangi yetkilere sahip olacağı, neler yapacağı belirlenir.

IAM, merkezi AWS hesap yönetim servisidir. Tüm hesap ayarları bu servis arayıcılığıyla yürütülür. user,group,role,Policy yaratıp yönetmemizi sağlar. Kullanıcıların her türlü şifre kısıtlamaları ve MFA ayarlamaları IAM ile yapılır. Dış kimlik doğrulamaları "Active Directory" kurulacak bağlantılar IAM üzerinden yönetilir.&#x20;

IAM User login bağlantısını IAM üzerinden değiştirebiliriz.

#### MFA

MFA hesabımıza güvenlik katmanı eklememizi sağlar. (2. factor authentication)

* Virtual MFA, mobil cihazlara yüklediğimiz yazılımlar ile kullanabiliriz.
* U2F, Ubikey anahtarlar ile kullanabiliriz.
* Other Hardware, Farklı üreticiler tarafından üretilmiş, token, rakam üreten uygulamalar ile.

Password Policy, Kullanıcılar için parola ayarları belirleyebiliriz. (parola uzunluğu, özel karakterler)

#### IAM User oluştururken,

* Programmatic Access, AWS cli, SDK, Api erişimi için.
* AWS Management Console, web konsola erişim için.

{% hint style="info" %}
Her kullanıcı oluştururken, programmatic Access seçersek, AWS bize her kullanıcı oluştururken Access key oluşturacak.
{% endhint %}

![](../.gitbook/assets/2021-08-15\_18-12-51.webp)

#### Policy,

* Add user to group, Kullanıcıyı gruba ekleyebiliriz. Böylelikle kullanıcı policy'leri bağlı olduğu gruptan alır.&#x20;
* Copy Permission From user, Mevcut olan kullanıcının yetkilerinden kopyala.
* Attach exiting Policy,  AWS 'nin hazır policy'lerini seçebiliriz.

{% hint style="info" %}
Policy aslında birer json dosyalarıdır.
{% endhint %}

Tags, kullanıcılar için etiket atayabiliriz. Misal,

| Key       | Value      |
| --------- | ---------- |
| departman | bilgiislem |
| lokasyon  | istanbul   |

{% hint style="info" %}
Access key ve secret Access key unutmamak için CSV dosyasını indirebiliriz.
{% endhint %}

Yaratılan kullanıcıya, bilgileri mail olarak iletebiliriz. (Send Email) IAM kullanıcıları IAM login bağlantısı üzerinden konsola giriş yapabilirler.&#x20;

#### Group,

Grupta bir kural değiştirdiğimizde, gruba bağlı olan tüm kullanıcıları etkiler.&#x20;

#### Policies,

Policies, 2 şekilde yaratılabilir. Editör ile json oluşturabilir ve ya kendi json policy dosyamızı yükleyebiliriz.&#x20;

* [x] Service : Erişim olacak servis hangisi ?
* [x] Actions : Yetki ne olacak ?
* [x] Resources : Hangi kaynaklarda geçerli olacak ?
* [x] Request Condition :&#x20;
  * [x] MFA Required. Policy 'nin atanmış olduğu kullanıcının MFA devre de değilse, Verdiğimiz yetkilere sahip olamaz.&#x20;
  * [x] Source IP : Kullanıcıların, IP adreslerini belirleyebiliriz. Başka bir IP (Burada belirtmediğimiz) ile gelirse yetkileri engelle diyebiliriz.
* [x] Policy create edip, kullanıcılara bağlayabiliriz.

#### Role,

* Rolü seç.
* AWS Service : Bir AWS servisinin başka bir AWS servisine ulaşmasını istiyorsak, bunu seçiyoruz.&#x20;
* Another AWS Account : Başka bir AWS hesabının, bu hesaba erişmesini istiyorsak, bunu seçiyoruz.
* Web identy : Cognito seçeneği ve ya openID provider seçeneği kullanabiliriz.
* SAML 2.0 : Kendi Active Directory kullanıcılarımızın bu hesaba bağlanmalarını istiyorsak, bu seçeneği seçiyoruz.
* AWS Service Seçiyoruz.
* EC2 Seçiyoruz.
* Next
* 2.kısımda bir önceki kısımda belirlediğimiz servisin yetkilerini belirtiyoruz.
* Misal, S3FullAccess , EC2 kaynağımız için, s3'e full erişim verir.
* Tag ekleyebiliriz.
* Role isim girebiliriz.
* Create role

Böylelikle EC2 sunucular, tam yetki ile S3 servisine bağlanabilir.

#### Fatura Alarm Oluşturma,

* My biling dashboard
* Budgets
* Create budgets&#x20;

{% hint style="info" %}
Budget üzerinde alarmlar oluşturabiliriz.
{% endhint %}

{% hint style="info" %}
Cost Budget, Fiyatlandırma ile ilgili alarm.

Usage Budget, Kullanım saati ile ilgili alarm.

Reservation, Rezerve edilen kaynaklar ile ilgili.
{% endhint %}

* Cost Budget
* Name : Faturaalarm
  * montly => period
  * 10 => Budget Amount
  * Recuring Budget => Her ay kontrol et.
  * Expring Budget => Bitiş zamanı ayarla.
* Filtering : Dilersek, bütün servisleri kontrol ettirebiliriz. (Varsayılan) dilersek, belirlediğimiz servisleri kontrol ettirebiliriz.
* Advanced Options, default kalabilir.
* Configure alert.
  * Actual Costs : Sürekli AWS kontrol edecek, aşağıda belirlediğimiz thresold geçerse, alarm gönderir.
  * Forecoster Costs : Kaynakların ne kadar para çıkaracağını, kontrol edip alarm oluşturur.
  * Alert Thresold : 10$ %80 doldurulursa bilgi verir. Yukarıda $10 olarak belirtmiştik.
  * Email Contacts : Alarmın gelmesini istediğimiz mail adresini giriyoruz.
* Finish.
