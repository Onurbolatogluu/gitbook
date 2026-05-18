---
icon: clone
---

# Cloning a Windows Virtual Machine: Destination Host

Bir sistem yöneticisinin mesaisinde en çok zaman kazandıran işlemlerin başında "Sanal Makine (Virtual Machine) Klonlama" gelir. Saatler sürecek işletim sistemi kurulumlarını, güncellemeleri ve uygulama konfigürasyonlarını dakikalara indiren bu teknoloji, vCenter'ın sunduğu en kritik "orkestrasyon" yeteneklerinden biridir.

Bu makalede, bir Windows sanal makinesinin klonlanma sürecindeki temel adımları, "Migrate" (Taşıma) ile "Clone" (Klonlama) arasındaki o ince çizgiyi ve hedef (Destination) belirlerken yapılan mühendislik tercihlerini inceleyeceğiz.

**1. Kesin Çizgi: Migrate vs. Clone (Taşıma mı, Kopyalama mı?)**

vCenter arayüzünde bir sanal makineye sağ tıkladığınızda karşınıza çıkan menüde hem `Clone` hem de `Migrate` seçenekleri bulunur. İşlevsel olarak birbirlerine benzeseler de, arka planda (Datastore ve Ağ mimarisinde) tamamen farklı işler yaparlar:

* Migrate: Bir sanal makineyi bulunduğu Host'tan (Donanımdan) veya Datastore'dan (Diskten) alıp başka bir yere taşır. Kaynak (orijinal) makine eski yerinden tamamen silinir. Ortamda o makineden hala sadece 1 tane vardır.
* Clone: Seçilen sanal makinenin disklerinin (VMDK), konfigürasyon dosyasının (VMX) ve içindeki tüm verilerin birebir kopyasını çıkararak yeni bir makine yaratır. İşlem bittiğinde ortamda o makineden tam 2 tane olur. Asıl makine çalışmaya devam ederken, kopyası yeni bir kimlikle (yeni MAC adresi) hayata başlar.

**2. Klonlama Operasyonu: 3 Farklı Strateji**

Clone seçeneğine tıkladığınızda vCenter size 3 farklı operasyon tipi sunar. Bunlar, klonun gelecekteki yaşam döngüsünü (Life Cycle) belirler:

1. Clone to a Virtual Machine (Sanal Makine Olarak Klonla): Yeni kopyanın anında çalıştırılabilir, aktif bir sunucu olarak ortama dahil edilmesidir.
2. Clone to a Template: Kopyanın çalıştırılabilir bir makine olarak değil, ileride yeni kurulumlar yapmak için kullanılacak salt okunur bir "Kalıp" (Template) olarak Datastore'a kaydedilmesidir.
3. Clone to a Template in Library: Oluşturulan o salt okunur kalıbın, diğer vCenter'ların (örneğin Ankara'daki merkezin) da erişebilmesi için ortak bir İçerik Kütüphanesine (Content Library) atılmasıdır.

**3. İsimlendirme Standartları ve İzolasyon Kuralı**

Sihirbazdaki ilk kritik adım, yeni klona bir isim vermektir.

Aynı isimde (örneğin `SQL_Sunucu`) bir klonu aynı Datacenter içindeki farklı bir Host'a kursanız bile vCenter buna teorik olarak izin verebilir. Ancak bu, saatli bir bombadır:

* Datastore Çakışması: ESXi, sanal makineyi kurduğu Datastore içerisinde makinenin adıyla bir klasör açar. Eğer ileride bir felaket (Disaster) yaşanır ve o iki makine aynı Host/Datastore üzerinde buluşmak zorunda kalırsa, klasör isimleri çakışacak ve makineniz açılamayacaktır (Orphaned durumuna düşecektir).
* Bu nedenle, örneğin `SQL_Server_03` makinesi klonlanıyorsa, yeni makinenin adı mutlaka benzersiz (Örn: `SQL_Server_04` veya `SQL_Server_Test`) olarak isimlendirilmelidir.

**4. Hedef Host'u (Donanımı) Seçmek ve Klonlamanın Amacı**

İsim verildikten sonra vCenter sizden "Bu klon hangi donanımın (Host) üzerinde yaşayacak?" sorusuna yanıt bekler. Bu seçim, klonlamayı neden yaptığınıza göre iki farklı senaryo yaratır:

* Senaryo A (Yedekleme/Backup Amacıyla Aynı Host): Makineyi sadece bir "yedek" veya "geriye dönüş noktası" olarak klonluyorsanız, kaynak makineyle aynı Host'u (Örn: Host-10) seçebilirsiniz.
* Senaryo B (Yeni Sunucu Amacıyla Farklı Host): Amacınız ortamdaki yükü dağıtmak ve yepyeni bir sunucu devreye almaksa, orijinal makine Host-10'daysa, klonu Host-20 üzerine kurmayı seçersiniz. Bu sayede işlemcisi ve RAM'i tamamen farklı bir donanım üzerinden hizmet vermeye başlar.

Bu seçim ekranında vCenter arka planda anlık bir Uyumluluk Kontrolü (Compatibility Check) yapar. Seçtiğiniz hedefin (Host'un) o makinenin donanım versiyonunu veya ağ mimarisini destekleyip desteklemediğini kontrol eder. Yeşil onay (Succeed) yazısını görmeden bir sonraki "Datastore (Depolama)" aşamasına geçilemez.
