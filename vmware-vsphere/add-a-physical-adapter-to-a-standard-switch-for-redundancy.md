---
icon: square-minus
---

# Add a Physical Adapter to A Standard Switch for Redundancy

vSphere ortamında tek uplink'li bir Standard Switch, altyapınızdaki en kritik **single point of failure** noktalarından biridir. Tek fiziksel adaptöre bağlı bir switch'te; NIC'in kendisi, bağlı olduğu kablo, fiziksel switch portu veya upstream switch'in tamamı arızalandığında, o vSwitch üzerindeki tüm sanal makineler ve VMkernel servisleri ağdan kopar. Management trafiği de aynı switch üzerindeyse host'a uzaktan erişiminiz tamamen kesilir.

vSphere bu durumu açık bir uyarıyla bildirir: _"This virtual switch has no uplink redundancy."_ Bu makalede, bir Standard Switch'e ikinci bir physical adapter (uplink) ekleyerek bu riski nasıl ortadan kaldıracağınızı, işlemin arka planında failover ve load-balancing mekanizmalarının nasıl çalıştığını ve production ortamları için dikkat edilmesi gereken en iyi pratikleri ele alacağız.

### Uplink ve Port Group Kavramlarını Ayırt Etmek

İşleme başlamadan önce, bir Standard Switch topolojisinde göreceğiniz iki temel bileşeni netleştirelim:

* **Uplink (vmnic):** vSwitch'i fiziksel ağa bağlayan fiziksel network adaptörüdür. ESXi, fiziksel NIC'leri `vmnic0`, `vmnic1`, `vmnic2` şeklinde sıfırdan başlayarak numaralandırır. Tek uplink'li varsayılan kurulumda switch yalnızca `vmnic0`'a bağlıdır.
* **Port Group:** Switch'in "iç" tarafındaki mantıksal bağlantı noktalarıdır. Tipik bir varsayılan kurulumda iki port group görürsünüz: sanal makinelerin bağlandığı **VM Network** ve host yönetim trafiğini taşıyan VMkernel portunun bulunduğu **Management Network**.

Kısacası port group'lar trafiğin switch'e _girdiği_, uplink'ler ise switch'ten fiziksel dünyaya _çıktığı_ noktalardır. Redundancy ihtiyacı, bu çıkış noktasının tekilliğinden kaynaklanır.

### Adım Adım: İkinci Uplink Ekleme

İşlem, vSphere Client (veya doğrudan ESXi Host Client) üzerinden birkaç adımda tamamlanır ve **kesinti gerektirmez**; mevcut trafiği etkilemeden canlı ortamda uygulanabilir.

1. **Host → Configure → Networking → Virtual Switches** yoluyla ilgili Standard Switch'i (örneğin `vSwitch0`) seçin.
2. Switch topolojisinde mevcut durumda tek uplink (`vmnic0`) ve port group'ları göreceksiniz.
3. **Add Uplink** (veya **Manage Physical Adapters**) seçeneğine tıklayın. Sistem, host üzerinde henüz hiçbir switch'e atanmamış fiziksel adaptörleri otomatik olarak algılar ve listeler.
4. Kullanılabilir adaptörler listesinden eklemek istediğiniz NIC'i seçin. Örneğin `vmnic0` zaten kullanımda ise, listede `vmnic1` görürsünüz; host'unuzda dört fiziksel NIC varsa `vmnic1`, `vmnic2` ve `vmnic3` seçilebilir durumda olur.
5. **OK** ve ardından **Save** ile değişikliği uygulayın.

İşlem tamamlandığında switch topolojisini tekrar açtığınızda `vmnic0` ve `vmnic1`'in aynı switch'e bağlı olduğunu ve uplink redundancy uyarısının kaybolduğunu görürsünüz.

#### CLI Alternatifi

Aynı işlemi otomasyon veya toplu yapılandırma senaryolarında `esxcli` ile de gerçekleştirebilirsiniz:

```bash
# Kullanılabilir fiziksel NIC'leri ve link durumlarını listele
esxcli network nic list

# vmnic1'i vSwitch0'a uplink olarak ekle
esxcli network vswitch standard uplink add --uplink-name=vmnic1 --vswitch-name=vSwitch0

# Sonucu doğrula
esxcli network vswitch standard list
```

Çok sayıda host'u standartlaştırmanız gerekiyorsa bu komutları PowerCLI (`Add-VirtualSwitchPhysicalNetworkAdapter`) veya konfigürasyon yönetim araçlarıyla script'lemek, elle yapılan yapılandırma hatalarını ortadan kaldırır.

### Arka Planda Ne Olur: Load Balancing ve Failover

İkinci uplink eklendiği anda, switch'in **Teaming and Failover** politikası devreye girer ve trafik iki adaptör arasında dağıtılmaya başlar.

#### Trafik dağılımı gerçekte nasıl çalışır?

Kavramsal olarak trafiğin iki NIC arasında yarı yarıya, dört NIC arasında %25'er paylaşıldığını düşünebilirsiniz; toplam kapasite açısından bu doğru bir sezgidir. Ancak teknik olarak varsayılan politika (**Route Based on Originating Virtual Port ID**) trafiği paket paket bölmez:

* Her sanal makine (virtual port), switch'e bağlandığı anda uplink'lerden **birine** atanır ve tüm trafiği o uplink üzerinden akar.
* Dağılım, VM'lerin uplink'lere round-robin benzeri şekilde paylaştırılmasıyla sağlanır; yani denge **VM sayısı** üzerinden kurulur, anlık trafik hacmi üzerinden değil.
* Bu nedenle tek bir VM, tek bir uplink'in bant genişliğinden fazlasını hiçbir zaman kullanamaz. Gerçek yük bazlı dağıtım isteniyorsa Distributed Switch üzerindeki **Route Based on Physical NIC Load (LBT)** politikası gerekir.

#### Failover anında ne yaşanır?

Redundancy'nin asıl değeri arıza anında ortaya çıkar. `vmnic0`'a bağlı kablo çıkarsa, port kapanırsa veya adaptör arızalanırsa:

* Standard Switch, link durumundaki düşüşü (**Link Status**) anında algılar.
* Arızalı uplink'e atanmış tüm portlar **otomatik olarak** sağlıklı uplink'e (`vmnic1`) taşınır; manuel müdahale gerekmez.
* **Notify Switches: Yes** ayarı sayesinde vSwitch, fiziksel switch'lere RARP paketleri göndererek MAC tablolarını anında güncelletir ve kesintiyi milisaniyeler seviyesinde tutar.
* Arızalı adaptör tekrar ayağa kalktığında, **Failback** ayarına bağlı olarak trafik ya otomatik geri döner (Yes) ya da mevcut uplink'te kalmaya devam eder (No).

Aynı mantık üç veya dört uplink'li yapılarda da geçerlidir: herhangi bir adaptör düştüğünde onun taşıdığı portlar kalan sağlıklı adaptörlere yeniden dağıtılır.

### Production Ortamları için En İyi Pratikler

İkinci uplink'i eklemek uyarıyı kaldırır; ancak gerçek anlamda dayanıklı bir tasarım için şu noktaları da kapsamanız gerekir:

#### Fiziksel topolojiyi uçtan uca düşünün

* İki uplink'i **farklı fiziksel switch'lere** bağlayın. Her iki NIC aynı fiziksel switch'e gidiyorsa, o switch'in arızası redundancy'nizi anlamsız kılar.
* Mümkünse NIC'leri sunucudaki **farklı fiziksel kartlara** dağıtın (biri onboard, biri PCIe). Tek kart arızası her iki uplink'i birden düşürmemelidir.
* Kabloları farklı kablo kanallarından/patch panellerden geçirmek, fiziksel hasar senaryolarına karşı ek koruma sağlar.

#### Fiziksel switch portlarını eşitleyin

Yeni eklenen uplink'in bağlı olduğu fiziksel switch portu, mevcut portla **birebir aynı** yapılandırmaya sahip olmalıdır:

* Aynı VLAN'lar (trunk kullanılıyorsa aynı allowed VLAN listesi ve native VLAN)
* Aynı MTU değeri
* Aynı hız/duplex ayarları
* Spanning Tree tarafında **PortFast / Edge port** yapılandırması

Port yapılandırmaları farklıysa, her şey normal görünürken failover anında belirli VLAN'lardaki trafiğin kesildiği, teşhisi zor sorunlarla karşılaşırsınız. Bu, redundancy tasarımlarındaki en yaygın gizli hatadır.

#### Failover'ı gerçekten test edin

Eklenmiş ama hiç test edilmemiş bir yedek, yedek değildir. Kontrollü bir bakım penceresinde:

1. Aktif uplink'in kablosunu fiziki olarak çıkarın (veya fiziksel switch portunu kapatın).
2. VM'lere ve management IP'sine sürekli ping atarak kesinti süresini gözlemleyin; sağlıklı bir yapılandırmada kayıp birkaç paketi geçmemelidir.
3. `esxcli network nic list` ile link durumlarını, vSphere Client'ta switch topolojisini doğrulayın.
4. Kabloyu geri takıp failback davranışını da aynı şekilde test edin.

#### Trafik ayrıştırmasını değerlendirin

İki uplink'iniz varsa her ikisini de tüm trafik için active/active kullanmak zorunda değilsiniz. Port group seviyesinde failover order override ederek klasik çapraz tasarımı uygulayabilirsiniz:

* **Management Network:** `vmnic0` Active / `vmnic1` Standby
* **VM Network:** `vmnic1` Active / `vmnic0` Standby

Bu tasarımla normal işleyişte trafik türleri fiziksel olarak ayrışır, arıza anında ise her ikisi de karşı adaptöre taşınarak redundancy korunur. Management gibi düşük hacimli ama kritik trafiğin VM trafiğiyle bant genişliği yarışına girmemesi için tercih edilen bir desendir.

### Sonuç

Bir Standard Switch'e ikinci physical adapter eklemek, birkaç tıklamayla tamamlanan ancak ortamınızın erişilebilirliğini kökten değiştiren bir işlemdir. Özetle:

* Tek uplink'li her switch bir single point of failure'dır; uyarı mesajı görmezden gelinmemelidir.
* Uplink ekleme işlemi kesintisizdir; sistem boştaki `vmnic`'leri otomatik algılar ve ekleme sonrası trafik uplink'ler arasında dağıtılır, arıza anında sağlıklı adaptöre otomatik failover gerçekleşir.
* Gerçek redundancy yalnızca ESXi tarafında bitmez: farklı fiziksel switch'ler, eşitlenmiş port yapılandırmaları ve test edilmiş failover senaryoları tasarımın ayrılmaz parçalarıdır.
* Modern sunucular zaten iki ve üzeri NIC ile geldiğinden, bu koruma pratikte ek maliyet gerektirmez — gerektirdiği tek şey, kurulum aşamasında birkaç dakikalık doğru yapılandırmadır.

Uplink redundancy, bir sonraki adım olan teaming politikalarının ince ayarı ve trafik türlerinin izolasyonu için de zemin oluşturur; sağlam bir sanal ağ tasarımı, her katmanda "bu bileşen düşerse ne olur?" sorusunun cevaplanmış olmasıyla mümkündür.
