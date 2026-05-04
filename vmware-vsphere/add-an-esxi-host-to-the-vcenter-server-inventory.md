---
icon: arrow-turn-down-left
---

# Add an ESXi Host to the vCenter Server Inventory

Sanallaştırma ortamımızı kurarken, başlangıçta her bir ESXi sunucusuna (host) kendi IP adresi üzerinden, ayrı ayrı kullanıcı adı ve şifrelerle bağlanıp işlem yaparız. Ancak ortam büyüdükçe bu yönetim şekli sürdürülemez hale gelir. İşte bu noktada, tüm sunucuları ve sanal makineleri tek bir merkezden (vCenter) yönetme ihtiyacı doğar.

**1. Veri Merkezinin Temelini Atmak (Datacenter Oluşturma)**

vCenter'ı kurup arayüzüne giriş yaptığınızda, karşınıza tamamen boş bir envanter çıkar. Bu aşamada yapacağınız ilk iş doğrudan bir sunucu eklemek değildir; çünkü vCenter, sunucuları havada asılı bırakmaz. Önce o sunucuların barınacağı "Mantıksal Veri Merkezini" (Datacenter) oluşturmanız gerekir.

Datacenter Nedir?

vCenter içerisindeki Datacenter nesnesi, fiziksel dünyadaki bir binayı, şehri veya şirketin belirli bir iş birimini temsil eden en üst düzey kapsayıcıdır. Örneğin; coğrafi konum bazlı "Morocco", "Washington DC" gibi isimler verebileceğiniz gibi, işlev bazlı "Veritabanı Sunucuları", "Test Ortamı" gibi isimler de verebilirsiniz.

Kritik Kural: vCenter'da bir Datacenter objesi oluşturulmadan, ortama hiçbir ESXi host eklenemez.

**2. ESXi Sunucusunu (Host) vCenter'a Bağlama Süreci**

Datacenter oluşturulduktan sonra, dağınık haldeki ESXi sunucularımızı bu yapıya bağlamaya başlarız. Bu süreç şu adımlarla ilerler:

1. Kimlik Tanımlama: Host'un IP adresi (veya DNS adı) girilir.
2. Yetkilendirme: Host'un _root_ kullanıcı adı ve şifresi girilir. Bu işlem sadece bir kez yapılır; vCenter bu bilgileri güvenli bir şekilde kaydeder ve bir daha size sormaz.
3. Güvenlik ve Sertifika (Sertifika Uyarısı): Bağlantı sırasında vCenter genellikle bir "Sertifika doğrulanamadı" uyarısı verir. Bu, ESXi sunucularının varsayılan olarak kendi ürettikleri (self-signed) sertifikaları kullanmalarından kaynaklanır. Bu uyarı kabul edilerek geçilir ve vCenter o sunucuyu kendi güvenli iletişim ağına dahil eder.
4. Envanter Özeti: Bağlantı sağlandığında vCenter size o host'un üzerinde çalışan sanal makineleri, ESXi sürümünü (Örn: 6.5) ve donanım üreticisini bir özet ekranında gösterir. Bu sayede doğru sunucuyu ekleyip eklemediğinizi teyit edebilirsiniz.

**3. Güvenlik Çemberini Daraltmak: Lockdown Mode (Kilitleme Modu)**

Sunucu ekleme sihirbazının en can alıcı noktalarından biri Lockdown Mode seçeneğidir. Bu özellik, kurumsal güvenlik politikalarının temel taşlarından biridir.

<figure><img src="../.gitbook/assets/Screenshot 2026-05-04 at 11.30.48.png" alt=""><figcaption></figcaption></figure>

Bir ESXi sunucusu vCenter'a eklendiğinde, teorik olarak o sunucuyu hem vCenter üzerinden hem de doğrudan IP adresine giderek eski usul yönetebilirsiniz. Ancak güvenlik açısından bu durum büyük bir zafiyet yaratır.

* Lockdown Mode Devredeyken: Eğer bu modu aktif ederseniz, vCenter o ESXi sunucusunun etrafına bir "duvar" örer. Artık kimse o sunucuya doğrudan IP adresi üzerinden veya vSphere Client ile bağlanamaz. Yönetim işlemleri sadece ve sadece vCenter üzerinden veya sunucunun yanına fiziksel olarak gidip konsol (DCUI) üzerinden yapılabilir.
*   Normal vs. Strict (Sıkı) Mod: Strict mod seçildiğinde, sunucunun yanına fiziksel olarak gitseniz dahi konsol arayüzü kapatılır. Yönetim tek bir noktaya (vCenter'a) hapsedilir.

    _(Not: Lab ve test ortamlarında sorun giderme esnekliği için bu mod genellikle kapalı tutulur.)_

Özetle; Bu işlem tamamlandığında, "Standalone" (Bağımsız) olarak hayatına devam eden sunucularınız, vCenter'ın "Datacenter" şemsiyesi altında birleşir. Artık bu sunucular tek bir ekrandan izlenebilir, aralarında kaynak paylaşımı yapılabilir ve sanal makineler bu sunucular arasında kesintisiz olarak taşınabilir hale gelir.
