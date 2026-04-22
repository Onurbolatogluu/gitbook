---
icon: openid
---

# Thick And Thin-Provisioned Disks

Sanal makine (VM) oluşturmak, temelde yeni bir sanal işletim sistemine (Guest OS) kullanacağı donanımları tahsis etmektir. Bu tahsisi yaparken amaç her zaman ihtiyaç duyulan minimum kaynağı vererek donanımı en verimli şekilde kullanmaktır.

#### ⚙️ İşlemci (vCPU) ve Bellek (RAM) Tahsisi

* Compute (vCPU): Kural çok nettir: Sadece gerektiği kadar verin. Eğer bir sanal makineye ihtiyacından fazla vCPU verirseniz, makine işlem yapabilmek için fiziksel işlemcide o kadar çekirdeğin _aynı anda_ boşa çıkmasını beklemek zorunda kalır. Buna CPU Ready Time denir. Makineyi hızlandırayım derken, kuyrukta bekleterek yavaşlatmış olursunuz.
* Memory (RAM): RAM konusunda ise cimri olmamak gerekir. Örneğin bir Windows Server için 4 GB'ın altına inerseniz, işletim sistemi RAM yetmediği için verileri doğrudan diske (pagefile) yazmaya başlar. Diskler belleklerden çok daha yavaş olduğu için sistemde ciddi bir I/O darboğazı  oluşur.

***

#### 💾 Depolama (Storage): Disk Provisioning Stratejileri

Sanal bir disk (.vmdk) oluştururken, verinin fiziksel Datastore üzerine nasıl yerleşeceğini üç farklı yöntemle seçebilirsiniz:

1\. Thin Provisioning (Kullandıkça Büyüyen)

* Nasıl Çalışır? Sanal makineye 14 GB disk verseniz bile, disk tamamen boşsa Datastore üzerinde neredeyse hiç yer kaplamaz. Makine içine veri yazdıkça, fiziksel diskteki kapladığı alan dinamik olarak büyür.
* Avantajı: Atıl alanı israf etmez, ciddi yer tasarrufu sağlar. İçinde az veri olduğu için makineleri başka bir Datastore'a taşımak (Storage vMotion) çok hızlıdır.
* Dezavantajı: Fiziksel Datastore kapasitesi tamamen dolarsa, "Thin" olarak çalışan tüm sanal makineler anında donar (Pause durumu) ve sistem durur.&#x20;

{% hint style="info" %}
Thin Provisioning de tıpkı Lazy Zeroed gibi eski verilerin üzerine yazar. Ancak en büyük farkı; alanı baştan rezerve etmediği için, veri yazılacağı milisaniyede Datastore havuzundan rastgele bir blok çekmesidir.

Çekilen bu blokta eski/silinmiş makinelere ait "hayalet veriler" olabilir. Sistem, güvenlik açığı (Veri Sızıntısı) oluşmaması için bu bloğu makineye teslim etmeden önce mecburen sıfırlar (Zeroing), ardından yeni veriyi yazar. Bu "Alan Bul + Sıfırla + Yaz" mesaisi, yüksek veri trafiğinde anlık performans kayıplarına sebep olur.
{% endhint %}

2\. Thick Provision Lazy Zeroed (Rezerve Edilmiş, İhtiyaç Anında Temizlik)

* Nasıl Çalışır? 14 GB'lık alan fiziksel olarak anında adınıza rezerve edilir ve başkası kullanamaz. Ancak bu fiziksel blokların üzerindeki eski veri kalıntıları anında temizlenmez.
* Sonuç: Disk hızlıca oluşturulur. Ancak sanal makine bir bloğa _ilk kez_ veri yazacağı zaman, anlık olarak "önce o bloğu sıfırla, sonra yeni veriyi yaz" döngüsüne girer. Bu durum, ilk yazma anlarında hafif bir I/O gecikmesine neden olur.

{% hint style="info" %}
vCenter üzerinden eski bir sanal makineye "Delete from Disk" dediğinizde, sistem o devasa veriyi fiziksel olarak silmekle vakit kaybetmez; sadece o verinin adres defterindeki (indeks) kaydını siler. Fiziksel Datastore üzerinde eski veriler (binary kodlar) "hayalet" gibi kalmaya devam eder.

Eğer bu alanın üzerine Thick Provision Lazy Zeroed bir disk oluşturursanız, alan anında tahsis edilir ancak bu hayalet veriler temizlenmez. Yeni sanal makine bu alana ilk kez veri yazmak istediğinde ise donanım mecburen "Önce eskiyi temizle (Zeroing), sonra yeniyi yaz" şeklinde çift mesai yapar. İşte ilk yazma anında yaşanan bu anlık I/O performans düşüşüne sektörde Write Penalty denir.
{% endhint %}

3\. Thick Provision Eager Zeroed (Rezerve Edilmiş, Peşin Temizlik)

* Nasıl Çalışır? 14 GB'lık alan hem anında rezerve edilir hem de diskin tamamına "Sıfır" (Zeroing) yazılarak baştan sona tertemiz bir blok yaratılır.
* Sonuç: Temizlik peşin yapıldığı için diski oluşturmak saatler sürebilir. Ancak temizlenmiş bir diske sonradan veri yazarken hiçbir bekleme yaşanmaz. SQL Veritabanları gibi maksimum okuma/yazma (IOPS) performansı gerektiren sistemler için zorunlu standarttır.

***

#### 🌐 Diğer Kritik Ayarlar ve VMware Tools

* Spesifik bir lisanslama veya güvenlik gereksinimi yoksa, ağ kartının MAC adresi genellikle otomatik (DHCP veya vCenter havuzundan) bırakılır.
* Kurulması kritik öneme sahiptir. Hypervisor ile sanal işletim sisteminin birbiriyle pürüzsüz iletişim kurmasını sağlar. Sistem saatlerinin senkronize edilmesi ve fiziksel sunucu yeniden başlatılırken içerdeki VM'lerin fişinin çekilmek yerine güvenlice kapatılabilmesi (Graceful Shutdown) bu araç sayesinde mümkün olur.

