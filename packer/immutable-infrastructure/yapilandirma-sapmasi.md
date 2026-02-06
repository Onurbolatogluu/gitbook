---
icon: square-sliders-vertical
---

# Yapılandırma Sapması

Geleneksel (Mutable) altyapı yönetiminde, sunucular yayına alındıktan sonra zamanla başlangıç durumlarından uzaklaşır.

* Sistem yöneticilerinin SSH üzerinden yaptığı anlık müdahaleler, manuel paket güncellemeleri ve belgelenmemiş ayar değişiklikleri.
* Sonuç: Teorik olarak özdeş olması gereken sunucular, pratik olarak birbirinden farklı hale gelir. Bu durum, güvenlik açıklarına ve kök neden tespiti zor olan operasyonel hatalara yol açar.

#### 2. Çözüm Stratejisi

* Baking Yöntemi: Uygulama kodu, sistem kütüphaneleri ve konfigürasyon dosyaları, sunucu henüz başlatılmadan önce Makine İmajı içerisine gömülür.
* Üretilen imaj, Değişmez (Immutable) bir yapıttır. Bu imajdan başlatılan 1. sunucu ile 1000. sunucunun, bit seviyesinde aynı durumda olduğu matematiksel olarak garantidir.
* Sunucu yaşam döngüsü boyunca manuel değişiklik yapılmaması beklenir. Değişiklik ihtiyacı doğduğunda sunucu güncellenmemesi önerilir, yenisiyle değiştirilmesi gerekir.

#### 3. Ortamlar Arası Eşitlik

Yapılandırma sapması sadece sunucular arasında değil, geliştirme (Dev) ve üretim (Prod) ortamları arasında da yaşanır.

* Packer, aynı şablon dosyasını kullanarak farklı platformlar (AWS, VMware, Docker) için teknik olarak eşdeğer imajlar üretir.
* Bu yaklaşım, yazılımın geliştiricinin yerel makinesinde çalışıp üretim ortamında hata vermesi sendromunu ortadan kaldırır.

***

Özetle: Yapılandırma Sapmasını Önleme; Packer aracılığıyla sunucuların dinamik ve belirsiz yapılar olmaktan çıkarılıp, sürümlenebilir, test edilebilir ve öngörülebilir Artifacts'lere dönüştürülmesi sürecidir. Bu, operasyonel kaosu önleyen en güçlü savunma mekanizmasıdır.
