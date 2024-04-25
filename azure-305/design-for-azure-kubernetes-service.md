# 🎢 Design for Azure Kubernetes Service

Azure Kubernetes Service (AKS), Microsoft'un bulut platformu Azure üzerinde Kubernetes konteyner orkestrasyon hizmetidir. AKS, uygulamalarınızın konteynerize edilmiş versiyonlarını dağıtmanıza, yönetmenize ve ölçeklendirmenize yardımcı olur.

<figure><img src="../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

1. **Pools**: Bu, aynı yapılandırmaya sahip makinelerin(nodes) mantıksal bir gruplamasıdır. AKS'de bunlar genellikle node pool olarak adlandırılır ve belirli türde iş yüklerini çalıştırmak için optimize edilmiş olabilir.
2. **Nodes**: Bunlar, konteynerize uygulamaların çalıştığı sanal makinelerdir (VM'ler). Node 'lar, Kubernetes master node(lar) tarafından yönetilir ve bu master node kullanıcıdan gizlidir.
3. **Pods**: En küçük dağıtım birimi olup, uygulamanızın tek bir örneğini temsil eden bir veya daha fazla konteyner koleksiyonudur. Bir node 'da birden fazla pod bulunabilir.
4. **Deployment**: Pod'unuzun bir veya daha fazla aynı kopyasını oluşturur. Bu, uygulamanızın istenen sayıda örneğinin çalıştığından emin olmak ve gerektiğinde otomatik olarak ölçeklendirmek için kullanılır.
5. **Manifest**: Kubernetes'te dağıtım için kullanılan YAML veya JSON formatındaki dosyadır. Bu dosya, Kubernetes'e hangi kaynakların (pods, services, volumes vb.) oluşturulması gerektiğini ve nasıl yapılandırılması gerektiğini söyler.

Her bir node içinde, birden fazla pod bulunabilir ve bir pod içinde bir veya daha fazla konteyner yer alabilir. Deployment nesneleri, podların istenilen durumda olmasını sağlamak için kullanılır ve bu durum manifest dosyaları kullanılarak tanımlanır.

AKS, tüm bu kavramları otomatik olarak yöneten ve yüksek kullanılabilirlik, güvenlik, ölçeklenebilirlik gibi özellikleri sağlayan bir hizmettir. Bu sayede, geliştiriciler uygulama koduna odaklanabilir ve altyapı yönetimi ile ilgili zorluklardan kurtulmuş olur.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Kubernetes düğüm havuzları (node pools), farklı tipte iş yüklerini veya uygulama bileşenlerini çalıştırmak için farklı yapılandırmalara sahip düğüm gruplarıdır. Her bir node pool, örneğin CPU veya bellek gibi kaynaklar açısından farklılık gösterebilir.&#x20;

\
Örneğin, yüksek CPU gereksinimlerine sahip uygulama bileşenleri için daha güçlü makineler, bellek yoğun iş yükleri için ise daha fazla belleğe sahip makineler kullanabilirsiniz. Bazı node 'lar daha düşük maliyetli olabilir ve daha az kritik iş yükleri için kullanılabilirken, diğerleri daha yüksek performanslı ve dolayısıyla daha maliyetli olabilir. \


Bu yapılandırma esnekliği, Kubernetes'in iş yüklerini mikro hizmet mimarilerindeki gibi çeşitli bileşenlere bölme ve her bir bileşeni en uygun kaynaklarla çalıştırma yeteneği ile iyi bir uyum sağlar.
{% endhint %}

{% embed url="https://learn.microsoft.com/en-us/azure/aks/" %}

