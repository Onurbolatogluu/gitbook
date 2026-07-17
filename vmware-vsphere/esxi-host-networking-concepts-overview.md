---
icon: paintbrush-fine
---

# ESXi Host Networking Concepts Overview

VMware vSphere altyapısında ağ mimarisini kurarken, her şeyin başladığı bir "sıfır noktası" vardır.&#x20;

### 1. Sıfır Noktası: vSwitch0 ve Temel Topoloji

Bir fiziksel sunucuya ESXi işletim sistemini kurduğunuz anda, sistem sizi ağsız bırakmamak için varsayılan olarak vSwitch0 adında bir Standard Switch yaratır. Sunucunun arkasındaki ilk fiziksel ağ kartı (vmnic0) bu switch'e Uplink olarak atanır.

Arayüzdeki "Topology" görünümü, ağ trafiğinin akış yönünü görselleştiren kusursuz bir haritadır. Bu topoloji haritasına baktığımızda veri akışının şu 3 temel katmandan oluştuğunu görürüz:

* Sol Taraf (Sanal Bileşenler): Virtual Machine'ler ve ESXi'ın kendi yönetim arayüzleri.
* Orta Kısım (Standard Switch): Trafiği toplayan ve yönlendiren yazılımsal merkez.
* Sağ Taraf (Physical Adapter / Uplink): İçerideki sanal trafiği dış dünyadaki fiziksel ağa çıkaran gerçek donanım kapısı.

### 2. VM Port Group vs. VMkernel Port

VMware ağ mimarisinin en kritik odak noktası burasıdır. Standard Switch üzerinde oluşturulan Port Group'lar işlevlerine göre iki ana kategoriye ayrılır. Bunları birbirine karıştırmak, ağ tasarımında yapılabilecek en büyük hatadır:

#### A. Virtual Machine Port Group (VM Port Group)

Adından da anlaşılacağı gibi, bu port grupları sadece sanal makinelerin (Virtual Machine) ağa veya internete çıkması için kullanılır.

* İçeride çalışan Windows veya Linux işletim sistemlerinin ürettiği standart veri trafiği bu portlardan geçer.
* Varsayılan kurulumda gelen `VM Network` bağlantısı, standart bir VM Port Group örneğidir.

#### B. VMkernel Port (Hypervisor'ın Kendi Ağı)

VMkernel portları, sanal makinelerin değil, doğrudan ESXi işletim sisteminin (Hypervisor) kendisinin ağla konuşabilmesi için yaratılmış özel IP alan bağlantılarıdır. ESXi sunucunuz aşağıdaki altyapı işlemlerinden herhangi birini yapacaksa, kesinlikle bir VMkernel portuna ihtiyaç duyar:

* Management: Sizin sunucuya arayüzden veya SSH ile bağlanırken kullandığınız yönetim trafiği.
* vMotion: Bir Virtual Machine'i kapatmadan başka bir Host'a taşırken, RAM üzerindeki gigabaytlarca verinin aktığı özel trafik yolu.
* Fault Tolerance: İki Host üzerindeki sanal makinelerin eşzamanlı ve kesintisiz çalışması için gereken anlık senkronizasyon trafiği.
* IP Storage (iSCSI / NFS): Sunucunun dışarıdaki depolama ünitelerine (Storage) erişirken kullandığı veri yolu trafiği.

> 💡 Güvenlik ve performans (QoS) nedenleriyle VMkernel trafiği ile Virtual Machine trafiği asla aynı VLAN üzerinde barındırılmamalıdır. Hatta vMotion ve iSCSI gibi yoğun veri aktarımı yapan VMkernel portları için fiziksel sunucu arkasında ayrı Uplink'ler tahsis etmek en sağlıklı mimari karardır.

### 3. Layer 2 Sadeliği

_"Standard Switch, Layer 2 seviyesinde çalışır."_

Bunun pratik anlamı şudur: Standard Switch'in içinde bir IP yönlendirme (Routing) tablosu yoktur. Cisco veya Juniper gibi fiziksel ağ switchlerinde komut satırına girip saatlerce konfigürasyon yapmanız, Spanning Tree protokolleri yazmanız veya karmaşık kurallar tanımlamanız gerekebilir.

Ancak ESXi içindeki Standard Switch tamamen "Tak-Çalıştır" (Unmanaged Layer 2 Switch) mantığıyla çalışır. Siz sadece Port Group'u oluşturur, VLAN numarasını (Tag) yazar ve Virtual Machine'i bu gruba bağlarsınız. Geri kalan tüm karmaşık yönlendirme işlemleri ESXi'ın dışında, şirketinizin fiziksel ağ cihazlarında çözülür.

### Özet

Eğer doğrudan ESXi sunucusunun kendi arayüzüne giriş yaparsanız, yalnızca o sunucunun içindeki Standard Switch'i görebilir ve yönetebilirsiniz. Distributed Switch mimarisi, doğası gereği merkezi bir yapı olduğu için ESXi ekranında hiç görünmez. Bu gelişmiş mimariyi kurmak veya yönetmek istiyorsanız, işlemleri doğrudan vCenter arayüzü üzerinden gerçekleştirmeniz gerekir.

Bu temel yapıda unutulmaması gereken altın kural; Virtual Machine'lerin dış dünyaya çıkışı için VM Port Group, ESXi sunucusunun altyapı operasyonları (Management, vMotion, Storage) için ise her zaman VMkernel Port kullanılması gerektiğidir.
