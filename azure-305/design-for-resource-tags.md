# 📟 Design for resource tags

Azure kaynak etiketleri (tags), kaynakları kategorilere ayırmak, maliyet tahsisi, yönetim, organizasyon veya politika uygulama gibi amaçlar için meta veri atamak amacıyla kullanılır. Etiketler, kaynakları daha verimli bir şekilde yönetmenize yardımcı olur.

<figure><img src="../.gitbook/assets/tag-example-01 (1).svg" alt=""><figcaption></figcaption></figure>

* Organizasyon genelinde standart bir etiketleme şeması belirlenmelidir. Bu, etiketlerin tutarlı ve anlamlı olmasını sağlar.
* Etiket adlarının ve değerlerinin okunabilir ve kolayca tanımlanabilir olmasına önem verilmelidir.
* Etiket adları ve değerleri için belirli kurallar ve konvansiyonlar geliştirilmelidir. Örneğin, departman isimleri, ortam türleri (prod, dev, test) veya maliyet merkezleri gibi.
* Büyük/küçük harf kullanımı, boşluk kullanımı (veya kullanılmaması) ve özel karakterler hakkında net kurallar konulmalıdır.
* Zorunlu etiketler tanımlanarak kaynakların uygun şekilde etiketlendiğinden emin olunmalıdır. Örneğin, her kaynağın bir "Ortam" ve "Sorumlu" etiketi olması gerektirebilir. [https://medium.com/@geralexgr/azure-policy-require-specific-tags-on-resources-72b911f6c725](https://medium.com/@geralexgr/azure-policy-require-specific-tags-on-resources-72b911f6c725)\
  [https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/enable-tag-inheritance](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/enable-tag-inheritance)
* Kaynakların maliyetini izlemek ve analiz etmek için etiketler kullanılmalıdır. Bu, maliyet merkezi, proje kodu veya bütçe kodu gibi etiketler aracılığıyla yapılabilir.
* Kaynak gruplarını otomatik olarak yönetmek ve yapılandırmak için etiketler kullanılmalıdır. Örneğin, belirli bir etikete sahip kaynaklara otomatik yedekleme politikaları uygulanabilir.
* Maliyet raporlama, kaynak kullanımı analizi ve optimizasyon çalışmaları için etiketler kullanılarak kaynaklar kolayca sınıflandırılmalı ve filtrelenmelidir.
* İş ihtiyaçları değiştikçe etiketleme şeması düzenli olarak gözden geçirilmeli ve güncellenmelidir. Eski veya artık kullanılmayan etiketler temizlenmelidir.
* Azure Policy gibi araçlar kullanılarak etiketleme kuralları otomatik olarak uygulanmalı ve etiketleme süreçleri kolaylaştırılmalıdır.



{% embed url="https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-tagging" %}

{% embed url="https://blog.hametbenoit.info/2022/12/13/azure-you-can-now-inherit-tags-for-cost-management/" %}
