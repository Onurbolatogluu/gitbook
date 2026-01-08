# 🎞️ Design for Azure storage accounts

### Choose the storage account type:

Azure Storage hesapları için farklı depolama seçenekleri ve performans katmanları mevcuttur.

<figure><img src="../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

1. **Standard GPv2**: Genel amaçlı v2 (GPv2) hesapları, blobs (Block Blobs ve Page Blobs), files, queues ve tables da dahil olmak üzere Azure Storage hizmetlerinin çoğunu destekler. Bu hesap türü, çoğu senaryo için önerilir ve lokal olarak yedeklenmiş depolama (LRS) sunar.
2. **Premium Page Blobs**: Sadece Page Blobs'ı destekleyen ve yüksek performanslı depolama senaryoları için ideal olan premium katmanda bir depolama hesabıdır. Özellikle düşük gecikme süresi ve yüksek işlem hızı gerektiren uygulamalar için önerilir.
3. **Premium Block Blobs**: Block Blobs ve veri göllerini (data lakes) destekleyen premium depolama hesabıdır. Yüksek işlem oranı ve düşük gecikme süresi gerektiren senaryolar için idealdir.
4. **Premium File Shares**: Azure File Shares'ı destekleyen premium depolama hesabıdır. Bu, özellikle büyük ölçekli SMB ve NFS dosya paylaşımları için gerekli olan kurumsal düzeyde performans gerektiren senaryolar için tasarlanmıştır.

Her bir yapılandırmanın lokal olarak yedeklenmiş depolama (LRS) özelliği bulunmakta, bu da verilerin aynı veri merkezi içinde yedeklendiği ve böylece veri kaybına karşı koruma sağlandığı anlamına gelir. Standard ve Premium seçenekleri arasındaki temel farklar performans, erişilebilirlik ve maliyettir. Premium katman, genellikle daha yüksek performans ve daha düşük gecikme süresi sunar ama maliyet açısından da daha yüksektir.

### Plan for Azure Storage:

<figure><img src="../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

* **Konum**: Depolama hizmetinin, kullanıcılarınıza yakın bir bölgede olmasını sağlayarak düşük gecikme süreleri elde edin.
* **Uyum**: Verilerin yerel yasalara ve şirket politikalarına uygun şekilde saklandığından emin olun.
* **Maliyet**: İşlem sayısı ve veri erişim sıklığı gibi faktörlerin maliyet üzerinde etkisi olacak, bunları optimize edin.
* **Güvenlik**: Verilerinizin yetkisiz erişime karşı korunmasını sağlayacak güvenlik önlemlerini planlayın.
* **Yönetim Yükü**: Depolama hesaplarını yönetme sürecini göz önünde bulundurarak zaman ve kaynak tahsis edin.
* **Replikasyon**: Veri kaybına karşı koruma ve yüksek erişilebilirlik için uygun replikasyon stratejisi belirleyin.

{% embed url="https://learn.microsoft.com/en-us/azure/well-architected/service-guides/storage-accounts/security" %}

<br>
