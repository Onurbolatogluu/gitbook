---
icon: teeth-open
---

# Maximum Number Of Virtual Machines Per Host

Sanal makinelerin donanım şablonlarını oluşturup içlerine işletim sistemlerini kurmaya başladığımızda, genellikle sanallaştırmanın o sınırsız gibi görünen büyüsüne kapılırız. İstediğimiz kadar makine oluşturabilir, her birine devasa RAM ve CPU'lar atayabiliriz. Ancak "Power On" tuşuna bastığımız anda, fiziksel donanımın acımasız matematiksel gerçekleriyle yüzleşiriz.

Bir ESXi sunucusunda aynı anda kaç sanal makine çalıştırılabileceği ve bu makinelerin hypervisor ile nasıl sağlıklı bir iletişim kuracağı, altyapı mimarisinin en temel iki direğidir.

#### 🛑 Oluşturmak vs. Çalıştırmak

Sanallaştırma dünyasında yeni başlayanların en sık düştüğü yanılgı, oluşturulan makine sayısı ile çalıştırılan makine sayısını birbirine karıştırmaktır.

Bir ESXi sunucusunda, diskiniz (Datastore) yettiği sürece yüzlerce sanal makine oluşturabilirsiniz. Makineler kapalı (Power Off) durumdayken sadece diskte yer kaplarlar; fiziksel işlemciden veya bellekten (RAM) hiçbir kaynak tüketmezler.

Ancak makineleri çalıştırmaya (Power On) başladığınızda fiziksel RAM ve CPU havuzunuzdan aktif olarak kaynak tüketimi başlar.

Örneğin; elinizde toplam 4 GB RAM'e sahip fiziksel bir ESXi host'u olduğunu düşünelim.

İçeriye 3 farklı sanal makine oluşturdunuz:

* VM-1: 4 GB RAM
* VM-2: 2 GB RAM
* VM-3: 2 GB RAM

Bu senaryoda matematiğin kuralları devreye girer:

Eğer VM-1'i başlatırsanız (4 GB), host'un tüm fiziksel RAM'i tükenir. Artık VM-2 veya VM-3'ü başlatamazsınız; sistem hata verir.

Eğer VM-2 ve VM-3'ü başlatırsanız (2 GB + 2 GB = 4 GB), yine limitlere ulaşırsınız ve bu kez VM-1'i başlatamazsınız.

> Özetle; bir host üzerinde aynı anda çalıştırabileceğiniz maksimum sanal makine sayısı, o makineler için talep edilen toplam RAM miktarının, ESXi sunucusundaki fiziksel RAM miktarını ne kadar doldurduğuyla doğrudan ilişkilidir.

#### 🛠️ VMware Tools

Sanal makinenize bir işletim sistemi (Windows, Linux vb.) kurduktan ve masaüstünü gördükten sonra atmanız gereken ilk ve en kritik adım, işletim sisteminin içine VMware Tools kurmaktır.

Çoğu kişi bunu sadece "ekran çözünürlüğünü düzelten basit bir sürücü paketi" olarak görse de, VMware Tools, Hypervisor (ESXi) ile sanal makine (Guest OS) arasındaki hayati iletişim köprüsüdür.

VMware Tools olmadan şunları yapamazsınız:

1. Güvenli Snapshot Alma: VMware Tools, makinenin yedeği (Snapshot) alınacağı milisaniyede işletim sistemine "Kısa bir süreliğine diske veri yazmayı durdur" (Quiescing) emri verir. Bu sayede veritabanları bozulmadan yedeklenir.
2. Graceful Shutdown: vSphere üzerinden makineye "Kapat" komutu gönderdiğinizde, makinenin fişini çekmek yerine işletim sistemine temiz bir kapanma sinyali gönderir. `Eğer sanal makinenizde VMware Tools kurulu değilse, vSphere arayüzündeki o alttaki "Shut Down Guest OS" butonu ya silik (pasif) görünür ya da bassanız bile hiçbir işe yaramaz. İşletim sistemi bu komutu duymaz.`
3. High Availability (HA) : Kritik bir özelliktir. ESXi, VMware Tools sayesinde sanal makinenin "yaşayıp yaşamadığını" (Heartbeat) saniye saniye izler. Eğer makine mavi ekran verirse, bunu Tools üzerinden anlar ve makineyi otomatik olarak yeniden başlatır.
4. Performans Sürücüleri: Sanal ağ kartları (VMXNET3) ve sanal disk kontrolcüleri (PVSCSI) gibi yüksek performanslı sanal donanımların sürücüleri sadece bu paketle gelir.

ESXi, bu kurulumu kolaylaştırmak için VMware Tools ISO dosyasını kendi içinde gömülü olarak barındırır. vSphere arayüzünden _Guest OS -> Install VMware Tools_ seçeneğine tıklandığında, bu paket sanal makinenin CD-ROM'una otomatik olarak takılır (Mount edilir) ve kurulum başlar. İşletim sisteminin sürümü (Windows 7, Server 2012 veya 2016) fark etmeksizin bu kural her zaman geçerlidir.

***

> 💡 Bellek Aşırı Tahsisi (Memory Overcommitment)
>
> "Fiziksel RAM dolduğunda yeni makine açılamaz" kuralı katı gibi görünse de, VMware ESXi kaputun altında Memory Ballooning ve Swapping adı verilen teknolojiler barındırır. Eğer VMware Tools kuruluysa; ESXi, çok fazla RAM kullanan ama aslında o RAM'i boşta tutan bir sanal makinenin belleğini "şişirerek" (Ballooning) geri alır ve o an acil ihtiyacı olan başka bir makineye verir. Bu sayede 16 GB fiziksel RAM'e sahip bir sunucuda, toplamı 20 GB RAM isteyen sanal makineleri (hafif performans kayıplarını göze alarak) aynı anda çalıştırabilirsiniz. Bu yeteneğin çalışmasının tek şartı, tüm makinelerde VMware Tools'un kurulu olmasıdır.
