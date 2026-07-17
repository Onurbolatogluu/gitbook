---
icon: right-left-large
---

# Types of Virtual Switch Available in VMware

Sanallaştırma altyapılarında ağ yönetimi kurgulanırken, sistem yöneticilerinin karşısına çıkan en temel yol ayrımı kullanılacak Virtual Switch türüdür. VMware vSphere ekosisteminde yer alan iki temel ağ mimarisi mevcuttur: Standard Switch ve Distributed Switch.

**1. Host Seviyesinde Yerel Yönetim: Standard Switch**

Standard Switch, VMware ağ mimarisinin en temel ve yapı taşı konumundaki bileşenidir. Doğrudan tek bir ESXi sunucusu (Hypervisor) seviyesinde çalışır.

* Bir sunucuya ESXi kurduğunuz anda, sistem varsayılan olarak bir Standard Switch yaratır ve içine `VM Network` adında bir ağ ekler. Bu sayede kurduğunuz ilk Virtual Machine anında ağa çıkabilir hale gelir.
* Her ESXi Host kendi Standard Switch konfigürasyonunu kendi içinde tutar. Bir sunucudaki switch ayarı, yanındaki diğer sunucuyu etkilemez.
* Birçok standart ortam için Standard Switch kusursuz çalışır. Küçük ve orta ölçekli yapılarda ek bir karmaşaya girmeden ağ ihtiyaçlarını fazlasıyla karşılar. Ancak sunucu sayısı arttıkça (örneğin 20 farklı Host olduğunda), aynı ağ ayarını 20 sunucuya tek tek elle girme zorunluluğu operasyonel bir yük yaratır.

**2. Merkezi Kontrol: Distributed Switch**

Distributed Switch, sanal ağ yönetimini bireysel Host'ların elinden alıp tamamen vCenter Server üzerine taşıyan gelişmiş bir mimaridir.

* Distributed Switch doğrudan vCenter seviyesinde yaratılır. Eğer ortamınızda bir vCenter Server yoksa, bu mimariyi kullanamazsınız.
* vCenter üzerinde tek bir devasa Distributed Switch yaratırsınız ve ortamınızdaki tüm ESXi Host'ları bu switch'e bağlarsınız. Yeni bir ağ (VLAN) eklemeniz gerektiğinde, bunu 20 sunucuya tek tek girmek yerine vCenter üzerinden sadece bir kez yaparsınız ve tüm sunuculara anında uygulanır.
* Bir Virtual Machine'i bir Host'tan diğerine taşırken (vMotion), iki sunucu da aynı merkezi Distributed Switch'e bağlı olduğu için makine ağ bağlantısında hiçbir tutarsızlık yaşamaz ve kesintisiz çalışmaya devam eder.

**3. Standard Switch ve Distributed Switch Karşılaştırması**

| **Özellik**         | **Standard Switch**                                               | **Distributed Switch**                                                      |
| ------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Yönetim Katmanı     | ESXi (Host) Seviyesi                                              | vCenter Server Seviyesi                                                     |
| Kapsam              | Sadece bulunduğu tek bir Host'u kapsar.                           | Cluster içindeki tüm Host'ları kapsar.                                      |
| Kurulum Gereksinimi | ESXi kurulumuyla varsayılan olarak gelir.                         | vCenter Server gerektirir.                                                  |
| Operasyonel Yük     | Çoklu Host ortamlarında her sunucu için manuel tekrar gerektirir. | Tek tıkla tüm Host'lara ağ politikaları dağıtılabilir.                      |
| Gelişmiş Özellikler | Temel ağ iletimini sağlar.                                        | Ağ trafiğini izleme, şekillendirme ve gelişmiş güvenlik politikaları sunar. |

Virtual Networking kurgusunda sistem yöneticileri, ortamın büyüklüğüne ve yönetim ihtiyaçlarına göre bir seçim yapmalıdır.

Eğer az sayıda sunucunuz varsa, her sunucunun kendi içinde barındırdığı bağımsız Standard Switch yapısı işinizi görecektir. Ancak altyapınız büyüyor, Virtual Machine'ler sunucular arasında sürekli göç ediyor ve ağ politikalarını tek bir merkezden, hatasız bir şekilde yönetmek istiyorsanız; vCenter'ın gücünü arkanıza alarak Distributed Switch mimarisine geçiş yapmak, operasyonel yükünüzü ortadan kaldıracak en profesyonel adımdır.
