# 🔭 CloudWatch

![](../.gitbook/assets/product-page-diagram\_Cloudwatch\_v4.55c15d1cc086395cbd5ad279a2f1fc37e8452e77.png)

* AWS dünyasının monitoring hizmeti.
* Sunucu kaynak kullanımlarını izler ve raporlar. Buna ek olarak uygulamalarımızı kısıtlı da olsa izler.
* S3 bucketlarımızın boyutunu içerisinde kaç tane obje olduğunu raporlar.
* Cloudwatch izlenilen servisler, sunucular için dashboard'lar yaratmamıza izin verir.
* Kaynak kullanımı gibi tuaf bir durum farketmesi halinde bizi bilgilendirebilir.
* Önceden belirlediğimiz bir durum gerçekleşirse, SNS ile bize bilgi verebilir.
* CloudWatch ile işlemleri tetikleyebiliriz. Misal kaynak kullanımı %70 in üzerine çıkarsa yeni sunucu kur vb. gibi taleplerde bulunabiliriz.

Yeni bir sunucu oluştururken, enable cloudwatch detailed monitoring kısmını işaretliyoruz. Ve 1 DK içerisinde sunucudan toplanan veriler cloudwatch'e yansıyacak. Panelden cloudwatch'e geçip, Dashboard'dan kendimiz için bir grafik oluşturabiliriz. S3 bucket da bulunan obje sayısını öğrenmek için number widget ekleyebiliriz. All metrics kısmından, hangi servis için grafiği ekleyeceksek, bunu seçiyoruz.

#### Alarm oluşturmak için,

* Create alarm
* metrics  : EC2  - Per instance > Tek sunucu. ve ya, Access All instance > Tüm sunucular.
* Name : İlkalarm
* whenever : Alarm ne zaman tetiklenecek?
* is : >=80 > Cpu kullanımı %80ni geçerse.
* for : 1 datapoint : 1 defa tekrarlarsa. (datapoint 5dk)
* Thread Missing Data as : cloudwatch servisi veriye erişemezse, bunu nasıl uygulayıp, cevap versin. Örn, sanal sunucu kapandı.
* Missing : Veriyi alamadım olarak işaretle.
* ignore : Bunu önemseme.
* Bad : Sunucuya ulaşamazsan alarm üret.
* Good : Alarm'a gerek yok.
* Missing seçebiliriz.
* Actions kısmında,
* Whenever this alarm : State is alarm.
* Send Notification to : Nereye alarm gönderilsin? > New list > Topic Name = Alarmlar > alarm@onurbolatoglu.com ' a alarm gönder.
* AutoScaling : %80ni geçtiğinde, yeni sunucu oluştur gibi kurallar oluşturabiliriz.
* EC2 Action kısmına gelip,
* Whenever this alarm : state is alarm
* Take this action : reboot this instance
* Sunucuda böylelikle %80den fazla cpu kullanımı olduğu zaman mail gönderecek.&#x20;
* Alarm : Belirlediğimiz kriterlere göre bize haber verir.

#### Events,

Bir durum tetiklenirse, o duruma göre git başka bir şey yap dediğimiz kısımdır.

#### Misal,

* Events > Event pattern
* Service Name : EC2
* Event Type :  EC2 instance state-change notification > Spescific state > Stopped.&#x20;
* Any instance > Herhangi bir sunucu.&#x20;
* Specific instance > Burada bir sunucu seçebiliriz.
* Any olarak devam edebiliriz.
* Eğer bir sunucum power-off olursa,
* Targets > SNS topic ile alarm gönder. Diyebiliyoruz.
* Events > Belirlediğimiz bir olay olursa, o olay ile ilgili belirlediğimiz Targets işlemini uygula.

#### Configure Details,

* Name : ilkrule
* Create rule
* Sunucu kapandığında alarm oluşacak.

Dashboard yaratmadan, metrics kısmından hızlıca grafikleri görebiliriz. Cloudwatch üzerinde, ram kullanımı göremeyiz. Örnek olarak sunucuda gerekli ayarları yapıp biz bunu cloudwatch 'a iletebiliriz.

#### Logs,

Sunucularımız içerisinde çalışan (nginx vb) uygulamaları servislerin access - error vs loglarını buradan görüntüleyebiliriz. Burada kendimize bir log grubu yaratıp, sunucuya giriş yapıp logların bulunduğu dosyayı, cloudwatch 'a düzenli olarak gönderebiliriz.

* Sunucuların, cloudwatch servisine log gönderebilmeleri için, IAM rolü yaratmamız gerekiyor.
* IAM
* Roles
* Create Role
* AWS Service > EC2
* Next
* CloudWatch Agent Admin policy
* Next
* Tag girebiliriz.
* Role Name: cwagent
* Create role
* EC2 bölümüne girip, Kuralı atayacağımız sunucuya "actions" kısmından attach/replace IAM role seçiyoruz. Oluşturduğumuz "role" seçiyoruz. Bu sayede CW servisine dosya gönderebileceğiz.

#### Logs Grubu oluşturmak,

* Cloudwatch > logs > get started > create log grup.
* Log group name : ilkgroup

Log göndereceğimiz sunucuya bağlanıp, cw agent kuruyoruz. Gerekli yapılandırmadan sonra sunucu loglarını "cw-logs" a iletilecektir.
