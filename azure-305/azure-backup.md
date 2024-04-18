# 👜 Azure Backup

Azure Backup, hem Azure'da (bulutta) hem de Azure dışında (on-premises) bulunan iş yükleriniz için yedekleme çözümleri sunan bir hizmettir.

<figure><img src="../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

#### Azure Backup'ın Ana Özellikleri:

**Backup Agents:** Azure dışındaki fiziksel veya sanal ortamlarda çalışan iş yüklerini yedeklemek için kullanılır. Örneğin, bir şirketin kendi sunucu odasında bulunan veya başka bir bulut sağlayıcısında barındırılan sistemlerdeki verileri korumak için Microsoft Azure Recovery Services (MARS) veya Microsoft Azure Backup Server (MABS) gibi ajanlar devreye girer. Bu ajanlar, verileri düzenli aralıklarla Azure Backup servisine aktararak güvenli ve merkezi bir şekilde saklanmasını sağlar.

**Built-in Backup:** Azure, kendi hizmetleri için özel olarak entegre edilmiş yedekleme çözümleri sunar. Bu, Azure üzerinde çalışan sanal makineler (VM'ler), dosya paylaşımları, SQL Server veritabanları ve SAP gibi uygulamalar için geçerlidir. Bu entegre çözüm sayesinde kullanıcılar, ek bir yedekleme yazılımı kurmadan doğrudan Azure içindeki kaynaklarını yedekleyebilirler.

**Restore Point Manager:** Bu özellik, oluşturulan yedeklemelerin zaman bazında yönetilmesini sağlar. Kullanıcılar, herhangi bir sorun anında veya veri kaybı durumunda, ihtiyaç duyulan spesifik bir tarihe ait veriyi kolaylıkla geri yükleyebilir.

#### Vaults,

Azure'da "vault" terimi, verilerin yedeklenmesi ve gerektiğinde geri yüklenmesi için kullanılan, güvenli ve yönetilen bir depolama hizmeti anlamına gelir. Vault'lar, verilerin korunmasını ve kurtarılmasını sağlayan bir çeşit dijital kasadır. Azure'da iki ana tür vault bulunmaktadır:

* **Recovery Services Vault**
* **Backup Vault**

<figure><img src="../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

#### Mabs vs Mars,

Microsoft Azure Backup Server (MABS) ve Microsoft Azure Recovery Services (MARS), Microsoft'un sağladığı yedekleme ajanlarıdır. MABS, yerel sunucuları ve Microsoft uygulamalarını yedeklemek için kullanılırken, MARS ise Windows işletim sistemlerinde dosya ve sistem durumu yedeklemelerini gerçekleştirir. MABS, yerel ve bulut tabanlı yedekleme seçenekleri sunarken, MARS sadece Azure bulutuna yedekleme yapar. Her iki çözüm de Microsoft'un Azure Backup hizmetiyle entegre çalışır.

#### Temel Farklar,

* MABS, MARS’tan daha geniş bir yelpazede teknoloji ve uygulama desteği sunar. MABS, MARS'ın sağladığı yalnızca dosya ve sistem durumu yedeklemelerinin ötesine geçerek, sanal makineler ve kurumsal uygulamalar için de yedekleme imkanı sağlar.
* MARS sadece Azure'a yedekleme yaparken, MABS yerel ve bulut depolama seçeneklerini destekler.
* MABS, daha karmaşık kurulum ve yönetim gereksinimleri ile gelir, ancak bu sayede daha esnek ve güçlü bir yedekleme çözümü sunar. MARS ise daha basit, doğrudan ve daha az yönetim gerektiren bir çözümdür.



{% embed url="https://learn.microsoft.com/en-us/azure/backup/" %}
