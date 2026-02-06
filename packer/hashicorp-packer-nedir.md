---
icon: hat-cowboy
---

# HashiCorp Packer Nedir?

Packer, tek bir kaynak yapılandırmasından birden fazla platform için (AWS, VMware, Docker vb.) makine ve konteyner imajları oluşturmanızı sağlayan açık kaynaklı bir araçtır.

#### 1. Temel Kavramlar

Packer ekosistemini oluşturan ana bileşenler şunlardır:

* Templates: Bir imajın nasıl oluşturulacağını detaylandıran, "yemek tarifi"ne benzeyen dosyalardır. Günümüzde HCL2 formatı, bu yapılandırmalar için standart ve tercih edilen yöntemdir.
* Builders: Belirli bir platform (örneğin AWS, Azure, vSphere) üzerinde imajı oluşturan modüllerdir. Packer, vSphere'den Docker'a kadar geniş bir yelpazede resmi destek sunar.
* Provisioners: İmaj statik hale getirilmeden önce, imaj oluşturma sürecinde makineye yazılım yüklemek veya yapılandırma yapmak için kullanılırlar. Shell script'leri, Ansible, Chef veya Puppet gibi araçlar bu aşamada devreye girer.
* Post-processors: İmaj oluşturulduktan sonra gerçekleştirilen işlemlerdir. Örneğin, oluşturulan imajların listesini içeren bir manifest dosyası yazmak veya imajı bir bulut sağlayıcısına yüklemek bu aşamada yapılır.
* Artifacts: Bir işlem sonucunda ortaya çıkan verilerdir. Bu, bir vSphere Builder'ı için dosyalar diziniyken, bir EC2 Builder'ı için AMI ID olabilir.

