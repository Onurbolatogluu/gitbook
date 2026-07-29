---
icon: italic
---

# Types of Virtual Standard Switch Connections

vSphere ortamında sanal ağ tasarımı, yalnızca bir switch oluşturup sanal makineleri bağlamaktan ibaret değildir. Bir Standard Switch'in hangi bağlantı tiplerini desteklediğini ve trafiğin tek bir switch üzerinde mi yoksa birden fazla switch'e ayrılarak mı taşınacağını bilinçli olarak tasarlamak, altyapının hem performansını hem güvenliğini hem de yönetilebilirliğini doğrudan belirler.

Bu makalede önce bir virtual switch'in desteklediği iki temel bağlantı tipini netleştirecek, ardından "tek switch üzerinde konsolidasyon" ile "ayrı switch'lerle fiziksel izolasyon" mimarilerini karşılaştırarak hangi senaryoda hangisinin tercih edilmesi gerektiğini en iyi pratiklerle birlikte ele alacağız.

### Virtual Switch Üzerindeki İki Bağlantı Tipi

Bir Standard Switch, dış tarafta **uplink**'ler (fiziksel network adaptörleri, `vmnic`) aracılığıyla fiziksel ağa bağlanır. İç tarafta ise iki farklı bağlantı tipine hizmet verir:

#### 1. Virtual Machine Port Group

Sanal makinelerin ağa bağlandığı mantıksal port topluluklarıdır. Tipik bir kurumsal ortamda trafik türlerine veya güvenlik bölgelerine göre ayrı port group'lar tanımlanır:

* **Production:** Canlı iş yükü trafiği
* **Test/Dev:** Geliştirme ve test ortamları
* **DMZ:** İnternete açık, izole edilmesi gereken sunucular

Her port group kendi VLAN etiketini, güvenlik politikalarını ve teaming ayarlarını taşıyabilir. Böylece aynı fiziksel altyapı üzerinde birbirinden mantıksal olarak ayrılmış ağlar işletilebilir.

#### 2. VMkernel Port

Sanal makinelerin değil, **ESXi host'un kendi trafiğinin** çıkış noktasıdır. Her hypervisor servisi bir VMkernel portu üzerinden ağa erişir:

* **Management:** Host yönetim trafiği (her host'ta en az bir tane zorunludur)
* **vMotion:** Sanal makinelerin canlı taşınması
* **Storage:** iSCSI ve NFS gibi IP tabanlı depolama trafiği
* **Fault Tolerance Logging:** FT korumalı VM'lerin senkronizasyonu
* **vSAN / Replication:** Dağıtık depolama ve replikasyon trafiği

Tek bir switch üzerinde hem VM port group'ları hem de birden fazla VMkernel portu bir arada bulunabilir. Örneğin dört uplink'li bir switch'te Production, Test/Dev ve DMZ port group'ları ile Management ve vMotion VMkernel portları aynı anda barınabilir. Tasarım sorusu tam da burada başlar: her şey aynı switch'te mi olmalı?

### Temel Kısıt: Bir vmnic, Yalnızca Bir vSwitch'e Bağlanabilir

Mimari kararınızı şekillendiren en önemli kural şudur: **bir fiziksel network adaptörü aynı anda yalnızca tek bir virtual switch'e uplink olabilir.** Bunun doğrudan sonuçları:

* İki fiziksel NIC'iniz varsa en fazla iki Standard Switch oluşturabilirsiniz (her birine birer uplink düşer) — ancak bu durumda hiçbir switch'te redundancy kalmaz.
* Redundancy korunarak çoklu switch mimarisi kurmak istiyorsanız, **her switch için en az iki NIC** gerekir; dört switch'li bir tasarım sekiz fiziksel adaptör anlamına gelir.
* Buna karşılık port group ve VMkernel portu sayısında böyle bir fiziksel kısıt yoktur; tek switch üzerinde onlarcası tanımlanabilir.

Bu kısıt, "host'ta ne kadar çok fiziksel adaptör varsa altyapı o kadar esnek ve profesyonel tasarlanabilir" ilkesinin teknik gerekçesidir. NIC sayınız, mimari seçeneklerinizin üst sınırını belirler.

### İki Mimari Yaklaşım

#### Yaklaşım 1: Tek Switch Üzerinde Konsolidasyon

Host üzerindeki tüm fiziksel adaptörler tek bir Standard Switch'e (genellikle varsayılan `vSwitch0`) uplink olarak bağlanır; tüm port group'lar ve VMkernel portları bu switch üzerinde tanımlanır. Trafik türleri **VLAN'lar ile mantıksal olarak** ayrıştırılır.

**Avantajları:**

* Tüm uplink'ler tek havuzda toplandığı için **maksimum redundancy** sağlanır: dört uplink'li tek switch'te üç adaptör arızalansa bile bağlantı sürer.
* Bant genişliği verimli kullanılır; boşta bekleyen adaptör kalmaz.
* Yönetim basittir: tek switch, tek teaming politikası seti.
* Az sayıda NIC ile (iki adet dahi) tam işlevsel ve yedekli bir ortam kurulabilir.

**Dezavantajları:**

* Trafik izolasyonu tamamen VLAN yapılandırmasına emanettir; fiziksel ayrım yoktur.
* Yoğun bir trafik türü (örneğin vMotion veya storage), doğru şekilde sınırlandırılmazsa diğer trafikleri etkileyebilir.
* Fiziksel switch tarafında trunk port yapılandırması zorunludur.

#### Yaklaşım 2: Ayrı Switch'lerle Fiziksel İzolasyon

Her trafik türü için ayrı bir Standard Switch oluşturulur ve her birine kendi uplink çifti atanır. Örnek bir tasarım:

* **vSwitch0:** Management (2 uplink)
* **vSwitch1:** vMotion (2 uplink)
* **vSwitch2:** Production VM trafiği (2 uplink)
* **vSwitch3:** Test/Dev (2 uplink)
* **vSwitch4:** iSCSI storage (2 uplink)

**Avantajları:**

* Trafik türleri **fiziksel olarak** izole edilir; bir switch'teki yoğunluk veya yanlış yapılandırma diğerlerini etkileyemez.
* Güvenlik sınırları nettir: DMZ trafiği ile management trafiği aynı kabloyu asla paylaşmaz.
* Sorun teşhisi kolaylaşır; hangi trafiğin hangi fiziksel yoldan aktığı bellidir.
* iSCSI gibi hassas storage trafiği için özel MTU (Jumbo Frames) ve yapılandırma, diğer trafiklere dokunmadan uygulanabilir.

**Dezavantajları:**

* Yüksek NIC maliyeti: yukarıdaki beş switch'li tasarım, redundancy ile birlikte **on fiziksel adaptör** gerektirir.
* Bant genişliği havuzlanamaz; bir switch'in uplink'leri boştayken diğeri doygunluğa ulaşabilir.
* Yönetilecek yapılandırma sayısı artar; host'lar arası tutarlılığı korumak zorlaşır.

### Hangi Yaklaşımı Seçmelisiniz?

Doğru cevap ortamınızın donanım profiline bağlıdır ve sektördeki eğilim son yıllarda netleşmiştir:

* **1GbE çağının çok portlu sunucularında** (6-8+ NIC), ayrı switch'lerle fiziksel izolasyon standarttı; düşük bant genişliği, trafik türlerinin ayrı yollara dağıtılmasını zorunlu kılıyordu.
* **Modern 10/25GbE ortamlarında** sunucular genellikle iki adet yüksek kapasiteli port ile gelir. Bu ortamlarda tercih, **tek switch üzerinde VLAN tabanlı konsolidasyon**dur; izolasyon fiziksel kablolarla değil VLAN'lar ve trafik şekillendirme mekanizmalarıyla sağlanır. Distributed Switch kullanılabilen ortamlarda **Network I/O Control (NIOC)**, konsolide uplink'ler üzerinde trafik türlerine bant genişliği garantisi tanımlayarak fiziksel izolasyonun performans avantajını büyük ölçüde telafi eder.
* **Hibrit tasarım** da meşru bir orta yoldur: iSCSI storage trafiğini kendi switch'ine ve adaptörlerine ayırıp (iSCSI port binding gereksinimleri için de temiz bir zemin sağlar), geri kalan her şeyi konsolide etmek sık kullanılan bir desendir.

Hangi yaklaşımı seçerseniz seçin, iki kural değişmez:

1. **Hiçbir switch tek uplink ile bırakılmaz.** Fiziksel izolasyon uğruna redundancy'den vazgeçmek, kabul edilebilir bir takas değildir.
2. **Tüm host'larda aynı mimari uygulanır.** vMotion ve HA'nın sorunsuz çalışması, host'lar arasında ağ yapılandırmasının (switch, port group adları, VLAN'lar) birebir tutarlı olmasına bağlıdır. Standard Switch her host'ta ayrı ayrı yapılandırıldığı için bu tutarlılığı elle korumak zordur; host sayısı arttıkça yapılandırmayı merkezi yöneten Distributed Switch'e geçiş bu nedenle değerlendirilmelidir.

### Sonuç

Sanal ağ mimarisinde doğru karar, iki temel bilginin üzerine kurulur: bir virtual switch'in **VM port group** ve **VMkernel port** olmak üzere iki bağlantı tipine hizmet ettiği ve bir fiziksel adaptörün **yalnızca tek bir switch'e** bağlanabildiği. Bu iki gerçek bir araya geldiğinde, host'unuzdaki NIC sayısı mimari özgürlüğünüzün sınırını çizer.

Özetle:

* Trafik türlerini ayrıştırmak bir tercih değil zorunluluktur; asıl karar bu ayrımın **VLAN ile mantıksal** mı, **ayrı switch'lerle fiziksel** mi yapılacağıdır.
* Az sayıda yüksek kapasiteli NIC'e sahip modern ortamlarda konsolidasyon + VLAN, çok portlu eski nesil ortamlarda veya sıkı izolasyon gereksinimlerinde çoklu switch yaklaşımı öne çıkar.
* Her koşulda redundancy ve host'lar arası tutarlılık korunmalıdır.

Sunucularınıza donanım aşamasında yeterli sayıda fiziksel adaptör koymak, ileride mimariyi yeniden tasarlamak zorunda kalmamanın en ucuz sigortasıdır; sanal ağ tasarımında esneklik, kablonun takıldığı gün kazanılır.
