# Desing For Blueprints

Azure Blueprints, Azure abonelikleriniz üzerinde tekrarlanabilir, denetlenebilir ve uyumlu ortamlar oluşturmak için kullanılan bir hizmettir. Azure Blueprints, bulut kaynaklarının ve yapılandırmalarının önceden tanımlı bir setini kullanarak, güvenlik, tasarım ve uyumluluk gereksinimlerini karşılayacak şekilde Azure ortamlarını hızlı ve tutarlı bir şekilde kurmanıza olanak tanır. Azure Blueprints, Resource Groups, Role Assignments, Policy Assignments, ARM Templates ve daha fazlasını içerebilir.



<figure><img src="../.gitbook/assets/image (237).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://learn.microsoft.com/en-us/azure/governance/blueprints/overview" %}

### **ARM. VS BLUEPRINTS;**

* ARM şablonları daha çok kaynak dağıtımı ve yapılandırması üzerine odaklanırken, Azure Blueprints bir Azure ortamının genel yapısını, güvenlik politikalarını, rol atamalarını ve uyumluluk standartlarını kapsar.
* Blueprints, bir ortamın birden fazla kez ve tutarlı bir şekilde kurulmasını sağlamak üzere tasarlanmıştır. ARM şablonları ise genellikle belirli bir dağıtım senaryosuna odaklanır.
* Blueprints, ARM şablonlarını içerebilir, bu da Blueprints'in ARM şablonlarının sunduğu tüm özellikleri ve esnekliği desteklediğini gösterir. Ayrıca, güvenlik politikaları ve rol atamaları gibi ek yapılandırma öğelerini de entegre eder.

Kısacası, Azure Blueprints ve ARM şablonları, Azure kaynaklarını yönetme ve dağıtma sürecinde birlikte çalışabilir. Blueprints, ARM şablonları dahil olmak üzere çeşitli yapılandırma öğelerini bir araya getirerek geniş kapsamlı ortam yapılandırmalarını ve uyumluluğu yönetmenizi sağlar.

<figure><img src="../.gitbook/assets/blueprints.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.linkedin.com/pulse/understanding-differences-between-azure-blueprints-arm-jason-zandri/" %}

{% embed url="https://www.scheer-group.com/en/company/blog/infrastructure-as-a-code-with-azure-blueprint/" %}



***



### Bonus:&#x20;

<figure><img src="../.gitbook/assets/cloud-broker-landing-zone.svg" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Azure Landing Zone, Azure bulut platformunda güvenli ve ölçeklenebilir bir altyapı kurulumunu kolaylaştırmak için Microsoft tarafından önerilen bir dizi uygulama ve yönergeler setidir. Bu kavram, kuruluşların Azure'a geçiş yaparken veya bulut altyapılarını genişletirken ihtiyaç duydukları temel yapılandırmaları, güvenlik politikalarını ve yönetim araçlarını içerir. Azure Landing Zone, bir bulut ortamını hızlı bir şekilde kurmanıza ve bu ortamı iş yükleriniz için hazır hale getirmenize yardımcı olur, böylece bulut kaynaklarınızı daha etkin bir şekilde yönetebilirsiniz.
{% endhint %}

{% embed url="https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/" %}