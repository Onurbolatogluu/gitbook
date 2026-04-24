---
icon: file-lines
---

# OVF and OVA File Formats and Templates

Bir sanal makineyi (VM) oluşturup yapılandırdıktan sonra, sistem yöneticilerinin karşısına sıkça çıkan kritik bir ihtiyaç vardır: Bu makineyi başka bir sunucuya, farklı bir ağa veya bambaşka bir sanallaştırma platformuna nasıl taşırız? Datastore mimarisini ve kaputun altındaki `.vmx`, `.vmdk` gibi dosyaları önceki konularda incelemiştik. Akla gelen ilk (ve en tehlikeli) yöntem, Datastore Browser'a girip bu dosyaları tek tek bilgisayarımıza indirmektir. Ancak bu yöntem, sanallaştırma dünyasının en büyük tuzaklarından biridir.

#### ⛔ Manuel Dosya Kopyalamanın Tehlikeleri

Sanal makine dosyalarını manuel olarak Datastore'dan indirmeye çalıştığınızda, %90'ın üzerinde bir ihtimalle o makineyi hedef sunucuda çalıştıramazsınız.

Çünkü manuel kopyalama sırasında makinenin donanım kimlikleri (UUID), ağ bağdaştırıcı MAC adresleri ve disk bağlantı yolları (metadata) eski sunucuya göre sabit kalır. Yeni sunucuya bu dosyaları attığınızda, hipervizör makineyi tanıyamaz veya diskleri bulamaz.

Güvenli, standart ve profesyonel taşıma işlemi "Export" prosedürü ile yapılır. Bu prosedür bize iki evrensel format sunar: OVF ve OVA.

#### 🌐 OVF (Open Virtualization Format) Nedir?

OVF, sanal makinelerin paketlenmesi ve dağıtılması için geliştirilmiş açık ve evrensel bir standarttır. Bir sanal makineyi OVF olarak dışarı aktardığınızda (Export), sistem makinenizi "donanımdan bağımsız" hale getirir. Bu işlemin en büyük avantajı Cross-Platform desteğidir. Yani VMware ESXi üzerinde oluşturduğunuz bir OVF paketini; Microsoft Hyper-V, Oracle VirtualBox veya KVM gibi bambaşka hipervizörlere sorunsuzca aktarabilir (Import) ve çalıştırabilirsiniz.

Dosya Yapısı: OVF bir tek dosya değil, bir dosyalar bütünüdür. Dışarı aktarma işlemi bittiğinde tarayıcınız genellikle size iki ana dosya indirir:

1. `.ovf` Dosyası: Sanal makinenin donanım özelliklerini (RAM, CPU, Disk yapısı) barındıran, sadece birkaç kilobayt boyutundaki XML tabanlı yapılandırma dosyasıdır.
2. `.vmdk` Dosyası: Makinenin asıl verilerinin bulunduğu, sıkıştırılmış sanal disk dosyasıdır. (İşletim sistemini taşır).

_Not: Bir makineyi dışarı aktarabilmek için makinenin kapalı (Power Off) durumda olması zorunludur. Açık bir makine export edilemez._

#### 🗜️ OVA (Open Virtual Appliance) Nedir?

OVA, en basit tabirle OVF paketinin fermuarlanmış (TAR arşivi yapılmış) tek dosyalık halidir. Bir USB belleğe atıp götürmek veya internet üzerinden birine göndermek istediğinizde, `.ovf` ve birden fazla `.vmdk` dosyasını ayrı ayrı taşımak risklidir (dosyalardan biri kaybolabilir). OVA formatı, bu dosyaların tamamını tek bir `.ova` uzantılı dosya içine hapseder.

_(Ekstra Bilgi: ESXi 6.5 ve sonrasındaki modern Web arayüzleri, Export işleminde genellikle tek parça OVA oluşturmak yerine `.ovf` ve `.vmdk` dosyalarını ayrı ayrı indirmeyi standart hale getirmiştir. Ancak sistem, dışarıdan gelen OVA dosyalarını içeri aktarmayı (Deploy) hala kusursuz bir şekilde destekler.)_

#### 🛒 Virtual Appliance Kavramı ve Hazır Sistemler

OVF ve OVA formatlarının BT sektörüne kattığı en büyük değer "Hazır Sanal Cihazlar" (Virtual Appliances) ekosistemidir.

Diyelim ki şirketinize bir Linux tabanlı Web Sunucusu (LAMP Stack) veya bir Ağ İzleme Yazılımı kuracaksınız. Sıfırdan bir sanal makine oluşturmak, ISO bağlamak, Linux kurmak ve yazılımları yapılandırmak saatlerinizi alır.

Bunun yerine VMware Marketplace veya Bitnami gibi platformlara girerek, uzmanlar tarafından önceden kurulmuş, ayarlanmış ve optimize edilmiş bir sistemi OVA veya OVF formatında hazır olarak indirebilirsiniz. Tek yapmanız gereken vSphere arayüzünden "Deploy OVF Template" seçeneğine tıklamak, indirdiğiniz dosyayı göstermek ve birkaç dakika içinde anahtar teslim sunucunuzu ayağa kaldırmaktır.



