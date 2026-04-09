---
icon: h
---

# Hardware Virtualization

Fiziksel bir sunucuyu alıp sanal bir altyapıya taşıdığımızda kaputun altında tam olarak ne olur?

Geleneksel dünyada donanım (işlemci, bellek, ağ kartı) tektir ve içine kurulan tek bir işletim sistemine zimmetlenir. Ancak araya VMware vSphere gibi bir hypervisor girdiğinde, fiziksel donanım adeta sihirli bir kıyma makinesinden geçirilerek esnek, parçalanabilir ve yönetilebilir "sanal donanımlara" ( virtual hardware ) dönüşür.

Peki bu dönüşüm bize sahada nasıl avantajlar sağlar? Gelin işlemci, bellek ve ağ mimarisindeki bu devrime yakından bakalım.

#### 🧠 1. CPU Virtualization

Geleneksel mimaride 4 adet fiziksel CPU'nuz varsa ve sadece 1 CPU kullanabilen eski nesil bir uygulama çalıştırıyorsanız, geriye kalan 3 CPU boşta yatarak israf olur.

Sanal mimaride ise hypervisor, fiziksel işlemcileri bir havuz (pool) haline getirir. İşin en büyüleyici kısmı şudur: Elinizdeki fiziksel kapasiteden daha fazlasını sanal makinelere dağıtabilirsiniz. (Buna CPU Overcommitment diyoruz).

Örneğin 4 fiziksel işlemciniz varken; içeride 1, 2 ve 4 vCPU'ya sahip üç farklı virtual machine (VM) oluşturabilirsiniz (Toplam 7 vCPU). Sistem çökmez, çünkü vSphere mikrosaniyeler seviyesinde bir Time-Slicing yaparak, o an boşta duran VM'in işlemci hakkını anında ağır yük altında çalışan diğer VM'e kaydırır.

> ⚠️ Sınır: Tüm VM'ler aynı anda %100 yük altına girerse fiziksel sınırlar aşılır ve sistem işlem yapmak için sıraya girer. Bu darboğaza CPU Ready Time diyoruz.

#### 🎈 2. Memory Virtualization

İşlemcilerde Time-Slicing kolaydır ama Memory fiziksel bir alan gerektirir. Yine de sanallaştırma, elinizdeki 16 GB fiziksel RAM ile toplamda 24 GB RAM isteyen sanal makineleri aynı anda çalıştırabilir (Memory Overcommitment).

Fiziksel RAM dolmaya başladığında vSphere sistemi çökertmemek için zekice stratejiler uygular:

* TPS (Transparent Page Sharing): RAM'deki birbirinin aynısı olan işletim sistemi dosyalarını tekilleştirir.
* Ballooning: Fiziksel RAM bittiğinde, vSphere dışarıdan müdahale etmek yerine VM'in içindeki "Balloon Driver" ajanını şişirir. Sanal makine kendi RAM'inin dolduğunu sanarak panikler ve gereksiz verilerini kendi rızasıyla diske (Swap) yazar. Böylece temizlenen RAM bloğu, acil ihtiyacı olan diğer makineye aktarılır. Sistem çökmez, sadece kendini optimize eder.

#### 🌐 3. Network Virtualization

Geleneksel fiziksel dünyada iki sunucunun konuşması için ağ kartlarından fiziksel kabloların çıkıp fiziksel bir switch'e (ağ anahtarına) gitmesi gerekir. Sanallaştırma dünyasında ise iletişim mimarisi, sanal makinelerin ( VM ) fiziksel olarak nerede durduğuna göre iki farklı şekilde çalışır:

1\. Aynı Host Üzerindeki İletişim (İç Trafik) Aynı hypervisor üzerinde çalışan sanal makineler (örneğin biri Web Sunucusu, diğeri Veritabanı) kendi aralarında konuşurken, hypervisor kendi içinde sanal bir anahtar ( vSwitch ) yaratır. Trafik hiçbir zaman sunucunun dışındaki fiziksel kablolara veya cihazlara inmez. Bu sayede sunucular arası iletişim doğrudan donanımın bellek (RAM) hızında, yani adeta ışık hızında gerçekleşir. Donanım üzerindeki fiziksel ağ kartı, sadece internete çıkış için kullanılır.

2\. Farklı Host'lar Arasındaki İletişim (Dış Trafik) Eğer mimari büyür ve Web Sunucunuz Host-A'da, Veritabanı Sunucunuz Host-B'de çalışırsa (yani iki farklı hypervisor söz konusuysa), iletişim sanal dünyadan çıkıp fiziksel gerçeklikle yüzleşmek zorundadır. Bu senaryoda Host-A'nın vSwitch'i, veriyi sunucunun arkasındaki fiziksel ağ kartına (sektörel adıyla Uplink veya pNIC) gönderir. Veri fiziksel kablodan çıkar, veri merkezindeki gerçek fiziksel switch'e uğrar ve oradan yönlendirilerek Host-B'nin fiziksel ağ kartından içeri girer.
