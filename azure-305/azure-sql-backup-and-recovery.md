# 🧺 Azure SQL backup and recovery

Yedeklemeler, Azure tarafından SQL Database veya Azure SQL Managed Instances gibi PaaS veritabanı ürünleri için dahili olarak yönetilir. Aynı zamanda yönetim karmaşıklıklarını ve DBA'ların veritabanı yedeklemelerini yönetmek için harcadıkları çabayı azaltır. Azure SQL Database, belirli bir noktada kurtarmaya ulaşmak için SQL Server gibi 3 tür yedeklemeyi destekler:

* Full Backup
* Differential Backup
* Transaction Log Backup

Full Backup, veritabanınızda bulunan her şeyin ve Transaction Log dosyasının tam bir dökümüdür. Differential Backup, yalnızca son Full Backup'dan bu yana yapılan değişiklikleri depolar. Transaction Log Backup, Transaction Log dosyasındaki günlük kayıtlarını yakalar. Transaction Log Backup, günlük bilgilerini artımlı biçimde kaydeder.



Veritabanları için otomatik yedeklemeler, her hafta bir tam yedekleme, 12 ila 24 saat aralığında bir Differential Backup ve her 5 ila 10 dakikada bir Transaction Log Backup çalıştırır. Azure SQL veritabanı oluşturduğunuz anda ilk tam yedekleme Azure tarafından alınacaktır. Bunun için herhangi bir şey yapılandırmanıza gerek yoktur. Herhangi bir SQL veritabanını oluşturduğunuzda Azure bunu sizin için yapar. Kalan Differential ve Transaction Log yedeklemesi, bu tam yedeklemeden sonra arka planda çalışmaya başlayacaktır. Sistem, veritabanınızdaki iş yüküne göre hangi yedeklemenin ne zaman çalıştırılacağına ve programlarına karar verecektir. Bu yedeklemeler, belirli bir noktada kurtarma için veya veritabanlarını başka bir Azure konumuna veya bölgesine geri yüklemek için kullanılabilir.



Azure SQL veritabanları iki tür yedekleme saklama **Policy**'ını destekler:

* **Short-Term** **Retention** **Policy**
* **Long-Term** **Retention** **Policy**

**Short-Term Retention Policy** belirli bir zamana geri yüklemeler için kullanılırken, **Long-Term Retention Policy** çeşitli denetim ve uyumluluk amaçları için uzun vadeli veya eski yedeklemelerden geri yüklemeleri ele almak için kullanılır. Bu yedekleme dosyalarını ayrıca 7-35 güne kadar **Short-Term Retention Policy**'nin bir parçası olarak da kaydedebiliriz. Varsayılan yedekleme saklama süresi 7 gündür, ancak hizmet katmanınıza bağlı olarak değişir. DTU satın alma modeli kapsamındaki temel hizmet katmanının maksimum saklama süresi 7 gündür, standart ve premium hizmet katmanlarının maksimum saklama süresi 35 gündür. Bu saklama süresini Azure portalını kullanarak ihtiyacınıza göre ayarlayabilirsiniz.

**Long-Term Retention Policy**, yedekleme dosyalarını daha uzun süre saklamak istiyorsanız kullanabilirsiniz. Maksimum 10 yıl **Long-Term Retention** süresi belirleyebilirsiniz. 10 yıllık bir **Long-Term Retention** süresi seçtiyseniz, 10 yıllık verilerinizi geri yükleme yeteneğine sahip olacağınız anlamına gelir. **Long-Term Retention**, yani LTR, haftalık bir yedekleme gerçekleştirir ve yedekleme kopyalarını 10 yıla kadar Azure BLOB deposunda saklar.&#x20;

Örneğin,&#x20;

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* **Weekly LTR Backups**: Bu bölümdeki "90 Days" seçeneği, haftalık yedeklerin 90 gün boyunca saklanacağını belirtir. Yani, sistem her hafta bir yedek alacak ve bu yedekleri en fazla 90 gün saklayacak. 90 günün sonunda, her yeni yedekle birlikte en eski haftalık yedek silinecektir.
* **Monthly LTR Backups**: "52 Weeks" seçeneği, her ayın ilk yedeğinin 52 hafta (yaklaşık 1 yıl) boyunca saklanacağını belirtir. Böylece, her ayın ilk haftasında alınan yedekler bir yıl (52 hafta) boyunca muhafaza edilir.
* **Yearly LTR Backups**: "5 Years" seçeneği, yıllık yedeklerin 5 yıl boyunca saklanacağını ifade eder. "Which weekly backup of the year would you like to keep?" sorusu ise hangi haftanın yedeğinin yıllık olarak saklanacağını seçmenizi sağlar. Bu durumda, "Week 1" seçildiği için, yılın ilk haftasına ait yedek 5 yıl süresince saklanacak.&#x20;



{% embed url="https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview?view=azuresql" %}

{% embed url="https://learn.microsoft.com/en-us/azure/azure-sql/database/long-term-backup-retention-configure?view=azuresql&tabs=portal" %}



