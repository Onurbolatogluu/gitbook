# 📐 AWSConfig

AWS config aws kaynaklarımızın, konfigürasyonlarını denetlememizi ve değerlendirmemizi sağlayan bir hizmettir. Config, AWS kaynak konfigürasyonlarımızı sürekli olarak izler ve kaydeder. İstenen konfigürasyonlara karşı izlenen konfigürasyonların değerlendirilmesini otomatikleştirmemizi sağlar.

![](../.gitbook/assets/product-page-diagram-Config\_how-it-works.bd28728a9066c55d7ee69c0a655109001462e25b.png)

&#x20;

Bütünlük ve uyumluluk denetimi sağlar. Örneğin, dün çalışan sunucu/servis bugün çalışmıyorsa, config servisine giderek dünden-bugüne bu kaynakta hangi değişiklikler olmuş görebiliriz. Sec-Group değişikliği dahi not edilir ve kaydedilir. Misal yaratılmış tüm EBS volume'lerini bir sanal sunucuya takılı olması kuralı yaratır ve config servisinin sanal sunucuya mount olmayan ve boşta duran bir EBS volume var ise, uyumsuzluk vermesini sağlayabiliriz.

#### Uygulama,

* Bir sunucu yaratalım, ardından elastic IP rezerve edelim.  Sonrasında bu IP adresini sunucuya atayalım.
* Config servisine geçelim.
* Resource type to record : Neleri izlemek istiyoruz?
* Amazon S3 Bucket : İzlemeleri nereye depolamak istiyoruz?
* Amazon SNS Topic : Değişiklikler bildirim olarak iletilsin mi?
* AWS Config Role : Config servsinin kaynaklara erişmesi için, role yaratılabilir ve ya mevcut role kullanılabilir.
* Next
* Config Rules : Burada kontrol edilmesini istediğimiz servisleri seçiyoruz.
  * örn, eip elastic ıp - eip attach
  * Böylelikle hesabımızda bulunan Elastic IP adresleri bir sunucu da bağlı değilse uyarı oluşturulacak.
* Next.

Sunucu da test için bir kaç değişiklik yapıp, volume ekleme vb.. Ardından tüm bu değişiklikleri ve elastic IP adresi mount değilse 1 saat içerisinde AWS config dashboard da görebileceğiz.
