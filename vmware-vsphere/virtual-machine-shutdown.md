---
icon: power-off
---

# Virtual Machine Shutdown

Bir veri merkezinde güç kesintisi yaşandığını, jeneratörlerin devreye giremediğini ve altyapınızı besleyen UPS (Kesintisiz Güç Kaynağı) bataryalarının size sadece 30 dakikalık bir yaşam alanı sunduğunu hayal edin. Bu kriz anında, veri kaybını önlemek için ESXi Host sunucunuza "Shut Down" (Kapat) emri verdiniz. Peki ama o an içeride aktif olarak çalışan, veritabanlarına veri yazan sanal makinelere ne olacak?

Eğer sanal makine kapanış (Shutdown) mimarisini önceden doğru yapılandırmadıysanız, fiziksel sunucuyu kapatma emriniz, sanal makineleriniz için tam bir felakete dönüşebilir. Bir önceki yazımızda sistemin nasıl "sırayla" ayağa kalktığını (Startup) incelemiştik. Bu makalede ise felaket anlarında sistemin kendini nasıl güvenle kapattığını ve "VM Shutdown" mimarisindeki altın kuralları inceleyeceğiz.

ESXi üzerinde sanal makine açılış/kapanış ayarlarını (VM Startup/Shutdown) ilk aktif ettiğinizde, sistemin varsayılan kapanış eylemi (Shutdown Action) "Power Off" olarak gelir.

Bu, sanallaştırma dünyasındaki en tehlikeli eylemlerden biridir. Power Off eylemi, fiziksel bir bilgisayarın fişini duvardan aniden çekmekle birebir aynı mantıkta çalışır. İşletim sistemine önbellekteki (cache) verileri diske yazması veya açık olan servisleri durdurması için hiçbir şans verilmez. Özellikle SQL, Oracle veya Exchange gibi yoğun I/O (Okuma/Yazma) yapan sunucularda bu durum; veritabanı bozulmalarına (Database Corruption) ve kritik veri kayıplarına yol açar.

**Doğru Mimari: "Guest Shutdown"**

Kriz anında veri bütünlüğünü sağlamak için yapılması gereken ilk ve en önemli mimari dokunuş, menüdeki _Shutdown Action_ seçeneğini "Guest Shutdown" olarak değiştirmektir.

Siz Host'a kapatma emri verdiğinizde, Host bu eylemi sanal makinelere "Guest Shutdown" olarak iletir. Bu komut, makinenin içindeki VMware Tools servisi aracılığıyla Windows veya Linux işletim sistemine _Start > Shut down_ (Başlat > Kapat) komutu olarak gider. İşletim sistemi servisleri güvenle durdurur, verileri diske yazar ve kendini nezaketle kapatır.

**Zaman Yönetimi: "Shutdown Delay"**

Sanal makinelere "Güvenli Kapanış" (Guest Shutdown) komutu gönderildiğinde, bu işlemin tamamlanması zaman alır. Bir Windows sunucunun güncellemeleri yapılandırıp tamamen kapanması bazen dakikalar sürebilir.

İşte bu noktada Shutdown Delay (Örneğin: 120 Saniye) ayarı devreye girer. Bu ayar sisteme şu emri verir: _"Bu makineye kapanış sinyalini gönder, kapanması için ona 120 saniye müddet tanı. Sonra sıradaki makineye geç."_ Eğer bu süreyi çok kısa tutarsanız (Örn: 10 saniye), makine daha işletim sistemini kapatmaya fırsat bulamadan ESXi Host sıradaki işlemlere geçeceği için süreç yine sağlıksız sonlanacaktır.

**Kapanış Sıralamasının Gizli Mantığı: LIFO**&#x20;

Mimari tasarımda sistem yöneticilerinin sıklıkla düştüğü bir yanılgı vardır: _Sistem nasıl 1-2-3 sırasıyla açılıyorsa, kapanırken de 1-2-3 sırasıyla kapanır zannedilir._ Ancak işin mühendislik gerçeği tam tersidir.

Daha önceki "Startup" (Açılış) makalemizde bağımlılık zincirinden bahsetmiştik: Önce Active Directory (DC) açılır, sonra Veritabanı (DB) açılır, en son Uygulama (App) açılır.

Kapanış senaryosunda ESXi müthiş bir zeka sergiler ve sistemi otomatik olarak tersten (Reverse Order) kapatmaya başlar:

1. Önce Uygulama sunucusunu kapatır (Böylece veritabanına gelen yeni kayıt akışı durur).
2. Sonra Veritabanı sunucusunu kapatır (Bağlantı koptuğu için güvenle kapanır).
3. En son Active Directory sunucusunu kapatır (Böylece diğer makineler kapanana kadar kimlik doğrulama mekanizması ayakta kalır).

Bu LIFO (Last-In, First-Out) mimarisi sayesinde, açılış (Startup) için yaptığınız o kritik sıralama, kapanış (Shutdown) anında sistem tarafından otomatik olarak tersine çevrilerek veri bütünlüğünüz korunmuş olur.

Kapanış senaryosunda dikkat edilmesi gereken bir diğer hayati konu, makinelerin hangi havuzda (Listede) bulunduğudur:

* Any Order (Sırasız Havuz): Bu havuzdaki makinelerin birbiriyle bağımlılığı yoktur. ESXi Host, yukarıdaki kritik makineleri tersten sırayla kapattıktan sonra bu havuza geçer ve buradaki makineleri de güvenli bir şekilde kapatır.
* Manual Startup: Dikkat! Kriz anlarında en büyük kayıplar burada yaşanır. Eğer çalışan (Power On) bir sanal makineniz "Manual Startup" havuzunda duruyorsa, ESXi Host kapanma sürecine girdiğinde bu makineye "Guest Shutdown" komutu GÖNDERMEZ. Çünkü o makinenin yönetimi tamamen sizin inisiyatifinize bırakılmıştır. Host tamamen kapanırken, bu havuzda çalışan makine acımasızca "Power Off" (Fiş Çekme) durumuna maruz kalır.

Özetle; Kurumsal bir yapıda, "Manual Startup" havuzunda çalışan (Aktif) hiçbir makine bırakılmamalıdır. Aktif makineleriniz mutlaka "Automatic" veya "Any Order" havuzlarında bulunmalı ve kapanış eylemleri muhakkak "Guest Shutdown" olarak ayarlanmalıdır. Bu basit görünen ama kaputun altında devasa bir mühendislik barındıran ayarlar, uykusuz geçecek bir Disaster Recovery gecesi ile sorunsuz bir mesai sabahı arasındaki o ince çizgiyi belirler.

