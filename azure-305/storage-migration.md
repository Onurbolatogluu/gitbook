# 👥 Storage migration

Azure Storage migration, mevcut veri depolama çözümlerinizden verilerinizi Azure Storage hizmetlerine aktarma işlemidir. Bu süreç, şirketlerin veya bireylerin on-premises veri depolama sistemleri, fiziksel sunucular veya başka bulut sağlayıcılarından Azure bulutuna geçiş yapmalarını kapsar. Azure Storage, çeşitli depolama seçenekleri sunar, bunlar arasında Azure Blob Storage, Azure File Storage, Azure Table Storage ve Azure Queue Storage yer alır. Bu hizmetler, geniş ölçeklenebilirlik, yüksek kullanılabilirlik ve güvenlik sunar.

#### Online Migration

Online taşıma, veri depolama alanınızı Azure'a taşırken kaynak verilerin sürekli kullanılabilir olmasını sağlayan bir yöntemdir. Kullanıcılar ve uygulamalar taşıma sürecinde verilere kesintisiz bir şekilde erişmeye devam edebilirler.&#x20;

* **Windows Server Storage Migration Service**: Bu hizmet, Windows sunucularınızın depolama verilerini Azure'a veya başka bir Windows sunucuya taşımak için kullanılır. Var olan verileri, yapılandırmaları ve kimlik bilgilerini yeni bir sunucuya veya doğrudan Azure'a aktarabilir.
* **Azure File Sync**: Bir bulut tabanlı depolama hizmetidir ve Azure File Share ile on-premises Windows Server'larınızı senkronize etmenizi sağlar. Bu şekilde, veriler buluta taşınırken yerel sunucunuzdaki dosyalara erişim devam eder.
* **AzCopy**: Komut satırı tabanlı bir araç olan AzCopy, verileri Azure Storage'a hızlı bir şekilde yüklemek veya indirmek için kullanılır.
* **Storage Explorer**: Grafiksel bir kullanıcı arayüzü sağlar ve Azure Storage kaynaklarını yönetmenize, dosya yüklemelerinizi ve indirmelerinizi kolayca yapmanıza olanak tanır.

#### Offline Migration

Offline taşıma, verilerinizi Azure'a taşımanız gerektiğinde, bu süreç boyunca verilere erişimin geçici olarak durdurulmasını kabul ettiğiniz bir yöntemdir. Taşıma tamamlandıktan sonra verilere Azure üzerinden erişilebilir.&#x20;

* **Azure Import/Export Service**: Bu hizmet, fiziksel diskleri kullanarak büyük miktarda veriyi Azure Blob Storage ve Azure Files'a taşımanıza olanak tanır. Disklerinizdeki verileri Azure data center’larına gönderirsiniz, burada veriler Azure Storage hesabınıza aktarılır.
* **Azure Data Box**: Büyük veri setlerini Azure'a güvenli ve hızlı bir şekilde taşımak için kullanılan sağlam, taşınabilir bir depolama cihazıdır. Özellikle ağ üzerinden taşıma çok zaman alacak kadar büyük veri setleri için uygundur.

