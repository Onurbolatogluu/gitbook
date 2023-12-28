# 💳 Design for MFA and Conditional Access

<figure><img src="../.gitbook/assets/RE4Mvu1.avif" alt=""><figcaption></figcaption></figure>

Azure Conditional Access, Microsoft Azure platformunun bir parçası olan ve şirket kaynaklarına erişim üzerinde koşullu denetimler sağlayan bir güvenlik özelliğidir. Bu özellik, Azure Active Directory (Azure AD) ile entegre çalışarak, belirli koşullar altında kullanıcılara uygulamalara ve verilere erişim izni verilip verilmeyeceğini tanımlamanıza olanak tanır.

* Organizasyonlar, kullanıcıların cihaz durumu, konum, kullanıcı risk seviyesi, uygulama hassasiyeti gibi çeşitli faktörler temelinde erişimini kontrol edebilirler. Bu koşullar, örneğin belirli bir ülkeden gelen erişim talepleri veya belirli bir cihaz sınıfı kullanıldığında uygulanabilir.
* &#x20;IT yöneticileri, kimin, ne zaman, nereden ve hangi koşullar altında şirket kaynaklarına erişebileceğini tanımlayan politikalar oluşturabilir. Bu politikalar, oturum açma işlemlerinde çok faktörlü kimlik doğrulama (MFA) zorunluluğu, belirli uygulamalar için belirli güvenlik gereksinimleri gibi erişim kontrollerini içerebilir.
* Azure Conditional Access, Azure AD Identity Protection ile entegre olarak riskli oturum açma davranışlarını belirleyebilir ve bu risklere göre otomatik tepkiler verebilir. Örneğin, bir kullanıcının hesabı tehlike altındaysa, sistem otomatik olarak ek kimlik doğrulama adımları talep edebilir.
* Kullanıcıların erişim istekleri anında değerlendirilir ve ilgili Conditional Access politikalarına göre erişim izni verilir veya reddedilir.
* Azure Conditional Access, 'sıfır güven' güvenlik modelinin bir yansımasıdır, bu da varsayılan durumun 'güvensiz' olduğu ve her erişim noktasının doğrulanması gerektiği anlamına gelir.

Azure Conditional Access ile kontrol edebileceğiniz çeşitli faktörler şunları içerir:

1. Kullanıcıların bilinmeyen bir konumdan veya daha önce kaydedilmemiş bir IP adresinden giriş yapmaya çalıştıklarında ek bir güvenlik katmanı olarak MFA'yı aktive edebilirsiniz. Bu, yetkisiz erişimi önemli ölçüde azaltabilir.
2. Eğer belirli ülkelerden hiçbir zaman giriş beklemiyorsanız, bu ülkelerden gelen oturum açma isteklerini engelleyebilirsiniz. Bu, coğrafi konumunuza göre olası zararlı girişimleri otomatik olarak bloke eder.
3. Kurumsal veri ve uygulamalara erişim için şirket tarafından yönetilen ve güvenli olduğu bilinen cihazların kullanımını şart koşun. Bu, güvenli olmayan veya kontrol edilmeyen cihazlardan kaynaklanabilecek güvenlik zafiyetlerini azaltır.
4. Kullanıcıların sadece onaylanmış ve güvenli bulduğunuz uygulamalar üzerinden verilere erişmesine izin verin. Bu, uygulama ekosisteminizi kontrol altında tutmanıza ve veri sızıntısı riskini azaltmanıza yardımcı olur.
5. Eğer bir kullanıcının hesabının tehlikede olduğunu düşünüyorsanız, bu hesap için şifre değişikliği zorunlu hale getirebilir, yüksek risk altında olduğunu düşündüğünüz kullanıcılar için MFA'yı aktive edebilirsiniz.
6. Belirli bir uygulamaya erişimin riskli olduğunu düşünüyorsanız, bu uygulamaya erişimi tamamen engelleyecek politikalar uygulayabilirsiniz.
7. Eski kimlik doğrulama protokolleri, güvenlik açıkları içerebilir ve saldırganlar tarafından kolayca istismar edilebilir. Bu yüzden bu tür eski protokollerin kullanımını engelleyerek sisteminizi daha güvenli hale getirebilirsiniz.
8. Politika değişikliklerinin etkisini önceden görmek için What-If aracını kullanabilirsiniz. Bu, planladığınız değişikliklerin beklenen etkilerini doğrulamanıza ve olası sorunları önceden tespit etmenize yardımcı olur.
9. Bir politikayı tam olarak uygulamadan önce, raporlama-modunu kullanarak politikanın gerçekte nasıl çalıştığını test edebilirsiniz. Bu mod, politikanın koşulları sağlandığında ne tür bir etki yaratacağını gösterir, ancak gerçek bir erişim engellemesi yapmaz.

{% embed url="https://www.gitbit.org/course/ms-500/blog/9-conditional-access-policies-youll-kick-yourself-for-not-setting-up-crntbkjzc" %}
