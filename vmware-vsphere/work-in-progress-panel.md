---
icon: bar-progress
---

# Work in Progress Panel

vCenter Server arayüzü,  karmaşık iş akışlarını bölmeden yönetebilmeniz için hayat kurtaran, ufak ama mimari açıdan çok akıllıca tasarlanmış bir özelliğe sahiptir: Work in Progress (Devam Eden İşler) paneli.

Bu özellik, arayüzdeki işlemlerinizi bir tarayıcı sekmesi gibi askıya almanızı ve çoklu görev (multi-tasking) yapabilmenizi sağlar.

**🛑 Karşılaşılan Temel Sorun**

Devasa bir altyapıda yeni bir Virtual Machine oluşturduğunuzu hayal edin. Hedef Data Center'ı seçtiniz, Host atamasını yaptınız, depolama (Storage) ayarlarını ince ince yapılandırdınız ve donanım özelleştirme (Customize Hardware) ekranına kadar geldiniz. Bu işlem belki de 15-20 dakikanızı aldı.

Tam bu aşamada, bu makinenin bağlanması gereken özel bir Network Adapter (Ağ Bağdaştırıcısı) veya vSwitch yapısını oluşturmayı unuttuğunuzu fark ettiniz. Eski usul yönetim arayüzlerinde önünüzde tek bir seçenek vardır: Sihirbazı iptal etmek (Cancel), ağ ayarlarını yapmak üzere Host ekranına dönmek ve o 20 dakikalık Virtual Machine yapılandırmasına sıfırdan tekrar başlamak.

**💡 Çözüm: Sihirbazı Park Etmek (Work in Progress)**

vCenter, konfigürasyon ekranlarının sağ üst köşesine yerleştirdiği bir "Minimize" (Küçült) butonu ile bu sorunu çözer. Bu butona tıkladığınızda:

1. Durum Koruma (State Preservation): Yaptığınız o 20 dakikalık konfigürasyon silinmez. İşlem, arayüzün sağ tarafında bulunan "Work in Progress" paneline bir blok (task) olarak küçültülür.
2. Özgür Navigasyon: Ekranınız özgür kalır. Gidip ilgili Host üzerinde ağ ayarlarını yapabilir, yeni bir Port Group açabilir veya eksik bir yetkilendirmeyi (Permission) tanımlayabilirsiniz.
3. Kaldığın Yerden Devam Etme: Eksik işleminizi tamamladıktan sonra, sağ paneldeki bekleyen görevinize tıklarsınız. Sihirbaz tam olarak bıraktığınız ekranda, girdiğiniz tüm verilerle birlikte tekrar açılır.

**⚙️ Kaputun Altındaki Akıllı Mimari**

Bu özelliğin sadece bir "ekranı simge durumuna küçültme" işlemi olmadığını gösteren en önemli mühendislik detayı, dinamik veri yenileme yeteneğidir.

Siz Virtual Machine sihirbazını park edip, arka planda yeni bir Network Adapter oluşturduğunuzda, vCenter veritabanı (Inventory) anında güncellenir. Sihirbazı tekrar ekrana çağırdığınızda, sayfayı yenilemenize veya bir önceki adıma dönmenize gerek kalmaz; yeni oluşturduğunuz o ağ bileşeni, açılır menüde anında seçilebilir duruma gelmiştir.

**🔄 Paralel Görev Yönetimi (Multi-Tasking)**

Work in Progress paneli tek bir görevle sınırlı değildir. Aynı anda birden fazla operasyonu askıya alabilirsiniz:

* Bir yanda yeni bir Virtual Machine oluşturma sihirbazını park edebilir,
* Diğer yanda ortama yeni bir Host ekleme sürecini başlatıp onu da park edebilir,
* Gerektiğinde bu işlemler arasında geçiş yapabilir veya panel üzerinden doğrudan "Remove" diyerek iptal edebilirsiniz.

Özetle; vCenter arayüzündeki bu özellik, sistem yöneticilerine doğrusal (lineer) bir sırayla çalışmak yerine, esnek ve modüler bir operasyon imkanı sunar. Kurumsal mimarilerde zaman yönetimi ve insan hatasını (yapılandırmayı unutma vb.) en aza indirmek için tasarlanmış kritik bir iş akışı aracıdır.
