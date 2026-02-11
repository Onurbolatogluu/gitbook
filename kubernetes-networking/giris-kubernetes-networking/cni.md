---
icon: table-picnic
---

# CNI

#### ⚙️ Kubernetes Ağ Motoru: CNI (Container Network Interface)

Kubernetes, ağ işlemlerini (IP atama, yönlendirme vb.) doğrudan kendi çekirdek kodunda barındırmaz. Bunun yerine, bu görevleri CNI adı verilen bir standart aracılığıyla harici eklentilere (plugin) devreder. CNI, Kubernetes ağ modelini fiziksel olarak uygulayan mekanizmadır.

**1. Operasyonel Yaşam Döngüsü (Lifecycle Management)**

Bir Pod oluşturulduğunda veya silindiğinde, Node üzerindeki agent olan Kubelet, CNI eklentisini tetikler. CNI şu dört temel görevi yerine getirir:

* Sanal Arayüz Yönetimi: Pod ile Node arasında bağlantıyı sağlayacak sanal arayüzü(veth) oluşturur.
* IP Adres Tahsisi: Daha önce belirlenen Pod CIDR bloğundan boş bir IP adresi seçer ve Pod'a atar. Bu işlem, "IP-per-pod" ilkesinin teknik olarak uygulandığı andır.
* Pod'un dışarıya erişebilmesi veya diğer Pod'larla konuşabilmesi için gerekli yönlendirme tablolarını (routing tables) ve varsa güvenlik duvarı kurallarını (iptables/IPVS) yapılandırır.
* Pod sonlandırıldığında CNI devreye girer, atadığı IP adresini havuza geri iade eder ve sanal arayüzleri siler.

**2. Performans**

* CNI, Pod'lar arası iletişimi NAT olmadan yapılandırır. Bu sayede paketler doğrudan iletilir ve ağ gecikmesi (latency) minimize edilir.
* Binlerce Pod'un IP ataması manuel yapılamaz. CNI, bu süreci tamamen otomatize ederek cluster'ın büyümesine olanak tanır.

**3. CNI Eklenti Ekosistemi**

* Flannel: En temel çözümdür. Genellikle basit bir Overlay Network (katman ağı) kurar. Kurulumu kolaydır ancak gelişmiş güvenlik politikalarını desteklemeyebilir.
* Calico: Kurumsal standartlarda en yaygın kullanılanlardan biridir. BGP (Border Gateway Protocol) kullanarak yönlendirme yapar ve gelişmiş Network Policies ile güvenliği sağlar.
* Cilium: Performans odaklıdır. Linux çekirdeğindeki eBPF teknolojisini kullanarak, paketleri çok yüksek hızda işler. Derinlemesine gözlemlenebilirlik ve güvenlik sunar.
* Weave: Şifreli ağ (encrypted mesh) oluşturma özelliğiyle bilinir.

**4. CIDR ve Bağlanabilirlik İlişkisi**

CNI'nın en kritik sorumluluklarından biri IP Çakışmalarını Önlemektir. CNI eklentisi yapılandırılırken, ona bir IP havuzu (Pod CIDR) verilir. CNI, atadığı IP'lerin, Servis IP'leri veya Node IP'leri ile çakışmadığından emin olarak ağ bütünlüğünü korur.
