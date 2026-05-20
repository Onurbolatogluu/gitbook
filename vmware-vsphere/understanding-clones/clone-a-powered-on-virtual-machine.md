---
icon: book-copy
---

# Clone a Powered On Virtual Machine

#### ⚡ Canlı Sistemleri Kesintisiz Klonlama (Hot Clone) ve Snapshot Mimarisi

Sanallaştırma altyapılarında (özellikle 7/24 hizmet veren kurumsal ortamlarda) bir sunucuyu klonlamak için onu kapatmayı (Power Off) beklemek her zaman mümkün değildir. Kritik bir veritabanı sunucusunun veya aktif bir Domain Controller'ın trafiğini kesmek, ciddi operasyonel krizlere yol açabilir.

İşte bu noktada VMware mimarisinin en güçlü yeteneklerinden biri olan "Hot Clone" (Çalışır Durumda Klonlama) teknolojisi devreye girer. Peki, saniyede binlerce okuma/yazma (I/O) işlemi yapan, sürekli değişen canlı bir makine veri kaybı yaşanmadan nasıl klonlanabiliyor?

Bu makalede, çalışan bir sanal makinenin klonlanma sürecini, vCenter'ın arka planda uyguladığı "görünmez" mekanizmaları ve veri tutarlılığının (Data Consistency) nasıl sağlandığını mimari boyutta inceleyeceğiz.

**1. Çalışan Bir Sunucu Klonlanabilir mi? (Hot Clone Mantığı)**

Kısa cevap: Evet, hiçbir kesinti yaşatmadan klonlanabilir.

Klonlama sihirbazını başlatıp kaynak olarak halihazırda çalışan (Powered On) bir sanal makineyi seçtiğinizde, vCenter bu işleme itiraz etmez. İşlemi başlattığınızda, kaynak sunucuya bağlı olan son kullanıcılar (uygulamayı kullananlar, veritabanına veri girenler) arka planda devasa bir diskin kopyalandığını kesinlikle hissetmezler. Herhangi bir bağlantı kopması veya hizmet kesintisi (Downtime) yaşanmaz.

Ancak burada bilinmesi gereken ilk altın kural şudur: Kaynak makine açık (Powered On) olsa da, işlem bittiğinde ortaya çıkan yeni klon makine varsayılan olarak kapalı (Powered Off) durumda doğar. Bu, ağda anında bir IP veya Hostname çakışması yaşanmasını önleyen kritik bir güvenlik kalkanıdır.

**2. Point-in-Time Snapshot**

Sürekli verisi değişen bir diski "canlı canlı" kopyalamak fiziksel olarak imkansızdır. Kopyalamanın başı ile sonu arasında veriler değişeceği için dosya sistemi bozulur (Corruption). vCenter bu fiziksel engeli aşmak için klonlama sihirbazının sonundaki "Finish" (Bitir) butonuna bastığınız o milisaniyede gizli bir sihir yapar: Otomatik Snapshot (Anlık Görüntü) Alma.

Siz "Finish" dediğiniz an arka planda şunlar gerçekleşir:

1. vCenter, kaynak makinenin ana diski (Base VMDK) üzerindeki tüm yazma işlemlerini anında dondurur (Lock-Read Only).
2. Kaynak makine çalışmaya ve yeni veri kabul etmeye devam eder, ancak bu yeni veriler ana diske değil, geçici bir "Delta" (Fark) dosyasına yazılmaya başlar.
3. vCenter, kilitlenmiş ve artık değişmeyen o ana diski rahat rahat, güvenle hedef Host'a kopyalamaya başlar.

**3. Veri Tutarlılığı Çizgisi: Hangi Veriler Klonlanır, Hangileri Uçar?**

Sistem yöneticilerinin Hot Clone yaparken en çok dikkat etmesi gereken kavram Zaman Çizgisidir. Klonlanan yeni makinenin içinde hangi verilerin olacağını belirleyen tek şey, sizin "Finish" butonuna bastığınız o spesifik andır.

Bir örnekle açıklayalım:

* Saat 14:00'da sihirbazda Finish butonuna bastınız.
* 14:00'a kadar o sunucuda yapılmış olan her şey (eklenen kullanıcılar, veritabanı kayıtları, kurulan programlar) yeni klon makinede eksiksiz olarak var olacaktır.
* Klonlama işlemi büyük olduğu için diyelim ki saat 14:30'da bitti.
* 14:00 ile 14:30 arasında kaynak makineye giren kullanıcıların ürettiği hiçbir veri (Delta dosyasına yazıldıkları için) yeni klon makineye geçmeyecektir. Klon makine, orijinal makinenin tam olarak saat 14:00'daki dondurulmuş halinin bir kopyasıdır. İşlem bittikten sonra vCenter, orijinal makinedeki o geçici Delta dosyasını tekrar ana diske birleştirir (Consolidation) ve arkasında hiçbir iz bırakmadan orijinal makinenin hayatına devam etmesini sağlar.

**💡 Veritabanları ve VSS Entegrasyonu**

Çalışan bir makineyi klonlarken "Customization of the guest OS is not supported" gibi hatalar alıyorsanız (önceki yazılarda bahsettiğimiz gibi), bunun sebebi makinenin içinin boş olması veya VMware Tools'un kurulu olmamasıdır.

Canlı klonlamada VMware Tools'un önemi sadece Sysprep yapmakla sınırlı değildir. Eğer klonladığınız makine canlı bir SQL veya Exchange sunucusuysa, VMware Tools arka planda Microsoft'un VSS (Volume Shadow Copy Service) mimarisiyle konuşarak veritabanı transaction'larını klonlama (Snapshot) anında güvenli bir şekilde dondurur. Bu sayede klonlanan makine açıldığında veritabanı bozuk (corrupt) bir şekilde değil, tutarlı bir şekilde ayağa kalkar.

Özetle; vCenter'da "Hot Clone" operasyonu, arka plandaki geçici Snapshot mimarisi sayesinde Prod ortamlarına sıfır kesinti yansıtarak operasyonel esnekliği zirveye taşır. Ancak sistem yöneticisi, kopyalama sırasında üretilen verilerin (Delta verisi) yeni klona aktarılmayacağının bilincinde olmalı ve kritik veri taşıma işlemlerini buna göre planlamalıdır.

{% hint style="info" %}
Canlı klonlama (Hot Clone) işleminde vCenter'ın değişen veriyi korumak için "gizli bir Snapshot" aldığını belirtmiştik. Peki sunucu kapalıyken durum nedir?

Kapalı bir sunucuyu klonlarken vCenter asla Snapshot almaz. Çünkü işletim sistemi kapalıyken sanal disk (`.vmdk`) üzerinde hiçbir okuma/yazma (I/O) işlemi gerçekleşmez. Disk halihazırda statik ve veri bütünlüğü (consistency) açısından kusursuz bir durumda olduğu için "dondurma" işlemine fiziksel olarak ihtiyaç duyulmaz. vCenter, doğrudan Datastore üzerinden blok seviyesinde kopyalama (Block-level copy) yapar. Bu nedenle kapalı klonlama işlemleri, arka planda Snapshot oluşturma ve silme (Consolidation) adımlarını içermediğinden, canlı klonlamaya kıyasla çok daha hızlı, saf ve donanıma daha az yük bindiren bir süreçtir.
{% endhint %}
