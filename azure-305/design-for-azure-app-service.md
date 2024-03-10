# 🛰️ Design for Azure App Service

Azure App Service, web uygulamaları, API'ler ve mobil backend için yönetilen bir hosting hizmetidir. Geliştiricilere, çeşitli programlama dillerini (örneğin, .NET, .NET Core, Java, Ruby, Node.js, PHP veya Python) kullanarak uygulamalarını kolayca oluşturma, dağıtma ve yönetme imkanı sunar. Azure App Service, otomatik ölçeklendirme, yüksek kullanılabilirlik, güvenlik ve izleme gibi önemli bulut özellikleri sağlar, böylece geliştiriciler altyapı yönetiminden ziyade uygulama geliştirmeye odaklanabilirler.

Azure App Service'in ana özellikleri arasında:

1. Azure App Service, Azure SQL Database, Azure Cosmos DB ve daha fazlası gibi diğer Azure hizmetleriyle entegrasyon sağlar.
2. Uygulamanızın trafiğine bağlı olarak kaynakları otomatik olarak ölçeklendirme yeteneği.
3. GitHub, Azure DevOps ve BitBucket gibi popüler kaynak kodu yönetim sistemlerinden doğrudan deploy desteği.
4. SSL sertifikası desteği, otomatik yazılım güncellemeleri (patch) ve Microsoft'un global veri merkezi ağı sayesinde yüksek erişilebilirlik.
5. Uygulamanızı özel bir alan adıyla kullanma yeteneği bulunur.
6. Facebook,Google,Microsoft gibi sosyal platformlardan gelen kimlik bilgilerini kullanarak uygulamalarda otantikasyon (doğrulama) işlemleri için destek sağlar.
7. Linux/Windows konteynerlerini kullanarak, paketlenmiş(image) uygulamalarınızı Azure App Service'te çalıştırabilirsiniz.

<figure><img src="../.gitbook/assets/slot_flow_code_diagam (1).png" alt=""><figcaption></figcaption></figure>

**Azure App Service Deployment Slots**, Azure App Service'te barındırılan bir web uygulaması için birden fazla dağıtım ortamı oluşturmanıza olanak tanıyan bir özelliktir. Bu slotlar, canlı ortamınıza etki etmeden yeni versiyonları test etmenize, yapılandırmaları denemenize ve hatta canlı trafiği yavaşça yeni versiyona aktarmanıza imkan verir. Ana uygulamanız "production" slotunda çalışırken, bir veya birden fazla "staging", "test", "development" gibi ek slotlar oluşturabilirsiniz. Bu slotlar arasında, yeni bir sürümü canlıya almadan önce kapsamlı testler yapmanıza ve yapılandırmaları doğrulamanıza yardımcı olacak şekilde içerik ve yapılandırma ayarlarını takas etme (swap) işlemi gerçekleştirebilirsiniz.



{% embed url="https://learn.microsoft.com/en-us/azure/app-service/" %}

{% embed url="https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots?tabs=portal" %}

