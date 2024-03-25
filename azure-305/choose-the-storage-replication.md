# 🌟 Choose the storage replication

📂 Azure kullanıcıları, ihtiyaç duydukları erişilebilirlik ve maliyet faktörlerini dikkate alarak aşağıdaki çeşitli depolama kopyalama stratejileri arasından seçim yaparlar. Örneğin, veri kesintisizliği ve felaket kurtarmaya yüksek önem veriliyorsa GRS veya GZRS tercih edilirken, daha düşük maliyetle yerel koruma yeterliyse LRS uygun bir seçenek olabilir. ZRS ise bölgesel hizmet kesintilerine karşı bir çözüm sunar.

1. **Locally-redundant storage (LRS)**: Verilerinizi aynı veri merkezi içinde birden fazla donanım üzerine kopyalar. En düşük maliyetli seçenek olup, temel koruma sağlar ve kritik olmayan senaryolar için önerilir. LRS, veri merkezindeki  "fault domain" içinde üç kez kopyalanır.
2. **Geo-redundant storage (GRS)**: Verilerinizi birincil bölgede kopyaladıktan sonra, bir felaket anında kullanılmak üzere başka bir coğrafi bölgedeki ikincil bir veri merkezine de kopyalar. GRS, yedekleme senaryoları için önerilir.
3. **Zone-redundant storage (ZRS)**: Veriler, aynı coğrafi bölge içinde farklı kullanılabilirlik bölgelerine dağıtılarak kopyalanır. Bu, yüksek erişilebilirlik senaryoları için önerilir.
4. **Geo-zone-redundant storage (GZRS)**: Bu, GRS ve ZRS'nin bir kombinasyonudur. Veriler önce birincil bölgede farklı kullanılabilirlik bölgelerine kopyalanır, sonra da başka bir coğrafi bölgedeki ikincil veri merkezine. Kritik veri senaryoları için uygundur.

Azure, verileri farklı fault domain'ler arasında dağıtarak tek bir hata durumunun birden çok kaynağı etkilemesini önlemeye çalışır. Örneğin, LRS kullanan bir hizmette, verileriniz aynı veri merkezinde, ancak farklı sunuculara veya donanım birimlerine dağıtılır. Bu, bir sunucu arızalandığında dahi veri erişilebilirliğini korumayı amaçlar.

Ancak, bu fiziksel konumda (datacenter) genel bir sorun yaşanırsa (örneğin elektrik kesintisi, doğal afet vb.), tüm sunucular etkilenebilir ve bu da veri kaybına veya kesintiye yol açabilir. Bu nedenle, kritik uygulamalar genellikle ZRS veya GZRS gibi, verileri birden fazla coğrafi konumda veya kullanılabilirlik bölgesinde saklayan stratejileri tercih edilmelidir.



{% embed url="https://learn.microsoft.com/tr-tr/azure/storage/common/storage-redundancy" %}
