# 💼 Desing Azure AD (Entra ID)

Azure Active Directory, Microsoft'un bulut tabanlı kimlik ve erişim yönetimi hizmetidir.&#x20;

**Azure AD için Kullanıcı Tipleri:**

1. **Cloud only users:** Bu kullanıcılar yalnızca bulut hizmetleri için var olan ve on-premise (yerel) altyapılarla senkronize edilmeyen kullanıcı hesaplarıdır. Yani, bu kullanıcılar doğrudan Azure AD'de oluşturulur ve yalnızca bulutta yönetilir.
2. **Guest users (B2B):** Azure AD B2B özelliği, iş ortaklarının, tedarikçilerin ve diğer dış grupların şirketin kaynaklarına güvenli bir şekilde erişmesine olanak tanır. Bu kullanıcılar, genellikle diğer organizasyonlardan gelen ve geçici erişim gereksinimleri olan kişilerdir.
3. **Directory synchronized users:** Bu kullanıcılar, şirketin yerel Active Directory'sinden Azure AD'ye senkronize edilmiş kullanıcı hesaplarıdır. Bu, yerel ve bulut hizmetlerinin birleştirilmesini sağlayan hibrit bir kimlik çözümü sunar.

**Best Practices:**

1. **En Az Yetki İlkesi**: Bu ilke, yöneticilere sadece işlerini yapmaları için gereken yetkilerin verilmesini ifade eder. Amacı, bir güvenlik sorunu yaşandığında zararı en aza indirmektir. Microsoft Entra'da, farklı roller arasından seçim yapabilir ve ihtiyaç duyulan özel rolleri oluşturabilirsiniz.
2. **Ayrıcalıklı Kimlik Yönetimi (PIM) Kullanımı**: PIM, yöneticilere yalnızca ihtiyaç duydukları zamanlarda belirli rolleri kullanma izni verir. Bu rolün süresi sınırlıdır ve belirlenen süre sonunda otomatik olarak kaldırılır.
3. **Çok Faktörlü Kimlik Doğrulama (MFA)**: Yönetici hesapları için MFA'nın kullanılması, hesapların güvenliğini artırır. MFA, kullanıcıların sadece şifreyle değil, aynı zamanda ek bir doğrulama yöntemiyle (örneğin telefon onayı) sisteme giriş yapmalarını sağlar.
4. **Düzenli Erişim İncelemeleri**: Yöneticilerin sistem erişimlerinin periyodik olarak gözden geçirilmesi önemlidir. Bu incelemeler, artık gerekli olmayan erişim izinlerinin kaldırılmasına yardımcı olur.
5. **Yöneticilerin Sınırlandırılması**: Güvenlik açısından, çok az sayıda (tercihen beşten az) Yönetici olması önerilir. Ayrıca, acil durumlar için en az iki yedek hesap bulundurulmalıdır.
6. **Koşullu Erişim Politikaları**: Bu politikalar, kimin, nereden ve hangi koşullar altında kurumsal kaynaklara erişebileceğini kontrol etmenize yardımcı olur. Özellikle eski ve güvensiz kimlik doğrulama yöntemlerinin kullanımını engellemek için kullanılırlar.
7. **Parola Yönetimi**: Kullanıcıların kendi parolalarını sıfırlayabilmelilerdir. Ayrıca, bulut tabanlı parola politikalarını şirket içi sistemlere de uygulamak faydalıdır.
8. **Rol Tabanlı Erişim Kontrolü (RBAC)**: RBAC, kullanıcıların Azure kaynaklarına erişimlerini yönetmenize yardımcı olur. Bu, hangi kullanıcının ne yapabileceğini ve hangi kaynaklara erişebileceğini belirlemenize olanak tanır. Güvenlik ekiplerine, riskleri değerlendirebilmeleri ve gerekli müdahaleleri yapabilmeleri için uygun izinlerin verilmesi önemlidir.

{% embed url="https://learn.microsoft.com/en-us/azure/security/fundamentals/identity-management-best-practices" %}
