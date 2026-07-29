---
icon: mask-ventilator
---

# Create and Manage A New Standard switch From ESXi

Her ESXi host, kurulumla birlikte varsayılan bir Standard Switch (`vSwitch0`) ile gelir ve birçok ortamda tüm trafik bu tek switch üzerinden yönetilir. Ancak trafik türlerini fiziksel olarak izole etmek, storage ağını ayırmak ya da host içinde kapalı bir test ağı kurmak istediğinizde ikinci, üçüncü hatta daha fazla Standard Switch oluşturmanız gerekir.

Bu makalede ESXi üzerinde sıfırdan yeni bir Standard Switch oluşturma sürecini adım adım ele alacak; uplink'siz bir switch'in ne anlama geldiğini, "boşta fiziksel adaptör yok" durumuyla karşılaştığınızda ne yapmanız gerektiğini ve uplink'lerin switch'ler arasında nasıl taşındığını production perspektifiyle inceleyeceğiz.

### Yeni Bir Standard Switch Oluşturmak

ESXi Host Client üzerinde **Networking → Virtual Switches → Add standard virtual switch** yolunu izleyerek yeni switch oluşturma ekranına ulaşırsınız. Burada karşınıza çıkan temel alanlar şunlardır:

* **Name:** Switch'in adı (örneğin `vSwitch1`). Anlamlı bir isimlendirme standardı kullanmak — `vSwitch-iSCSI`, `vSwitch-vMotion` gibi — kalabalık ortamlarda yapılandırmanın okunabilirliğini ciddi biçimde artırır.
* **MTU:** Varsayılan 1500 byte'tır. Switch'i Jumbo Frames gerektiren bir amaç (iSCSI, vSAN) için oluşturmuyorsanız bu değeri değiştirmeyin; değiştirecekseniz uçtan uca tüm fiziksel bileşenlerin aynı değeri desteklediğinden emin olun.
* **Link Discovery:** CDP davranışı — varsayılan ayarlar çoğu ortam için uygundur.
* **Security:** Promiscuous Mode, MAC Address Changes ve Forged Transmits politikaları. Yeni bir switch oluştururken bu üçünü **Reject** olarak ayarlamak, güvenli varsayılan (secure by default) prensibinin gereğidir.

**Add** butonuna tıkladığınızda switch oluşturulur — ancak kritik bir detayla: eğer host üzerinde boşta fiziksel adaptör yoksa (veya uplink seçmediyseniz), yeni switch **sıfır uplink ve sıfır port group** ile doğar. Bu durumun ne anlama geldiğini doğru anlamak, sanal ağ tasarımının en öğretici noktalarından biridir.

### Uplink'siz Switch: Host İçinde İzole Bir Ağ

Fiziksel adaptörü olmayan bir switch'e port group ekleyip sanal makineler bağladığınızda, bu makineler **yalnızca birbirleriyle** haberleşebilir. Fiziksel ağa çıkış noktası bulunmadığı için:

* Ortamdaki hiçbir kullanıcı veya sistem bu makinelere erişemez.
* Bu makineler de host dışındaki hiçbir kaynağa (DNS, gateway, internet, diğer sunucular) ulaşamaz.
* Trafik, host'un belleği içinde kalır; fiziksel kabloya hiç dokunmaz.

İlk bakışta bir eksiklik gibi görünen bu davranış, aslında bilinçli kullanıldığında değerli bir tasarım aracıdır:

* **İzole test/lab ortamları:** Production'a hiçbir şekilde dokunmaması gereken deneme makineleri, zararlı yazılım analizi veya yama testleri için doğal bir sandbox sağlar.
* **Air-gapped iş yükleri:** Ağa çıkması istenmeyen hassas sistemler host içinde tamamen kapalı tutulabilir.
* **Sanal appliance zincirleri:** Klasik bir desen olarak, çift bacaklı bir firewall/router VM'in bir vNIC'i uplink'li switch'e, diğeri uplink'siz "internal" switch'e bağlanır; iç switch'teki tüm makineler dış dünyaya yalnızca bu appliance üzerinden, denetlenerek çıkar.
* **Performans avantajı:** Aynı iç switch'teki VM'ler arası trafik hypervisor içinde memory hızında akar; fiziksel ağ bant genişliği tüketmez.

Ancak kural nettir: gerçek ortamda sanal makinelerin kullanıcılara hizmet verebilmesi için switch'in **en az bir uplink'e** sahip olması gerekir. İzolasyon bilinçli bir tercih değilse, uplink'siz switch bir yapılandırma eksiğidir.

### "No Free Physical Adapters" Durumu

Yeni switch'inize uplink eklemeye çalıştığınızda şu mesajla karşılaşabilirsiniz:

> _"There are no free physical adapters to attach to this virtual switch."_

Bu mesajın nedeni, önceki bölümlerde ele aldığımız temel kısıttır: **bir fiziksel adaptör aynı anda yalnızca tek bir virtual switch'e bağlanabilir.** Host'unuzdaki tüm `vmnic`'ler halihazırda başka switch'lere atanmışsa, yeni switch'e verilecek boşta adaptör kalmamıştır. Bu durumda üç seçeneğiniz vardır:

1. **Yeni fiziksel adaptör eklemek** — production ortamındaki doğru çözüm budur. Modern sunucularda genellikle boş PCIe slotu bulunur ve ek NIC maliyeti, mimari esnekliğin bedeli olarak düşüktür.
2. **Mevcut bir switch'ten uplink taşımak** — mevcut donanımla yeniden düzenleme.
3. **Switch'i bilinçli olarak uplink'siz (internal-only) bırakmak** — yukarıda anlatılan izolasyon senaryoları için.

### Uplink'i Bir Switch'ten Diğerine Taşımak

Boşta adaptör yoksa, mevcut bir switch'ten uplink söküp yeni switch'e atayabilirsiniz. İşlem iki adımdır:

1. Kaynak switch'te (örneğin `vSwitch0`) ilgili uplink'i seçip **Remove** ile kaldırın ve kaydedin. Adaptör artık "free" durumuna geçer.
2. Hedef switch'te **Add Uplink** dediğinizde serbest kalan adaptör listede görünür; seçip kaydedin.

Bu noktada, oluşturma sihirbazının kullanışlı bir davranışını da bilmekte fayda var: **host üzerinde boşta fiziksel adaptör varken yeni bir switch oluşturursanız, sistem bu adaptörü otomatik algılar ve uplink olarak önerir.** Birden fazla boş adaptör varsa hepsi listelenir ve istediğinizi seçersiniz. Yani doğru sıralama — önce adaptörü serbest bırakmak, sonra switch'i oluşturmak — süreci tek hamlede tamamlamanızı sağlar.

#### Production uyarısı: Uplink taşırken neyi feda ettiğinizi bilin

Lab ortamında zararsız olan bu işlem, canlı sistemlerde iki ciddi risk taşır:

* **Redundancy kaybı:** İki uplink'li `vSwitch0`'dan bir adaptörü söktüğünüzde, o switch tek uplink'e düşer ve bir önceki makalede ayrıntılı incelediğimiz _"no uplink redundancy"_ durumuna geri dönersiniz. Yeni switch'e kavuşurken mevcut switch'i single point of failure haline getirmiş olursunuz.
* **Erişim kaybı riski:** Management VMkernel portunun trafiğini taşıyan uplink'i yanlışlıkla kaldırırsanız host'a uzaktan erişiminizi kaybedebilirsiniz. Bu tür değişikliklerden önce hangi trafiğin hangi uplink'ten aktığını doğrulayın; ESXi'nin **DCUI** üzerindeki _Network Restore Options_ özelliğinin acil durumda kurtarma yolu olduğunu bilin ve mümkünse değişiklikleri konsol erişiminizin (iLO/iDRAC/IPMI) hazır olduğu bir bakım penceresinde yapın.

Genel prensip şudur: **çoklu switch mimarisi, uplink'leri bölüştürerek değil, yeterli sayıda fiziksel adaptörle kurulur.** İki NIC'li bir host'ta iki switch'e birer uplink dağıtmak, her iki switch'i de yedeksiz bırakır; böyle bir durumda tek switch üzerinde VLAN tabanlı konsolidasyon neredeyse her zaman daha doğru tercihtir.

### CLI ile Aynı İşlemler

Otomasyon ve toplu yapılandırma için tüm bu adımların `esxcli` karşılıkları mevcuttur:

```bash
# Yeni bir standard switch oluştur
esxcli network vswitch standard add --vswitch-name=vSwitch1

# Boşta olan fiziksel NIC'leri görüntüle
esxcli network nic list

# Uplink'i bir switch'ten kaldır
esxcli network vswitch standard uplink remove --uplink-name=vmnic1 --vswitch-name=vSwitch0

# Uplink'i yeni switch'e ekle
esxcli network vswitch standard uplink add --uplink-name=vmnic1 --vswitch-name=vSwitch1

# Sonucu doğrula
esxcli network vswitch standard list
```

Aynı mimariyi birden fazla host'ta kurmanız gerekiyorsa, bu komutları veya PowerCLI karşılıklarını (`New-VirtualSwitch`, `Add-VirtualSwitchPhysicalNetworkAdapter`) script'lemek hem zamandan kazandırır hem de host'lar arası tutarlılığı garanti eder — ki vMotion ve HA'nın sağlıklı çalışması bu tutarlılığa bağlıdır.

### Sonuç

Yeni bir Standard Switch oluşturmak teknik olarak birkaç tıklamalık bir işlemdir; asıl mühendislik, bu switch'in uplink stratejisini doğru kurgulamaktadır. Özetle:

* Yeni oluşturulan switch, uplink atanmadıysa **host içinde izole bir ağ** olarak doğar; bu, dış erişim gerektirmeyen lab, sandbox ve appliance senaryoları için bilinçli bir tasarım aracı, production VM'leri içinse bir yapılandırma eksiğidir.
* _"No free physical adapters"_ mesajı bir hata değil, "bir vmnic yalnızca bir switch'e bağlanır" kuralının doğal sonucudur; çözüm önceliğiniz uplink taşımak değil, yeterli fiziksel adaptör sağlamak olmalıdır.
* Uplink taşıma işlemi production'da redundancy ve management erişimi risklerini beraberinde getirir; her zaman mevcut trafiğin hangi yoldan aktığını doğrulayarak ve konsol erişimi hazırken uygulanmalıdır.
* Güvenlik politikalarını (Reject) ve isimlendirme standartlarını switch'i oluştururken baştan doğru ayarlamak, sonradan düzeltme maliyetini ortadan kaldırır.

Bu aşamada elimizde uplink'i bağlı ancak henüz port group'u olmayan bir switch var; bir sonraki adım, bu switch üzerine sanal makineler için port group'lar ve host servisleri için VMkernel portları tanımlayarak yapıyı kullanılabilir hale getirmektir.
