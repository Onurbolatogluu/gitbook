---
description: Elastic Compute Cloud
---

# 🚙 EC2

On-Demand : Kullandığın kadar öde, taahhütsüz.

Reserved Instance : Önceden anlaşılmış rezerve edilmiş 1 yıl ve 3 yıl taahhütlü.

Spot Instance : Teklif üzerine edinilen cloud sunucular.

Dedicated Host : Kişiye özel sunucu, kaynakları paylaşımsız.



Sunucu oluştururken;

* [x] İşimize uygun sunucu türünü seçmek.
* [x] Bu sunucu ailesi içerisinden işimize uygun modeli bulmalıyız.
* [x] Bu modelin hangi versiyonunu kullanmalıyız. Bunu seçmeliyiz.
* [x] Kaynak boyutlarını seçmeliyiz.

EC2 Storage Tipleri;

| Instance Storage (ephemeral)           | Elatic Block Storage (EBS)                 |
| -------------------------------------- | ------------------------------------------ |
| Instance fiziksel olarak bağlı.        | Kalıcı veri deposu, instance'dan bağımsız. |
| Veriler başka bir yere replike değil.  | Block base depolama.                       |
| Snapshot desteği yok.                  | %99 erişim garantisi                       |
| SSD ve ya HDD.                         | Snapshot desteği                           |
| Sunucunun bulunduğu fiziksel üzerinde. | Replike                                    |
|                                        | Ortak storage üzerinde.                    |

Diskte iops değeri ne kadar yüksekse okuma-yazma hızı yüksektir. Bu değer saniyede ne kadar verinin okunup, yazılacağı değerini belirtir.&#x20;

Throughput bir diskte saniyede kaç mb veri geçişine izin verildiğini belirten değerdir.

Misal, IOPS'u bir arabanın 0-100KMH kaç saniyede çıkabildiğini gösteren bir değer olduğunu düşünürsek, throughput ise o arabanın maksimum ne kadar hıza çıkabildiğini gösterir.

#### AMI - Amazon Machine Image

Daha önceden tanıtılmamış işletim sistemi ve uygulamalarını barındıran sanal makine şablonlarının bulunduğu yer.

Community Ami (Public) : Çeşitli topluluklar ve Amazon tarafından yönetilen AMI

Aws Marketplace (Paid) : Çeşitli kişilerin kendi sunucularında kurduğu yazılım, servislerin, imajını alarak imajını satışa sunduğu kısım.

Private : Kendi sunucularımızı kendimiz yaratıp, kendimiz AMI yarattığımız seçenek.

#### EC2 Sunucu kurulumu esnasında dikkat edilmesi gereken kısımlar,

Number of instance : Yaptığımız ayarlarla kaç adet sunucu kurulacaksa bunu belirtiyoruz.

Purchasing Option :  Spot teklifi vererek, sunucu talep edebiliriz. (opsiyonel)

Auto Assing Public IP : Otomatik olarak sunucuya public bir IP adresi atar.

Capaticy Reservation : Kaynak rezerve edebiliriz.

IAM Role : Burada kural ekleyebiliyoruz. Örneğin EC2 sunucularımızın S3 depolama alanına erişmesini istiyorsak, ilgili kuralı bulup ekleyebiliriz.

Shutdown Behavior : Sanal sunucuya kapama komutu geldiğinde,  sunucu için  AWS'nin ne yapacağını belirtebiliriz. \
\*Stop : Sunucuyu sadece kapat.\
\*Terminate : Sunucu kapandıktan sonra herşeyi sil.

Enable Termination Protection : Bu seçili olduğu zaman api ve management console üzerinden sunucuyu terminate edemeyiz. Bu koruma kalkana kadar sunucu silinmez.

Monitoring : Normalde AWS 5 dakika da bir sunucuyu izler ve biz istersek bu seçeneği aktif edip, bu  süre 1 dakikaya düşer.

Tenancy : Dedike bir host mu, yoksa shared bir host mu kullanacağız bunu seçiyoruz.

#### Advanced Details

Sanal sunucu ilk defa açıldığında çalışmasını istediğimiz komutları buradan yazabiliriz.

Disk,Security group,tag ekleyebiliriz.

Sanal sunucumuza erişmek için key-pair oluşturmalıyız.&#x20;

{% embed url="https://www.youtube.com/watch?ab_channel=Telusko&v=yPdmy--Uh50" %}

Actions kısmından image seçersek, bir ami yaratabiliriz.

Status Check hypervisor'da sorun olursa bunu status check üzerinde "system status checks" de görebiliriz. Sunucu da bir sorun olursa bunu instance status check üzerinde görebiliriz.

#### EC2 altında bulunan diğer özellikler;

Elastic Load Balancing : Yük dağıtım servisi, haproxy gibi düşünebiliriz.\
Aplication Load Balancing : Web sitesi uygulamalarımızın yük dağıtımını yapabiliriz.\
Network Load Balancing : TCP, ağ isteklerinin yük dağıtımını yapabiliriz.

Autoscaling : Sistem kaynaklarını optimize eder, yeni sunucuları devreye alır. Sistemin ihtiyacı olan kaynağı sağlar. Örneğin sanal sunucuların 5 dakika boyunca %90 CPU kullanımı yapıyorsa, git ve ortama sanal sunucu ekle ve bu sunucular diğer sunucular cpu kaynak kullanımı %30'un altına düşerse, yeni eklenen sanal sunucuyu sil gibi kurallar oluşturabiliriz. Ve tüm süreci otomatize edebiliriz.

Placement Group : EC2 üzerinde 2 sanal sunucu yarattık diyelim, bu sunuculardan biri X fiziksel hostu üzerinde oluşturulurken, diğer ise farklı fiziksel host üzerinde oluşturuluyor. AWS isterseniz sunucuları aynı host üzerinde çalışsın diyorsanız, Sunucuları placement group'a dahil edebilirsiniz. Placement gruba alınacak sunucular instance tipleri aynı olmalı yani m4 x1 ve c5 x1 tipleri kullanılmaz. Hepsi örneğin, m4=m4 olmalı. Tüm sunucuları tek seferde placement gruba alabiliriz. Örnek olarak, placement grubunda bir sunucuyu kapatıp açarsak düşük bir ihtimal hata alabiliriz. Hata alırsak tüm placement grubunda bulunan sunucuları kapat-aç yapmalıyız.

Cluster menüsü altında bulunan sunucular aynı cluster altındaki sunuculardır.

Spread  Placement Group : Tüm sunucuları ve ya önemli 2 sunucun var ve fiziksel host üzerinde sorun olduğunda 2 sunucunun da down olmasını istemiyorsan spread grup ile bunu sağlayabiliriz.

EC2 sunuculara dışarıdan erişim kapalıdır. Security group içerisinden bunu değiştirebilir ve izinler yazabiliriz.

Volumes kısmından yeni bir disk eklemek istediğimizde availability zone diski ekleyeceğimiz sunucu ile disk volume'ı aynı az(availability zone) da kullanılmalıdır.

Oluşturduğumuz volume seçip "actions" kısmından "attatch volume" olarak seçebiliriz. Burada instance kısmında bağlayacağımız sunucuyu seçiyoruz. Ve diskimizi sunucuya bağlıyoruz. OS içerisinde diski genişletebiliriz. Ve ya yeni bir volume oluşturabilirsiniz.

Ek olarak diski genişletmek için, "Actions" kısmından modify volume diyerek mevcut volume genişletebiliriz.

AMİ oluşturmak;&#x20;

{% embed url="https://www.youtube.com/watch?ab_channel=AOSNotes&v=eqGwgjcgZ2o" %}

Sunucuyu kapatıyoruz, Snapshot menüsünden snapshot oluşturuyoruz. volume ve sunucu adını seçiyoruz. Create diyerek ilerliyoruz. Snapshot datası s3 üzerinde tutulur. Snapshot 'u seçip actions diyoruz create volume diyerek, bir volume yaratıp başka bir sunucuya disk olarak bağlayabiliriz. Create image diyerek, AMI imajı oluşturabiliriz. Copy diyerek farklı bir region'a kopyalayabiliriz.&#x20;

### EC2 Metada

* EC2 sunucumuz ile ilgili bir çok bilgiyi tutan HTTPs API servisidir. Böylelikle sunucumuzdan API sorgusu atarak, hangi AMI kullandığını öğrenebiliriz vb... sunucu hakkında bir çok konu da bilgi alma şansımız olur.&#x20;
  * metadata v1 : API any açık. Token yok.
  * metadata v2 : Tokenli api çağrısı.
