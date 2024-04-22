# ⏰ Azure Site Recovery

<figure><img src="../.gitbook/assets/disaster-recovery-smb-azure-site-recovery (1).png" alt=""><figcaption></figcaption></figure>

Azure Site Recovery (ASR) bir felaket kurtarma hizmetidir ve Microsoft Azure bulut platformunun bir parçasıdır. Hizmet, Azure'da, şirket içinde (on-premise) veya diğer bulut sağlayıcılarında barındırılan sunucular ve uygulamalar için kurtarma çözümleri sunar.

1. **BCDR:** ASR, altyapınızı ikincil bir bölgeye replike etmenize olanak tanır. Bu, felaket anında, yani birincil bölgeye bir şey olduğunda, sistemlerinizi/uygulamalarınızı otomatik olarak veya manuel olarak ikincil bölgeye geçirebilmenizi sağlar.
2. **Migration:** Azure Migrate hizmeti olmasına rağmen, ASR da on-premise altyapınızı Azure'a taşımanıza imkan tanır. Bu sayede, altyapınızın replikasyonu tamamlandığında, orijinal kaynakları durdurmadan (cut over) yeni ortama geçiş yapabilirsiniz.
3. **Replikasyon:** ASR, Azure sanal makinelerinizi (VM) farklı bölgeler arasında çoğaltmanıza olanak tanır. Bu, birincil bölgenizde bir sorun olması durumunda, verilerin ve uygulamaların kullanılabilirliğini korumak için önemlidir.
4. **Retention:** Çoğaltılan veri noktalarının ne kadar süreyle saklanacağını tanımlayabilirsiniz. Bu, felaket kurtarma sürecinde esneklik sağlar ve belirli bir süre zarfı içindeki verileri kurtarabilirsiniz.

{% embed url="https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview" %}
