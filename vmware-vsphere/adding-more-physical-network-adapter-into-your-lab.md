---
icon: flask
---

# Adding More Physical Network Adapter Into Your Lab

Önceki ağ konseptlerinde sıkça vurgulandığı üzere, gerçek bir "Production" ortamında tek bir fiziksel ağ kartı (Physical NIC) ile çalışmak kabul edilemez bir durumdur. Yönetim (Management), sunucu taşıma (vMotion), harici depolama (Storage) ve sanal makine (VM) trafiklerinin tek bir kabloya sıkıştırılması hem devasa darboğazlara (bottleneck) hem de güvenlik risklerine yol açar. Bu nedenle kurumsal fiziksel sunucuların arkasında genellikle dört, sekiz veya daha fazla fiziksel ağ adaptörü bulunur.

Ancak bir öğrenim laboratuvarı kurduğunuzda, elinizdeki ESXi sunucusu muhtemelen VMware Workstation üzerinde çalışan bir sanal makineden ibarettir ve varsayılan olarak tek bir ağ kartıyla gelir. Gelişmiş ağ topolojilerini, vMotion yeteneklerini veya Fault Tolerance gibi özellikleri test edebilmek için, bu laboratuvar sunucusunu gerçek dünyadaki muadillerine benzetmemiz, yani ona yeni fiziksel ağ kartları eklememiz gerekir.

#### Nested Virtualization ile Donanım Simülasyonu

Fiziksel bir sunucuya gidip PCIe yuvasına yeni bir donanım kartı takmak yerine, bu işlemi VMware Workstation arayüzünden gerçekleştiririz. ESXi sanal makinesinin ayarlarına (Virtual Machine Settings) girerek sisteme yeni "Network Adapter" donanımları eklenmesi, arka planda yepyeni MAC adreslerine sahip sanal çipleri ESXi işletim sistemine gerçek birer donanımmış gibi tanıtır. Bizim tercih ettiğimiz "Bridged" ağ ayarı ise, bu yeni eklenen kartların doğrudan sizin ev veya ofis yönlendiricinizle (Router) aynı ağda IP almasını ve dış dünyayla şeffaf bir iletişim kurmasını sağlar.

#### ESXi Tarafındaki Yansıma: vmnic Terminolojisi

VMware Workstation üzerinden donanım eklendikten sonra ESXi arayüzüne (veya vCenter'a) dönüldüğünde, sistemin bu donanımları nasıl isimlendirdiği çok önemlidir.

ESXi, Linux tabanlı bir çekirdek yapısına sahip olduğu için donanımları "vmnic" ön ekiyle sıralar. Varsayılan gelen ilk kart "vmnic0" olarak adlandırılır ve genellikle Yönetim (Management) ağı için vSwitch0'a bağlıdır. Workstation üzerinden eklediğiniz yeni kartlar ise sırasıyla "vmnic1", "vmnic2" olarak sistemde belirir.

Arayüzdeki "Physical Adapters" sekmesine bakıldığında, yeni eklenen bu vmnic1 ve vmnic2 kartlarının henüz hiçbir Sanal Switch'e (vSwitch) bağlı olmadığı görülür. Bu durum, sistem yöneticisine şu özgürlüğü tanır: Artık vmnic1 kartı sadece vMotion trafiği için ayrı bir VMkernel portuna bağlanabilir veya vmnic2 tamamen bağımsız bir Sanal Switch oluşturularak DMZ simülasyonu için kullanılabilir.

#### Nested Laboratuvarlarda Gözden Kaçan Güvenlik Ayarı

Workstation üzerinde çalışan bir ESXi'ın içine, kendi sanal makinelerinizi kurduğunuzda; içteki sanal makinenin ürettiği ağ paketi, önce ESXi'ın sanal switch'ine, oradan da Workstation'ın ağ kartına ulaşmak zorundadır.&#x20;

Bu nedenle, laboratuvar ortamınıza yeni ağ kartları ekleyip gelişmiş testler yapmaya başladığınızda, hem ESXi üzerindeki vSwitch'te hem de gerekiyorsa Workstation ayarlarında güvenlik profillerini esnetmeniz gerekir. "Promiscuous Mode" ve "Forged Transmits" ayarlarının "Accept" olarak değiştirilmesi, Nested laboratuvar ağının sağlıklı çalışması için yazılı olmayan bir altın kuraldır.

