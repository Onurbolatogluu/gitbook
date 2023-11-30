---
description: Elastic File Storage (NFS Protokolü)
---

# 🗄 EFS

NFS bazlı dosya depolama servisi. EFS de diskler oluşturulur sunucuya bağlanır. EFS de bir dosya sistemi yaratıp, bunu bir sunucuya, bir klasöre bağlayabiliyoruz. İçerisine dosyaları upload edebiliyoruz. EBS de disk tek bir sunucuya bağlıdır. Tek diski birden fazla sunucuya mount edemeyiz. EFS bir disk değildir. Bir sunucuya bağlayıp üzerine işletim sistemi kuramayız. EFS NFS tabanlı bir dosya deposudur. Dosya ekledikçe otomatik olarak büyür. Dosya silindikçe otomatik olarak küçülür. Aynı anda birden fazla sunucu erişebilir.

#### EFS için Security Group işlemleri;

* EFS 'in sunuculara erişimi için "Security Group" kısmından kural yazmalıyız.
* Create Security group kısmından, bir isim belirliyoruz.
* SG içerisinde inbound rule kısmına gelip, type kısmını NFS Protocol kısmını TCP seçiyoruz.
* TCP Port Range kısmını 2049 yazıyoruz. Source kısmı Custom olarak seçip,
* EFS'e bağlanacak olan sunucularımızın bulunduğu SG seçiyoruz. ve SG grubunu oluşturuyoruz.

#### EFS Oluştururken;

* EFS servisine girip, Create file system diyoruz.
* VPC ve AZ seçiyoruz. ve yukarıda oluşturduğumuz SG seçiyoruz.
* Default gelen SG kaldırabiliriz. Next.
* Key kısmına bir isim giriyoruz. Value kısmına EFS olarak belirtiyoruz. Farklı bir şey de yazabiliriz.
* Diğer seçenekler default kalabilir.&#x20;
* Next. Ve EFS oluşturulur.
* Sunuculara mount edebiliriz.

{% embed url="https://www.youtube.com/watch?ab_channel=AmazonWebServices&v=Aux37Nwe5nc" %}

### UPDATE

* S3 gibi tier sistemi eklendi (Storage tier sistemi)
* EFS yapılandırılırken, dosyalarımızı yedeklemek istersek, enable automatic backups seçiyoruz.
* Dosyalarımızı artık tek bir zone üzerinde saklayabiliyoruz.
* EFS oluştururken File system policy ile dosyalarımıza erişim izinlerini düzenleyebiliriz.
* Farklı IAM kullanıcılarına grant additional permissions ile yetki düzenleme yapabiliriz.
* S3 gibi farklı farklı policyler ile farklı access pointler oluşturabiliriz.
