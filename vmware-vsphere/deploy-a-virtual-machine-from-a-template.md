---
icon: fly
---

# Deploy a Virtual Machine from a Template

Önceki makalelerimizde, saatler süren sunucu kurulum süreçlerini dakikalara indiren "Altın İmaj" (Template) mimarisini kurmuş ve ideal bir sanal makineyi mühürleyerek şablona dönüştürmüştük. Şimdi sıra, depolama alanımızda bekleyen bu mühürlü kalıptan yepyeni, amaca özel ve bağımsız sanal makineler üretmeye (Deploy) geldi.

Bu makalede, bir şablondan (Template) yeni bir sunucu ayağa kaldırma sürecini, arayüzdeki kritik kırılımları ve vCenter'ın mimari koruma mekanizmalarını inceleyeceğiz.

**1. Şablon Menüsünün Doğası: "Power On" Neden Yok?**

vCenter arayüzünde (`VMs and Templates` görünümünde) bir şablona sağ tıkladığınızda, standart bir sanal makinede görmeye alışkın olduğunuz "Power On" (Çalıştır) veya "Guest OS" gibi seçeneklerin tamamen yok olduğunu fark edersiniz.

Bunun sebebi şablonların çalıştırılabilir bilgisayarlar değil, salt okunur (read-only) referans kalıpları olmasıdır. Bunun yerine vCenter size tamamen şablon yönetimine özel şu stratejik menüyü sunar:

* New VM from this Template: Bu mühürlü kalıbın bir kopyasını alarak yepyeni ve çalıştırılabilir bir sanal makine üretir. (En sık kullanılan operasyondur).
* Convert to Virtual Machine: Şablonu "çözer" ve tekrar normal bir sanal makineye dönüştürür. Dikkat: Bu işlem sonucunda elinizdeki o altın imaj (şablon) yok olur, geriye sadece çalıştırılabilir tek bir makine kalır. Şablon üzerinde kritik bir güncelleme (Örn: yeni bir Windows Update) yapmanız gerektiğinde kullanılır; güncellemeyi yapar, makineyi tekrar şablona çevirirsiniz.
* Clone to Template: Mevcut şablonun birebir kopyası olan ikinci bir şablon yaratır (Yedekleme veya farklı lokasyonlara taşıma amacıyla kullanılır).

**2. Şablondan Yeni Sunucu Üretmek (Deploy): İki Altın Kural**

`New VM from this Template` seçeneğiyle sihirbazı başlattığınızda, süreç klasik bir makine oluşturma adımlarına (İsim, Host, Datastore seçimi) benzer. Ancak işin kalitesi ve güvenliği, sihirbazın sunduğu şu iki kritik kutucuğun işaretlenmesine bağlıdır:

Kural 1: İşletim Sistemini Özelleştirmek (Customize Guest OS)

Bir şablondan yeni bir makine ürettiğinizde, bu yeni makine orijinal şablonun "kılcal damarlarına kadar" birebir kopyasıdır. Eğer bu adımı atlarsanız; yeni makine eski makineyle aynı IP adresine, aynı Bilgisayar Adına (Hostname) ve en tehlikelisi aynı SID'ye (Security Identifier) sahip olarak doğar. Ağa bağlandığı an devasa bir IP çakışması ve Active Directory krizi patlak verir.

Bu yüzden klonlama yaparken daima önceden hazırladığımız Customization Specification (Sysprep) şablonları devreye sokulmalı ve makinenin yeni, benzersiz bir kimlikle ağa çıkması sağlanmalıdır.

Kural 2: Donanımı Özelleştirmek (Customize Hardware)

Şablonlar genellikle "Baseline" (Taban) donanım özellikleriyle (Örn: 2 vCPU, 4GB RAM, 50GB Disk) oluşturulur. Ancak siz bu şablondan ağır bir Veritabanı (SQL) Sunucusu veya yoğun bir Mail Sunucusu ayağa kaldırıyor olabilirsiniz.

İşte klonlama sihirbazındaki `Customize Hardware` adımı, işletim sistemi kurulmadan önce makinenin kasasını büyütmenize (RAM'i 16GB yapmanıza, ekstra bir 500GB Veri diski eklemenize) olanak tanır. Makine ilk kez açıldığında, tam da o göreve uygun donanım gücüyle uyanır.

**3. Mimari Fark: "Convert to Template" vs. "Clone to Template"**

vCenter'da bir sanal makineden şablon elde etmenin iki farklı yolu vardır ve bu yolların kuralları, prod ortamının kesintisizliği açısından hayati önem taşır:

* Clone to Template (Şablona Klonla): Mevcut sanal makineye hiç dokunmaz, arka planda diskin kopyasını alarak yeni bir şablon yaratır. En büyük avantajı: Makine çalışır durumdayken (Powered On) yapılabilmesidir. Arka planda anlık bir Snapshot alınır ve kullanıcılara hiçbir kesinti yaşatılmadan şablon kopyası dışarı çıkarılır.
* Convert to Template (Şablona Dönüştür): Mevcut sanal makinenin kimliğini direkt olarak şablona çevirir. Kopya alma işlemi yoktur, saniyeler içinde gerçekleşir. Ancak vCenter'ın katı kuralı gereği: Bu işlem sadece makine KAPALIYKEN (Powered Off) yapılabilir.

💡 Ufuk Açıcı Katkı: vCenter Neden "Convert" İşleminde Makinenin Kapalı Olmasını İster?

Eğer vCenter, çalışan ve aktif olarak hizmet veren (örneğin üzerinde kullanıcıların işlem yaptığı bir veritabanı) bir sanal makineyi anında "Convert to Template" ile şablona çevirmenize izin verseydi, o makine saniyeler içinde "Çalıştırılamaz" (Salt Okunur / `.vmtx`) bir nesneye dönüşeceği için ağdan anında kopar, bellek (RAM) boşalır ve tüm kullanıcı verileri anında kaybolurdu. vCenter, yöneticinin yapabileceği bu tarz operasyonel hataları engellemek için, makine kapatılmadan "Dönüştürme" butonunu aktif etmez.

Özetle; Şablon mimarisi sadece kurulumları hızlandırmakla kalmaz; donanım modifikasyonu, kimlik yönetimi (Sysprep) ve OVF ile dışa aktarma (Export) yetenekleriyle birleştiğinde, sistem yöneticisine altyapıyı bir "kod gibi" esnek ve hatasız yönetme gücü verir.

