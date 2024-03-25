# 🌟 Choose the storage replication

📂 Azure kullanıcıları, ihtiyaç duydukları erişilebilirlik ve maliyet faktörlerini dikkate alarak aşağıdaki çeşitli depolama kopyalama stratejileri arasından seçim yaparlar. Örneğin, veri kesintisizliği ve felaket kurtarmaya yüksek önem veriliyorsa GRS veya GZRS tercih edilirken, daha düşük maliyetle yerel koruma yeterliyse LRS uygun bir seçenek olabilir. ZRS ise bölgesel hizmet kesintilerine karşı bir çözüm sunar.

1. **Locally-redundant storage (LRS)**: Verilerinizi aynı veri merkezi içinde birden fazla donanım üzerine kopyalar. En düşük maliyetli seçenek olup, temel koruma sağlar ve kritik olmayan senaryolar için önerilir. LRS, veri merkezindeki  "fault domain" içinde üç kez kopyalanır.
2. **Geo-redundant storage (GRS)**: Verilerinizi birincil bölgede kopyaladıktan sonra, bir felaket anında kullanılmak üzere başka bir coğrafi bölgedeki ikincil bir veri merkezine de kopyalar. GRS, yedekleme senaryoları için önerilir.
3. **Zone-redundant storage (ZRS)**: Veriler, aynı coğrafi bölge içinde farklı kullanılabilirlik bölgelerine dağıtılarak kopyalanır. Bu, yüksek erişilebilirlik senaryoları için önerilir.
4. **Geo-zone-redundant storage (GZRS)**: Bu, GRS ve ZRS'nin bir kombinasyonudur. Veriler önce birincil bölgede farklı kullanılabilirlik bölgelerine kopyalanır, sonra da başka bir coğrafi bölgedeki ikincil veri merkezine. Kritik veri senaryoları için uygundur.

"Fault domain" ise bir hata durumunda etkilenen bileşenlerin kümesidir. Örneğin, bir veri merkezindeki bir sunucu rafı, bir fault domain olarak düşünülebilir. Eğer bu raf da bulunan cihazlar arızalanırsa, LRS stratejisini kullanıyorsanız verilerinize erişemezsiniz. Çünkü tüm kopyalar aynı fault domain içindedir. ZRS ve GZRS gibi stratejiler ise farklı fault domainlerde veri kopyalarını saklayarak tek bir fault domaindeki hata durumundan korunmanızı sağlar.



{% embed url="https://learn.microsoft.com/tr-tr/azure/storage/common/storage-redundancy" %}
