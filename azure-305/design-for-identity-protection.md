# ⛑️ Design for Identity Protection

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Azure Identity Protection, Azure Active Directory'nin bir parçası olarak, kuruluşunuzdaki kimlik tabanlı güvenlik risklerini otomatik olarak algılayıp düzelten bir güvenlik hizmetidir. Makine öğrenimi kullanarak şüpheli etkinlikleri belirler ve potansiyel kimlik hırsızlığı veya kötü niyetli erişim girişimlerine karşı koruma sağlar. Risk tabanlı ve koşullu erişim politikalarını uygulayarak, kullanıcıların oturum açma risk düzeyine göre ek kimlik doğrulama adımları istemesini mümkün kılar.&#x20;

Örnek olarak, bir kullanıcı farklı bir ülkeden nadiren kullanılan bir cihazdan şirketinize ait bir hizmete oturum açmaya çalışıyor. Azure Identity Protection, bu girişimi yüksek riskli olarak algılar çünkü kullanıcının normal davranış modelinden sapmaktadır. Bu durumda, sistem ek doğrulama adımları talep edebilir, kullanıcının şifresini değiştirmesini isteyebilir veya kullanıcının oturum açmasını tamamen engelleyebilir. Bu, şüpheli etkinliklere hızlı bir şekilde yanıt vererek olası bir güvenlik ihlalini önlemeye yardımcı olur.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Best Practices;**

* Azure Identity Protection için gerekli olan Premium P2 lisansı kullanılmalıdır.
* Etkili olacak politikalar oluşturulup, sonuçları aktif bir şekilde gözden geçirilmelidir.
* Acil durum hesapları politikaların dışında tutulmalıdır.
* Oturum açma risk politikasını orta ve üzeri seviyelere ayarlayıp, kullanıcıların kendilerine çözüm bulmalarını sağlanmalıdır.&#x20;
* Identity Protection'dan elde ettiğiniz verileri Conditional Access veya SIEM çözümleri ile entegre edilmelidir.

{% embed url="https://morethanpatches.com/2020/12/04/microsoft-identity-protection-1/" %}
