---
icon: signal-bars
---

# ESXi Network management

VMware Workstation içinde sanal olarak (veya gerçek dünyada fiziksel bir sunucuda) ESXi kurulumunu tamamlayıp makineyi yeniden başlattığınızda, sizi siyah ve sarı renklerin hakim olduğu o meşhur DCUI (Direct Console User Interface) ekranı karşılar.

Bu ekran, sunucunun klavye ve monitörü ile doğrudan yapabileceğiniz son derece kısıtlı ama hayati ayarları barındırır. Asıl amacımız, bu sunucuyu ağa dahil edip, kendi bilgisayarımızdan rahatça yönetilebilir hale getirmektir.

#### 1. DCUI Ekranı

ESXi açıldığında karşınıza çıkan ekranda, sunucunuzun işlemcisi (Örn: Intel Core i5) ve RAM kapasitesi gibi donanım özetleri yazar. Eğer klavyeye bir süre dokunmazsanız, sarı/siyah ekran griye döner.

Bu bir hata değildir! Gerçek dünyada sunucular 7/24 açık olduğu için, VMware ekranın (monitörün) yanmasını (Screen Burn-In) engellemek amacıyla arayüzü karartır. Klavyeden bir tuşa basmanız ekranı uyandırmak için yeterlidir.

#### 2. Yönetim Paneline Giriş (F2 Tuşu)

Sistemi yapılandırmak için klavyeden F2 (Customize System/View Logs) tuşuna basmanız gerekir. Sistem sizden kullanıcı adı ve şifre isteyecektir.

* Varsayılan Kullanıcı: `root`
* Şifre: Kurulum aşamasında kendi belirlediğiniz şifre.

#### 3. En Kritik Adım: Configure Management Network

Şifreyi girip menüye ulaştığınızda, kurumsal bir ESXi sunucusu için atmanız gereken en önemli adım olan "Configure Management Network" bölümüne girersiniz. Burada şu kritik yapılandırmaları yapmalısınız:

**🌐 Network Adapters**

Gerçek bir sunucuda burada 4 veya 6 adet fiziksel ağ kartı (vmnic0, vmnic1 vb.) görebilirsiniz. Yönetim trafiğinin (sizin sunucuya bağlanacağınız trafiğin) hangi ağ kartı veya kartları üzerinden geçeceğini buradan seçersiniz. Lab ortamında tek bir ağ kartımız olduğu için sadece `vmnic0` görünür.

**⛔ IPv4 Configuration: Neden Statik IP Kullanmalıyız?**

Eğer ESXi sunucunuz dinamik IP alırsa ve modeminiz yeniden başlarsa, sunucunun IP adresi değişebilir. Bu durumda ona bağlı olan diğer araçlar (vCenter gibi) sunucuya ulaşamaz ve tüm mimariniz çöker.

Yapılması Gerekenler:

1. "Set static IPv4 address and network configuration" seçeneğini boşluk tuşuyla seçin.
2. Sunucunuza değişmeyecek bir IP atayın. Eğitimdeki örnekte; evdeki diğer cihazlarla (cep telefonu, TV) çakışmaması için `192.168.1.10` gibi rezerve edilmiş bir IP belirleniyor.
3. Subnet Mask ve modemin/router'ın IP adresi olan Default Gateway (Örn: 192.168.1.1) değerlerini girin.

**🗑️ IPv6 Kapatma**

Şirketinizde IPv6 kullanılmıyorsa, gereksiz yayınları (broadcast) azaltmak için IPv6'yı kapatabilirsiniz. Ancak unutmayın; IPv6'yı devre dışı bırakmak, ayarları kaydettikten sonra sunucunun tamamen yeniden başlatılmasını (Reboot) gerektirecektir.

**🏷️ DNS ve Hostname Ayarları**

Sadece IP adresleriyle uğraşmak zordur. Sunucunuza akılda kalıcı bir ad vermek için Hostname bölümünü `esxi-1` veya `host1` olarak değiştirebilirsiniz. Kurumsal yapılarda bu isimler, şirketin lokal DNS sunucularına (Örn: `esxi-1.sirketim.local`) kaydedilir.

#### 🎯 Sonuç: Kurulum Tamamlandı

Değişiklikleri kaydedip çıkmak için ESC tuşuna bastığınızda, sistem sizden onay ("Apply changes and reboot") ister. "Y" (Yes) diyerek onayladığınızda sunucu yeniden başlar.

Artık ESXi sunucunuzun siyah/sarı ekranıyla işiniz bitti!

Sunucu yeniden açıldığında ekranda yeni statik IP adresini (Örn: `192.168.1.10`) göreceksiniz. Bundan sonraki tüm yönetim işlemlerini, kendi laptop'ınızdaki web tarayıcısını (Chrome, Firefox) açıp bu IP adresini girerek, modern ve görsel vSphere Client üzerinden yapacaksınız.

