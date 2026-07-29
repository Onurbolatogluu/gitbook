---
icon: house-laptop
---

# Add a Virtual Machine Port Group to a Standard Switch

Uplink'i bağlanmış ancak üzerinde henüz hiçbir port group bulunmayan bir Standard Switch, fiziksel ağa açılan ama iç tarafında kimsenin bağlanamadığı bir yapıdır. Switch'i işlevsel hale getiren adım, sanal makinelerin bağlanacağı **Virtual Machine Port Group**'ların tanımlanmasıdır.

Bu makalede yeni bir port group'un nasıl oluşturulacağını adım adım ele alacak; port group ile VMkernel portu arasındaki kritik ayrımı netleştirecek, VLAN ID alanının gerçekte ne işe yaradığını açıklayacak ve oluşturulan port group'a sanal makinelerin nasıl bağlandığını göstereceğiz.

### Port Group mu, VMkernel mi? Ayrımı Netleştirelim

Bir Standard Switch üzerinde iki farklı iç bağlantı tipi oluşturabilirsiniz ve doğru olanı seçmek, tasarımın ilk kararıdır:

* **Virtual Machine Port Group:** Sanal makinelerin vNIC'lerinin bağlandığı porttur. Production, Test/Dev, DMZ gibi iş yükü ağları bu tiple tanımlanır. VM trafiği taşır; host'un kendi servisleriyle ilgisi yoktur.
* **VMkernel Port:** ESXi host'un _kendi_ trafiğinin (management, vMotion, iSCSI, vSAN, Fault Tolerance) çıkış noktasıdır. Sanal makine bağlanamaz.

Varsayılan kurulumdaki `vSwitch0` bu ayrımı zaten örnekler: **VM Network** bir port group'tur ve sanal makineleri taşır; **Management Network** ise bir VMkernel portudur ve üzerine VM bağlanamaz. Yeni switch'inizde sanal makine barındırmak istiyorsanız oluşturmanız gereken, port group'tur.

### Adım Adım: Yeni Port Group Oluşturma

ESXi Host Client üzerinde **Networking → Port groups → Add port group** yolunu izleyin. Karşınıza çıkan alanlar:

#### 1. Name

Port group'un adı — örneğin `VM-NewSwitch` ya da daha iyisi, ağın amacını anlatan bir isim: `PG-Production-VLAN10`, `PG-DMZ` gibi.

İsimlendirme burada kozmetik bir detay değildir; **operasyonel bir zorunluluktur.** Sanal makineler ağlara port group _adıyla_ bağlanır. Cluster ortamında vMotion, hedef host'ta **birebir aynı isimde** bir port group arar; `Production` ile `production` bile farklı ağlar olarak değerlendirilir ve migration uyumluluk hatasıyla sonuçlanır. Bu nedenle:

* Tüm host'larda aynı isimlendirme standardını harfi harfine uygulayın.
* İsme VLAN bilgisini dahil etmek (`PG-App-VLAN20`), yapılandırma denetimini kolaylaştıran yaygın bir pratiktir.

#### 2. Virtual Switch

Port group'un hangi switch'e bağlanacağını burada seçersiniz. Host'ta birden fazla Standard Switch varsa (`vSwitch0` ve yeni oluşturduğunuz switch gibi) tümü listelenir. Bu seçim, port group'a bağlanacak makinelerin **hangi uplink'ler üzerinden, hangi fiziksel ağa** çıkacağını belirler — yani port group'u yanlış switch'e bağlamak, VM'leri yanlış fiziksel altyapıya bağlamak demektir.

#### 3. VLAN ID

Bu alan, aynı fiziksel altyapı üzerinde birden fazla mantıksal ağ işletmenin anahtarıdır. Standard Switch'te üç anlamlı değer aralığı vardır:

* **0 (varsayılan):** VLAN etiketleme yapılmaz (External Switch Tagging — EST). Trafik switch'ten etiketsiz çıkar; VLAN ayrımı tamamen fiziksel switch portunun access VLAN yapılandırmasına bırakılır.
* **1–4094:** Virtual Switch Tagging (VST) — en yaygın kullanılan mod. vSwitch, bu port group'tan çıkan trafiğe belirtilen VLAN etiketini basar ve gelen trafikte etiketi ayıklar. Fiziksel switch portunun **trunk** olarak yapılandırılması ve ilgili VLAN'lara izin vermesi gerekir.
* **4095:** Virtual Guest Tagging (VGT) — tüm VLAN etiketleri dokunulmadan sanal makineye kadar taşınır; etiketlemeyi guest OS yapar. Yalnızca sanal firewall/router gibi özel appliance senaryolarında kullanılır.

Aynı switch üzerinde farklı VLAN ID'lerle birden fazla port group tanımlayarak — örneğin `PG-Production-VLAN10` ve `PG-Test-VLAN20` — aynı iki uplink üzerinden birbirinden tamamen yalıtılmış ağlar taşıyabilirsiniz. Bir önceki mimari tartışmasında değindiğimiz "tek switch üzerinde VLAN tabanlı konsolidasyon" tam olarak bu mekanizmayla kurulur.

**Dikkat:** VST kullanacaksanız VLAN ID'yi girmeden önce fiziksel switch tarafının hazır olduğundan emin olun. Port group'ta VLAN 10 tanımlayıp fiziksel portta trunk/allowed VLAN yapılandırmasını atlamak, "her şey doğru görünüyor ama VM ağa çıkamıyor" şeklindeki klasik teşhis senaryosunun bir numaralı nedenidir.

#### 4. Security

Port group seviyesinde Promiscuous Mode, MAC Address Changes ve Forged Transmits politikalarını görürsünüz; varsayılan değer \*\*"Inherit from vSwitch"\*\*tir ve doğru pratik de budur:

* Politikaları **switch seviyesinde** merkezi olarak tanımlayın (üçü de Reject).
* Port group'larda inheritance'ı koruyun; böylece güvenlik duruşunuz tek noktadan yönetilir.
* Override'ı yalnızca gerçekten gerektiren port group'ta, gerekçesini belgeleyerek yapın (örneğin bir IDS sensörünün bağlandığı port group'ta Promiscuous Mode: Accept).

Bu yaklaşım, istisnaların zamanla unutulup genele yayılmasını engeller ve güvenlik denetimlerinde "hangi port group neden farklı" sorusuna her zaman cevabınız olur.

**Add** dedikten sonra switch topolojisinde yeni port group'u, VLAN ID'sini ve bağlı olduğu uplink'i birlikte görürsünüz. Yapı artık sanal makine kabul etmeye hazırdır.

### Sanal Makineleri Yeni Port Group'a Bağlamak

Port group oluşturulduktan sonra, host üzerindeki ağ seçeneklerine otomatik olarak eklenir ve iki senaryoda karşınıza çıkar:

#### Yeni VM oluştururken

VM oluşturma sihirbazının donanım özelleştirme adımında, network adapter için açılır listede artık tek seçenek olan `VM Network` yerine iki seçenek görürsünüz: `VM Network` (vSwitch0 üzerinden) ve yeni oluşturduğunuz port group (yeni switch üzerinden). Seçiminiz, makinenin hangi fiziksel yola ve hangi VLAN'a bağlanacağını belirler. Sihirbaz tamamlandığında switch topolojisinde VM'in yeni port group'a bağlandığını doğrulayabilirsiniz.

#### Mevcut bir VM'i taşırken

Var olan bir makinenin ağını değiştirmek için VM'in ayarlarından (**Edit Settings → Network Adapter**) port group seçimini güncellemeniz yeterlidir. Bu işlem VM çalışırken yapılabilir; vNIC anlık olarak yeni port group'a taşınır. Guest OS tarafında IP yapılandırmasının yeni ağa uygun olması gerektiğini unutmayın — port group değişikliği Layer 2 bağlantıyı taşır, IP adresini değiştirmez.

### Production İçin Ek Notlar

* **Port group bir politika sınırıdır:** Güvenlik, teaming/failover ve traffic shaping ayarlarının tümü port group seviyesinde override edilebilir. Bu esneklik, örneğin management trafiğini ayrı uplink'e sabitlerken VM trafiğini active/active bırakmak gibi ince tasarımların temelidir.
* **Kullanılmayan port group'ları temizleyin:** Hiçbir VM'in bağlı olmadığı, unutulmuş port group'lar hem envanteri kirletir hem de yanlış bağlantı riskini artırır.
* **Değişiklikleri belgeleyin:** Port group adı, VLAN ID, bağlı switch ve amaç bilgisini içeren basit bir ağ matrisi tutmak, hem yeni ekip üyelerinin adaptasyonunu hem de sorun teşhisini hızlandırır.

### Sonuç

Virtual Machine Port Group, sanal makineler ile fiziksel ağ arasındaki bağlantının tanımlandığı katmandır ve dört temel kararın kesişiminde durur: **doğru isim** (host'lar arası tutarlılık ve vMotion uyumluluğu), **doğru switch** (hangi uplink'ler, hangi fiziksel yol), **doğru VLAN ID** (mantıksal izolasyon) ve **doğru güvenlik duruşu** (switch'ten inheritance).

Bu dört karar baştan doğru verildiğinde, VM'leri ağlara bağlamak açılır listeden seçim yapmak kadar basitleşir; yanlış verildiğinde ise sonuçları — çalışmayan migration'lar, ağa çıkamayan makineler, denetlenemeyen güvenlik istisnaları — aylar sonra ortaya çıkar.

Switch'imizde artık sanal makineler için bir port group var; yapıyı tamamlamak için sıradaki adım, aynı switch üzerine host servislerini taşıyacak bir **VMkernel portu** eklemek olacak.

***

#### BONUS

**vmnic — fiziksel adaptör (host'un donanımı).** Sunucunun üzerindeki gerçek network kartının portudur; kablo buna takılır. ESXi bunları `vmnic0`, `vmnic1` diye numaralandırır. Makalelerde "uplink" dediğimiz şey budur — vSwitch'in fiziksel dünyaya çıkışı. `esxcli network nic list` çıktısında gördüklerin bunlardır.

**vNIC — sanal makinenin network kartı.** VM'in donanımının parçasıdır; guest OS'in içinden baktığında gördüğü Ethernet adaptörüdür (E1000, VMXNET3 gibi). Fiziksel bir karşılığı yoktur, tamamen yazılımsaldır. vNIC bir **port group'a** bağlanır — az önce yazdığımız makalede VM oluştururken açılır listeden port group seçmek, aslında VM'in vNIC'ini o port group'a takmaktı.

**vmk — VMkernel portu (host'un kendi sanal arayüzü).** ESXi'nin kendisinin ağa çıktığı arayüzdür (`vmk0`, `vmk1`...); management, vMotion, iSCSI trafiği buradan akar. Bir önceki sohbette `esxtop`'ta baktığımız `vmk0` buydu. Kabaca "host'un kendi vNIC'i" gibi düşünebilirsin.

**vSwitch (Standard Switch / vSS) — Sanal switch.** Host üzerinde çalışan, Layer 2 seviyesinde frame ileten yazılımsal switch'tir. İç tarafında port group'lar ve vmk portları, dış tarafında uplink'ler (vmnic) bulunur. Her host'ta bağımsız olarak yapılandırılır ve yönetilir; kurulumla birlikte gelen varsayılan switch `vSwitch0`'dır.

**vDS (vSphere Distributed Switch) — Dağıtık sanal switch.** Standard Switch'in vCenter üzerinden merkezi yönetilen versiyonudur: yapılandırma bir kez tanımlanır, cluster'daki tüm host'lara otomatik dağıtılır. LLDP, Network I/O Control, LBT (load-based teaming) ve port mirroring gibi vSS'te bulunmayan özellikler sunar; Enterprise Plus lisansı gerektirir.



