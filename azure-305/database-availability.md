# 🕯️ Database availability

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Standard Model – General Purpose / Standard / Basic

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Topoloji, Azure region içinde bir uygulamanın mimari yapısını göstermektedir. Uygulamadan gelen veritabanı istekleri, Control ring üzerinden yük dengeleyicilere (GW) yönlendirilir. İstekler, birincil replika üzerinden işlenir. Eğer birincil replika kullanılamaz hale gelirse yapılandırılmış failover mekanizmaları devreye girer ve yedek nodelardan biri birincil replika olarak devreye alınır.&#x20;



### Premium Model – Business Critical / Premium

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Uygulamadan gelen istekler, control rlng 'deki yük dengeleyicilere (GW) yönlendirilir ve ardından birincil replikaya ulaşır. Always On Availability grubu, birincil replikanın yanı sıra birden fazla ikincil replikaya sahiptir ve her birinin SSD'de veri ve log dosyaları bulunur. Eğer birincil replikada bir sorun olursa, otomatik failover mekanizması devreye girer ve bir ikincil replika birincil olarak devreye alınır.&#x20;



### Hyperscale Model – Hyperscale tier

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Bu modelde, Compute layer'da bulunan primary compute node, okuma-yazma işlemlerini yaparken, read-only işlemler için ek secondary compute node'lar da bulunur. Compute node'larda bir sorun olması durumunda bile bu layer, veritabanı verilerinin bütünlüğünü korumayı garanti eder.&#x20;



{% embed url="https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla?view=azuresql&tabs=azure-powershell" %}

