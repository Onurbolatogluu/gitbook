---
icon: wifi-fair
---

# Migrate VMs to Another Network

Önceki bölümde Distributed Switch'in fiziksel tarafını devreye almış, host'ları ekleyip uplink'leri eşlemiştik. Yapı hazır ancak eksik bir halka kalmıştı: sanal makinelerin tamamı hâlâ Standard Switch üzerindeki port group'lara bağlıydı. Bu makalede son adımı atıyoruz — VM ağ bağlantılarını yeni hedefe taşımak.

İki yöntem ele alacağız: tek bir makine için kullanılan doğrudan yöntem ve onlarca makineyi tek işlemde taşıyan toplu migrasyon aracı. İkisinin de ne zaman kullanılacağını, migrasyonun gerçekte neyi değiştirip neyi değiştirmediğini ve production'da nelere dikkat edilmesi gerektiğini inceleyeceğiz.

### Migrasyon Gerçekte Neyi Değiştirir?

Başlamadan önce işlemin kapsamını netleştirmek gerekir, çünkü buradaki yanlış anlama en sık yaşanan sorunun kaynağıdır.

VM'in ağını değiştirmek, makinenin **vNIC'inin hangi port group'a bağlı olduğunu** değiştirmekten ibarettir. Yani Layer 2 bağlantı noktası taşınır. Değişmeyenler:

* Guest OS içindeki **IP adresi, subnet mask ve gateway** — bunlar makinenin kendi yapılandırmasıdır, sanal ağ katmanı bunlara dokunmaz.
* vNIC'in **MAC adresi** — makine kimliğini korur.
* VM'in çalışma durumu — işlem VM açıkken yapılabilir ve makine yeniden başlatılmaz.

Pratik sonuç şudur: hedef port group farklı bir VLAN'a bağlıysa ve guest OS'in IP yapılandırması o VLAN'a uygun değilse, migrasyon "başarılı" görünür ama makine ağa çıkamaz. **Hedef port group'un VLAN'ı ile makinenin IP yapılandırması uyumlu olmalıdır.** Aynı VLAN'a bakan bir Standard Switch port group'undan distributed port group'a geçişte (vSS→vDS migrasyonunun tipik senaryosu) bu sorun yaşanmaz — sadece bağlantı noktası değişir, ağ aynı kalır.

### Yöntem 1: Tek VM'i Doğrudan Taşımak

Tek bir makineyi taşımak için VM'in kendi ayarlarını kullanırsınız:

1. VM'e sağ tıklayıp **Edit Settings** açın.
2. **Network Adapter** bölümünde açılır listeden hedef ağı seçin. Liste, host'ta erişilebilir tüm ağları gösterir: Standard Switch port group'ları (VM Network, önceki bölümlerde oluşturduğumuz port group'lar, private network) ve distributed port group'lar.
3. Distributed port group'lar listede ayrı bir kategori olarak görünür; hedef olarak istediğinizi seçin.
4. **OK** ile kaydedin.

İşlem anlıktır. VM artık Standard Switch yerine Distributed Switch üzerinden trafik taşır; vDS'in VM listesinde göründüğünü doğrulayabilirsiniz.

Bu yöntem tek makine, birkaç makine veya bir test/doğrulama adımı için idealdir. Ancak yirmi, elli, yüz makinelik bir ortamda her VM'i tek tek açıp değiştirmek ne pratiktir ne de güvenlidir — atlanan bir makine, ancak bir sorun çıktığında fark edilir.

### Yöntem 2: Toplu Migrasyon (Migrate VMs to Another Network)

vCenter, bu iş için özel bir araç sunar: kaynak ağdaki tüm (veya seçili) makineleri tek işlemde hedef ağa taşır.

vDS'e (veya doğrudan datacenter'a) sağ tıklayıp **Migrate VMs to Another Network** seçeneğiyle başlayın.

#### 1. Kaynak ve hedef ağ seçimi

Sihirbaz önce kaynak ve hedefi sorar:

* **Source network:** Makinelerin şu anda bağlı olduğu ağ. Belirli bir ağ seçebilir (VM Network, private network, başka bir distributed port group) veya **"No network"** seçeneğiyle hiçbir ağa bağlı olmayan vNIC'leri hedefleyebilirsiniz.
* **Destination network:** Makinelerin taşınacağı ağ.

Buradaki önemli nokta, aracın **tek yönlü olmadığıdır**. Standard Switch'ten vDS'e taşıma en yaygın kullanım olsa da, aynı araçla:

* distributed port group'tan Standard Switch port group'una **geri dönebilir** (rollback senaryosu),
* bir distributed port group'tan diğerine geçiş yapabilir,
* Standard Switch port group'ları arasında toplu taşıma yapabilirsiniz.

Bu esneklik, özellikle geçiş sırasında bir sorun çıktığında geri dönüş yolunuzun açık olması anlamına gelir — planlı bir migrasyonda bilinmesi gereken en değerli detaylardan biridir.

#### 2. Taşınacak makineleri seçme

Sihirbaz kaynak ağa bağlı tüm VM'leri, hangi host üzerinde çalıştıkları bilgisiyle birlikte listeler. Burada:

* **Tümünü seçebilir** veya yalnızca belirli makineleri işaretleyebilirsiniz.
* Liste, birden fazla host'a yayılmış makineleri bir arada gösterir; kaynak ağ birden çok host'ta tanımlıysa hepsi tek tabloda görünür.

Bu seçicilik, production geçişlerinde kritik bir imkândır: yüz makineyi bir anda taşımak yerine önce beş tanesini taşıyıp doğrulayabilir, sonra kalanlara devam edebilirsiniz.

#### 3. Gözden geçirme ve tamamlama

Özet ekranı kaynak ağı, hedef ağı ve taşınacak vNIC sayısını gösterir. **Finish** ile işlem tamamlanır ve seçili makinelerin tamamı yeni ağa geçer.

### Production Geçişi İçin Öneriler

vSS→vDS geçişini planlı bir operasyon olarak yürütmek, sonradan çıkacak sürprizleri önler:

#### Kademeli ilerleyin

Geçişi tek hamlede yapmayın:

1. Önce **kritik olmayan bir veya iki test VM'ini** hedef port group'a taşıyın.
2. Ağ erişimini doğrulayın: makineye ping atın, üzerinden dışarı çıkın, servisine erişin.
3. Doğrulama başarılıysa makine gruplarını partiler halinde taşıyın.
4. En son, en kritik iş yüklerini taşıyın.

#### Doğrulamayı taşımadan önce yapın

Hedef distributed port group'un yapılandırması, kaynak port group ile **eşdeğer** olmalıdır:

* Aynı **VLAN ID**
* Uyumlu **MTU**
* Uygun **teaming/failover** politikası
* Uplink'lerin bağlı olduğu fiziksel switch portlarının o VLAN'a izin vermesi

Bu kontroller yapılmadan taşınan makineler, migrasyon sonrası sessizce ağdan düşer. En sık atlanan kalem, fiziksel switch tarafındaki trunk/allowed VLAN yapılandırmasıdır — vDS'e yeni eşlediğiniz vmnic'ler farklı fiziksel portlara bağlıysa bu risk gerçektir.

#### Geri dönüş planınızı hazır tutun

Aynı araç ters yönde de çalıştığı için rollback yolunuz açıktır; ancak bunu geçiş öncesinde bilinçli olarak planlayın: kaynak port group'ları hemen silmeyin, geçiş doğrulanana kadar Standard Switch yapısını olduğu gibi bırakın.

#### Otomasyonu değerlendirin

Büyük ortamlarda PowerCLI ile toplu migrasyon hem daha hızlı hem de tekrarlanabilirdir:

```powershell
# Belirli bir port group'taki tüm VM'lerin ağını değiştir
$target = Get-VDPortgroup -Name "DPG-Production-VLAN10"
Get-VM | Get-NetworkAdapter |
  Where-Object { $_.NetworkName -eq "VM Network" } |
  Set-NetworkAdapter -Portgroup $target -Confirm:$false
```

Script'i önce `-WhatIf` ile çalıştırıp etkilenecek makineleri görmek, toplu işlemlerde standart bir güvenlik adımıdır.

### Migrasyonun Asıl Kazancı

Makineler distributed port group'a taşındığı anda vDS'in vaadi somutlaşır: **artık politikayı bir kez tanımlarsınız, tüm makinelere uygulanır.**

Port group üzerinde yaptığınız bir değişiklik — güvenlik politikası, VLAN, teaming/failover davranışı, traffic shaping — o port group'a bağlı **her host'taki her VM** için anında geçerli olur. Yirmi host'a yayılmış yüz makine için tek bir yapılandırma yeterlidir.

Standard Switch'te aynı sonuç için her host'a girip aynı değişikliği tek tek uygulamanız, sonra da hiçbirini atlamadığınızı doğrulamanız gerekirdi. vDS'e geçişin tüm gerekçesi bu farkta özetlenir: tutarlılık, elle korunan bir hedef olmaktan çıkıp mimarinin garantisi haline gelir.

Bunun ikinci bir sonucu daha vardır: VM'ler artık host'a özgü bir port group'a değil, tüm host'larda ortak olan bir tanıma bağlıdır. vMotion sırasında "hedef host'ta aynı isimde port group var mı?" sorusu tamamen ortadan kalkar; ağ tanımı makineyle birlikte taşınır.

### Sonuç

VM ağ migrasyonu, Standard Switch'ten Distributed Switch'e geçişin son ve en görünür adımıdır. Özetle:

* Migrasyon yalnızca **vNIC'in bağlı olduğu port group'u** değiştirir; IP, MAC ve çalışma durumu korunur. Hedef VLAN ile guest OS yapılandırmasının uyumu sizin sorumluluğunuzdadır.
* Tek makine için **Edit Settings** yeterlidir; ölçekli geçişler için **Migrate VMs to Another Network** aracı kullanılır.
* Araç çift yönlüdür: vSS→vDS, vDS→vSS ve port group'lar arası her yönde çalışır — bu, rollback planınızın temelidir.
* Geçişi kademeli yapın, hedef port group'un VLAN/MTU/teaming eşdeğerliğini önceden doğrulayın ve kaynak yapıyı doğrulama tamamlanana kadar silmeyin.

Bu adımla birlikte Distributed Switch tam anlamıyla devrede: host'lar bağlı, uplink'ler eşlenmiş, VM trafiği merkezi port group'lar üzerinden akıyor. Bundan sonrası, vDS'in sunduğu ileri yeteneklerin — teaming ve failover politikaları, Network I/O Control, port mirroring — bu merkezi yapı üzerinde ince ayarlanmasıdır.
