---
description: AWS Database
---

# 🦈 RDS

Veritabanı, Temelde içerisinde bir ve ya binlerce yapısal tablolar yapıp, içerisine veri doldurduğumuz uygulamalardır. Belirli bir formatta veriyi saklamaya ve kendi özel dilinde yapılan sorgularla bu saklanan veriden cevaplar çıkarmaya ve aynı zamanda bu veriye yeni eklemeler yapmaya yarayan uygulamaların ortak adıdır.

Veritabanları 2'ye ayrılır.&#x20;

1- Relational database (ilişkisel)

* Tablo tabanlıdır.
* Hiyerarşik ve ilişkisel veriler için uygun.
* Ön tanımlı şema gerektirir.
* Dikey genişler.
* SQL dilini destekler.
* Karmaşık sorguları destekler.
* Join işlemine imkan verir.
* Yüksek işlem kapasitesi sağlar.
* En eski ve en çok kullanılan database tipidir. Verileri kolonlar ve satırlar halinde tablolar içerisinde barındırıyor.
* Bu SQL tipine ilişkisel denilmesinin sebebi tabloların kendi aralarında ilişki-bağlantı kurup,  ortak sonuçlar çıkarabilmesidir.
* İki ve ya daha fazla tablodaki satırları aralarında ilgili bir sütuna göre birleştirmek işlemine join denilmektedir.

2- _Non_-_relational database_

* Döküman tabanlı.
* Dağıtık ve büyük veri setlerine uygun.
* Şema dinamiktir.
* Yatay genişler.
* SQL dilini desteklemez.
* Daha temel ve basit sorguları destekler.
* Join işlemine imkan sağlamaz.
* Tek dize işlem kapasitesi sağlar.
* Veriler tablolarda değil, dökümanlarda anahtar-veri eşlikleri halinde tutuluyor.
* Tablo yok, koleksiyon denilen içerisinde json formatında veri tutan dökümanlar mevcut.
* Dinamik olarak, kodun içerisinde istediğin veriyi, istediğin veri tipini bir ortalama yazabiliyoruz.
* Join işlemi yok, karmaşık sorgu yok.

AWS hem SQL hem de NoSQL yönetilebilen veri tabanı hizmeti sunmaktadır. Yönetilen veri tabanı hizmeti olarak RDS :&#x20;

* MSSQL
* Oracle
* MYSQL
* MariaDB
* PostgreSQL
* Amazon Aurora

Gibi servisleri kullanma imkanı sunmaktadır.

NoSQL tarafında ise, DynamoDB hizmetini AWS bize sunuyor.&#x20;

RDS AWS tarafında yönetilen DB servisi hizmetidir. Yönetilen kısmı, veritabanının kurulumu, kaynak yönetimi, bakım işlemlerinin AWS tarafından yapılıyor olmasıdır. Veritabanı dizayn ve veri manipülasyon işlemleri bize aittir. Ek olarak Aurora mysql ve postgresql veritabanı moturunu destekliyor.

RDS Kullanmak için,

* Hangi DB uygulamasını kullanacaksak bunu seçiyoruz.
* Ardından DB servisimizin kaynaklarını seçiyoruz.
* Disk tipini seçiyoruz (disk i/o önemli)

RDS Özellikleri,

* Instance start-stop desteği var. 7 gün sonra instance terminate olur.
* Tek butona tıklayıp, instance boyutunu büyütüp ve ya küçültebiliriz.
* Kesintisiz depolama imkanı.
* Otomatik bakım ve güncelleme.
* Otomatik yedekleme, manuel yedekleme, snapshot ve snapshot'dan yeni database oluşturmak.
* Günün istenilen anına yedekten dönme imkanı.

Multi AZ,

* Veri tabanının aynı region içerisindeki senkron bir kopyası.
* Aktif-Pasif cluster.
* Otomatik failover imkanına sahip.
* Kesintisiz çalışma ve felaketten kurtarma senaryoları için.
* Performans artışı sağlamıyor.

Read Replica,

* Veritabanının aynı ve ya farklı region'da oluşturabileceğimiz bir ve ya birden fazla asenkron kopyası.
* Bu bir cluster değil.
* Read only bir kopya.
* Performans artışı sağlamak için.

Eğer amacımız performans ise, Read replica mantıklı seçenektir. Amacımız kesintisiz - sağlam altyapı ise, multi AZ mantıklı bir seçenektir.

#### RDS Servisi Oluşturmak,

* RDS > Create Database
* Hangi DB kullanacaksak, bunu seçiyoruz. Misal Mysql.
* Next
* DB Engine versiyonunu seçiyoruz.
* DB instance class seçiyoruz. Hangi tip kaynaklarda DB istediğimizi soruyor.
* Allocate Storage. Disk Alanı seçiyoruz.
* Storage Type, Disk tipini seçiyoruz.

#### Settings,

* DB instance idenfier : Oluşturacağımız DB'nin isimini giriyoruz.
* Storage Type, Disk tipini seçiyoruz.
* Master username, Database yetkili kullanıcısının oluşturulmasını istiyor. Buraya DB kullanıcı ismini giriyoruz. Misal : admin
* Master password, Kullanıcımızın şifresini oluşturuyoruz.
* Next

#### VPC Ayarları,

* Public Accesibilty : Yes, Tüm dünyadan erişme açık. Böylelikle public bir IP alacak. No, seçersek eğer, sadece bulunduğu VPCden erişlebilir.
* VPC Sec Group, ister mevcut, istersek yeni bir sec grup yaratabiliriz.

#### Database Options,

* Dbname, database ismini giriyoruz.
* Port, Dilersek farklı bir port yazabilir, dilersek boş bırakıp default kullanabiliriz.
* DB Parameter group : Default
* IAM DB Auth : Disable, Normal mysql kullanıcısı ve şifre ile dbye bağlanabiliriz. Enable ettiğimiz de, IAM de yaratılan kullanıcılar bu db üzerinde yetkilendirebiliriz.

#### Backup,

* Kaç gün geriye dönük backup alacaksak, bunu belirtiyoruz.
* Backup Window, yedek ne zamanlar alınmalı? Select windows diyerek bir zaman seçebiliriz.
* Duration , Kaç saate kadar işini bitirmeli?

#### Monitoring,

* İzlemeyi açıp, kapatabiliriz.

#### Log Exports,

* Hangi log tipini cloudwatch servisine göndereceğimizi soruyor.&#x20;

#### Maintenance,

* Otomatik minör updateler yapılsın mı?
* Maintenance Window, Ne zaman minör updateler kontrol edilsin?
* Deletion Protection, Enable dersek, RDS'i manuel olarak silemeyiz.
* Create

DB oluşturulması 5 dakikayı bulabilir. Yaratılan instance SSH ile bağlanamayız. Bu instance AWS'ye aittir Sadece SQL client ile bağlantı kurabiliriz. DB Sec Group üzerinden bu DB için gerekli izinleri inbound rule'da görebiliriz. Databases sekmesinde "endpoint" kısmında yazan dns name ile DBye bağlanabiliriz. Bağlanırken SQL client ugulamarından yararlanabiliriz.

#### RDS Yapılandırma Sekmeleri,

Performance insights : DB performasına yönelik istatisliklerini görebiliriz.

Snapshots : Alınan yedekleri görebileceğimiz kısım. Otomatik backup yanı sıra manuel olarak da yedekleme işlemini başlatabiliriz.

Automated Backups : Backupların nasıl hangi zamanlarda alındığını görüyoruz.

Reserved Instance : Rezerve edilmiş kaynaklar alabiliyoruz.&#x20;

Subnet Groups : DB'mizin kayıtlı olduğu subnet grubu.

Parameter groups : DB mizin, ekleyebileceğimiz modülleri, ayarlamalarını seçebiliyoruz.

Option Group : DB için seçebileceğimiz ek özellikler.

Events : DBnin geçmişe ait loglarını görebiliriz.

Events Subs : Önemli olayları mail göndermesini istiyorsak, buradan ayarlayabiliriz.

Recommendations : AWS nin DBmiz için bize sunduğu öneriler bu kısımda bulunuyor.



#### Actions Kısmı,

* Create Read Replica : Read replika yaratabiliriz.
* Create Aurora Read Replika : Aurora DB kullanıyorsak, read replika oluşturabiliriz.
* Migrate snapshots : Mevcut DBmizi seçtiğimiz farklı bir DB engine çevirmemize yarıyor.

Restore işlemi ile yeni bir DB instance yaratılarak olmaktadır. Mevcut DBnin üzerine yedek yazılmaz.&#x20;

Latest restoreble tpye : son alınan yedeğe geri dönmek.

Custom  : İstediğimiz bir zaman dilimine dönebiliriz.

RDS veritabanlarına IP adreslerinden erişilemez. DNSname kullanarak erişilir. Bu dns ismine databases altında instance'nın üzerine tıklayıp "endpoint" kısmından görebiliyoruz.



Modify > Multi AZ deployment > Yes > Contiune > Apply immendatly > modifydb instance : Böylelikle veritabanımızı multi az olarak ayarladık.&#x20;

Multi AZ : DB'mizin bir kopyasını başka bir AZye alıyor. Ana sunucuda sorun olduğunda, diğeri devreye girecek. Aktif-pasif çalışacaktır. Böylelikle ana sunucuda yapılan her değişiklik pasif sunucuya da yansır.

#### Read Replica oluşturmak,

* DB seç > Actions > Create read replika
* Destination Region > DB mizin, nerede hangi region'da aynısını oluşturmak istiyoruz?
* DB instance identifiler : veritabanımızı seçiyoruz. misal, calisanlar\_db
* Diğer ayarlar default kalabilir.
* Create read replika

Read replika sunucumuza erişmek için sec-group ayarı yapılmalı. Read replika sunuculardan okuma yapılır. Veri girilmez. DBmizi sildiğimiz zaman otomatik backupların tamamı silinir. Manuel olarak bizim aldıklarımız (snaphot) silinmez. Dilersek ileride manuel aldığımız snapshotları kullanarak yeni bir DB yaratabiliriz.

### Redshift - Veri Ambarı

Veri ambarında bulunan tüm verileri analiz etmemize yarar. Mevcut veri analizi yapabildiği gibi, S3de de bulunan verileri de analiz etmemize yarar.

#### Online transaction processing,

* Veri tabanı.
* Veri giriş ve veri manipülasyonu.
* Operasyonel günlük veri işlemleri için.
* Veri analiz sorgularını çalıştırmak, veri tabanının performansını düşürüyor ve mevcut iş akışının yavaşlamasına neden oluyor. Mevcut verileri saklamak için bu seçeneği kullanabiliriz.

#### Online analytical processing,

* Veri ambarı (redshift)
* Veri analizi.
* Konsolide edilmiş birden fazla kaynaktan gelen veri ile çalışabilir.
* Oluşan verileri analiz etmek için kullanabiliriz.

Analiz isteğini SQL sorgusu halinde master node'a gönderir. Master node compute node'lara gönderir, gelen cevabı iş zekası yazılımı ile görsel bir grafiğe dönüştürür ve kullanıcıya sunar.

{% hint style="info" %}
Maneger node ücretsizdir.
{% endhint %}

### DynamoDB

* İlişkisel olmayan veri tabanı (noSQL)
* Key-value şeklinde kodlanan verileri json formatında dökümanlar halinde saklayan noSQL veri tabanı türüdür.
* Temelde noSQL veri tabanları tek anahtar üzerinden index'lemeye izin verirken, DynamoDB birden fazla anahtarın birleşmesinden 2.index yaratılmasına izin verir.
* <10ms gecikme süresi sağlayan, yönetiken no-SQL veri tabanı hizmetidir. Alt yapı AWS'ye aittir.
* Veriler key-value eşlenikler şeklinde json dosyalarında saklanır.
* Multi key indexleme imkanı sağlar.
* SSD tabanlı 3 farklı veri merkezinde 3 farklı kopya halinde tutularak, yüksek erişilebilirlik ve tutarlılık sağlar.
* Autoscaling imkanı bulunur.
* Read-write capacity üstünden scaling sağlar.
* İhtiyaca göre read-write kapasitemizi genişletebiliriz.
* Global tables özelliği ile multi region hem okuma, hem yazma imkanı sağlar.
* Streams özelliği devreye alınır. Streams ile yapılan her işlemin kaydı tutulur. Streamsler başka olayları da tetiklememizi sağlayabilir.
* Stream servisi aktif hale getirirsek, multi region özelliğini kullanabiliriz.
* Streams, öğe bazında yapılan her değişikliğin kaydının tutulmasını sağlar.

#### Global Tables,

![](../.gitbook/assets/DynamoDB\_Global-Tables-01.dad2508b80e8b7c544fe1a94a2abd3f770b789da.png)

Kullanıcılar dünyanın farklı bölgelerinde replike çalışan dynamoDB nosql veritabanından okuma yazma yapabilir.

DynamoDB accelerator,

DynamoDB accelerator ile in-memory caching imkanı sağlanır. 35 güne kadaar otomatik yedeklemenin yanı sıra manuel yedekleme imkanı da vardır. Öğelere TTL değerleri atanabilir.

#### DynamoDB Oluşturmak,

* Services > DynamoDB > Create table
* Table Name : calisanlar\_db
* Primary Key : personelno - number (text değer olarak ekleseydik "string" seçmeliyiz.
* Add Sort Key : Ad (string) İkincil key ekleyebiliyoruz.
* Use Default Settings : Ayarları otomatik yapmak için.
* Secondary indexes : İkincil bir index girebiliriz.
* Read-Write Capacity mode : On-demand, Her hangi bir kapasite belirtmiyoruz. O andaki yüke göre otomatik olarak AWS ayarlıyor.
* Provisioned Capacity : İstediğimiz, ihtiyacımız olan, Read-Write değerlerini girebiliyoruz.
* AutoScaling : İhtiyaca göre read-write kapasitesini DynamoDB burada belirlediğimiz seçeneklere göre bize sağlıyor.
* Target utilization : Kapasitesinin %70ini doldurursa, otomatik olarak aşağıdaki kapasiteler eklenecek.
* Minimum Provisioned : En az vereceği kapasite.
* Maximum Provisioned : En fazla vereceği kapasite
* Encryption At rest : Default
* Create

DynamoDB oluşturuldu, Streams devreye almak için,

* Overview > Manage Stream > New and old images > Enable. Böylelikle ileride global table yaratabiliriz.



DynamoDB RDS de olduğu gibi, Bağlantı için DNS ve IP adresi yoktur. Dynamo ile bağlantı kurmak için tek seçenek, AWS SDK kullanarak, kendi yarattığımız uygulamaları direkt dynamoDB ile bağlantı kurdurabiliriz. Bunu DynamoDB Api kullanarak yapabiliriz.

Point in time Recovery : Günün istediğimiz anına bu tabloyu geri döndürebiliriz.

TTL attributes : Manage TTL > TTL Attribute > TTL yazarsak, artık tablonun içinde bir ttl değeri atayabiliyorum. Örnek olarak tabloya ttl kalonu ekleyip, bir tarih yazabiliriz. O tarihten sonra da TTL eklenmiş olduğu o kolondaki kayıt silinecektir. TTL yazmamızın sebebi, hangi kolonda bu değeri tutacağımızı söyledik.



Global Tables Oluşturmak için, Add region > Frankfurt > Tablomuzun bir kopyasını da frankfurt region'da mevcut.&#x20;

Stream : Yaptığımız tüm değişikliklerin saklanmasını sağlıyor.

İtems > Create items > yeni bir kayıt(döküman) yaratabiliriz.

* No SQL de daha sonradan bir tablo (kayıt) yaratıp farklı kolonlar ekleyebiliriz. Örneğin her dökümanda dogumtarihi ekleme zorunluluğumuz yok.
* Alarms : Dynamodb için alarmları bilgilendirme maili ve ya SMS gönderebilmemizi sağlar.
* Dynamo 'ya çoğu çalışan manuel kayıt girmez. Bu kayıtarı uygulamarımız dynamodb'ye otomatik olarak yazar.
* Backups : Manuel backup alabilir, ve ya otomatik backup 'lar başlatabiliriz. Günün her saniyesine yeni bir restore başlatabiliriz.
* Trigger : Tablona veri ekleyip, lambda fonksiyonunu çalışmasını(tetiklemesini) söyleyebiliriz.
* Access Control : Misal, kullanıcılar yarattığımız uygulamaya facebook bilgileri ile girebilmelerini ve DynamoDB de okuyup, yazabilmeleri için buradan izinler oluşturabilir. Sosyal ağlara izinler verebiliriz (Twitter-Instagram-Facebook-Reddit vb)

#### DynamoDB Accelerator(DAX),

![](../.gitbook/assets/dax\_high\_level.e4af7cc27485497eff5699cdf22a9502496cba38.png)

* <10ms civarında uygulamalarımız veri okuyabilir ve yazabilir.
* Caching Cluster, istekler (uygulama istekleri) Dax'a geliyor. Dax bu istekleri dynamodb'ye yönlendiriyor.
* Dax cluster sayesinde in-memory-cache özelliğini kullanarak uygulamalarımız, DynamoDB tablosundan okuma yapabiliyoruz.
* Böylelikle cluster'ı ayarlayıp devreye alırsak, dynamodb yerine dax'a bağlanıp, uygulamalarımız dax 'a istekleri iletecek ve dax arkada dynamodb' ile iletişim kuracak.

