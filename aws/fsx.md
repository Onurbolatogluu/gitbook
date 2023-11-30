# 🗃 FSx

* Yönetilen Windows File server

![](../.gitbook/assets/Product-Page-Diagram\_Managed-File-System-How-it-Works\_Updated@2x.c0c4e846c0fca27e8f43bd1651883b21b4cc1eec.png)

<mark style="color:red;">Linux ortamlarda (EFS), Windows ortamlarda (FSx) kullanabiliriz.</mark>&#x20;

FSx For Lustre : High performance computing. Yüksek performans ihtiyaç olan yerlerde dosya saklama deposu olarak kullanabiliyoruz.

#### Uygulama,

* File System Name : satis
* Deployment Type :&#x20;
  * Multi AZ : 2 ayrı node kurulumu yapılır. Bir AZ 'de sorun olduğunda, diğerinden bağlantı devam eder.
  * Single AZ : Tek node üzerinden çalışır.
* Storage Type : Disk tipi seçimi yapabiliriz.
* Storage Capacity : 32GB - 65000GB arasında seçim yapabiliriz.
* Throughput : Saniye de okunacak veriyi yazabilir (seçebiliriz) veya önerilen ayarları kullanabiliriz.
* VPC : File Server 'ın bağlı olacağı network seçimini yapabiliriz.
* Sec-Group oluşturabilir, mevcut olan gruplardan kullanabiliriz.
* Win Auth : Bu seçeneği kullanmak için AD yapısı şart.
  * AWS Managed Microsoft AD : AWS bizim yerimize AD oluşturur, kendi yönetir. İlk önce bu servise gidip AD oluşturmalıyız.
  * Self Managed Microsoft AD : Kendi AD sunucum var, Kendi lokalimde veya EC2 sunucularımdan birinde olabilir. Bu AD sunucumu kullanmak istiyorum.&#x20;
* Ecryption
  * Encryption key : Verinin şifrelenmesine yardımcı olur. Böylelikle verimizin saklandığı yerde şifrelenmiş bir şekilde saklanır. Bunu KMS servisi sağlıyor.

{% hint style="info" %}
Client cihazlar, FSx servisine SMB protokolü kullanarak erişir.
{% endhint %}

* Maintenance Preferences :  Yedeklemeleri ayarlayabiliriz.
  * Daily automatic backup window : Günlük olarak.
  * No Preference : AWS 'in belirlediği zamanlarda alınacak.
* Weekly Maintenance : File server bakımı için zaman ayarı girebiliriz.&#x20;
  * Select Start Time For : Bizim belirlediğimiz zaman diliminde olsun.
* Next dediğimizde bize bağlanabileceğimiz URL verecek (10-15dk sürebilir) Win+R tuşları ile (çalıştır) veya File explorer ile bu file share(paylaşıma) servisine bağlanabiliriz.
