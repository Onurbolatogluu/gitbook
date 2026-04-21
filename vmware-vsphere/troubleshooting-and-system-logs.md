---
icon: grid-4
---

# Troubleshooting and System Logs

Bir önceki adımda ESXi sunucumuzun ağ yapılandırmasını tamamlamıştık. Peki ya sunucumuz ağdan koparsa? Ya donanımsal bir hata alırsak veya VMware destek ekibi sistemimize uzaktan bağlanmak isterse?

Web arayüzüne (vSphere Client) erişemediğiniz o kriz anlarında, fiziksel sunucunun başına geçip (veya KVM üzerinden bağlanıp) F2 tuşuyla girdiğiniz DCUI (Direct Console User Interface) ekranı sizin tek kurtarıcınızdır.

#### 📡 1. Test Management Network

IP ayarlarını yaptınız ama sunucuya kendi laptop'ınızdan ulaşamıyorsunuz. Sorun nerede? Kabloda mı, switch'te mi, yoksa yanlış IP mi girdiniz?

İşte DCUI üzerindeki "Test Management Network" seçeneği, sunucunun bir nevi "Nabız Yoklaması"dır. Bu testi başlattığınızda ESXi, otomatik olarak 3 farklı adrese ping atar:

1. Default Gateway (Router/Switch): Sunucunun bulunduğu ağdan dışarı çıkıp çıkamadığını test eder. _(OK almalısınız)_
2. DNS Server: İsim çözümlemesinin çalışıp çalışmadığını kontrol eder.
3. Hostname Resolve: `esxi-1.home` gibi kendi isminin DNS üzerinde doğru çözümlenip çözümlenmediğine bakar.

> 💡 Eğer birinci ping başarısız oluyorsa, sunucunuz ıssız bir adadadır. Fiziksel kabloları, Uplink bağlantılarını veya bağlı olduğunuz fiziksel switch'in VLAN ayarlarını kontrol etmeniz gerekir.

#### 🔓 2. Troubleshooting Options: SSH ve ESXi Shell

Sistem yöneticilerinin en çok kullandığı, ancak güvenlik nedeniyle varsayılan olarak kapalı (Disabled) gelen menüdür. Burada iki kritik özellik bulunur:

* ESXi Shell: Sunucunun başına geçip klavyeden `Alt + F1` tuşlarına bastığınızda, o sarı/siyah ekranın arkasındaki gizli Linux benzeri komut satırına (CLI) inmenizi sağlar.
* SSH (Secure Shell): Kendi laptop'ınızdan bir terminal (Putty vb.) açarak sunucunun içine komut satırından bağlanmanızı sağlar.

Gerçek Dünya Senaryosu:

Sisteminizde çok garip bir "Purple Screen of Death (PSOD)" (Mavi ekranın VMware versiyonu) hatası aldınız ve çözemiyorsunuz. VMware Support ekibine bilet açtınız (Ticket). Mühendisler size şöyle der: _"Lütfen DCUI üzerinden SSH'ı Enable yapın ve bize IP verin."_

SSH'ı açtıktan sonra, kendi bilgisayarınızdan şu komutla sunucunun kalbine inebilirsiniz:

```bash
# ESXi sunucusuna SSH üzerinden root erişimi
ssh root@192.168.1.10

# İçeri girdikten sonra çalışan tüm sanal makineleri (VM) listelemek için:
esxcli vm process list
```

> ⚠️ Güvenlik Uyarısı: Videoda da özellikle belirtildiği gibi; SSH veya Shell sadece ihtiyaç anında açılmalı, iş bittiğinde derhal Disabled konuma getirilmelidir. Açık unutulan bir SSH portu, hackerlar için veri merkezinize giden otobandır.

#### 📜 3. View System Logs

Eğer sistemde "ters giden" bir şeyler varsa (Örn: bir sanal makine sürekli donuyorsa veya ağ kartı saniyeler içinde kopup geri geliyorsa), cevaplar her zaman loglardadır. DCUI üzerinden View System Logs menüsüne girip 1 ile 6 arasındaki tuşlara basarak kritik kayıtları okuyabilirsiniz:

* `[1] Syslog:` Sistemin genel günlük kayıtlarıdır. Hangi servisin ne zaman başladığını gösterir.
* `[2] VMkernel:` İşte en kritik log budur! Hypervisor'ün (ESXi çekirdeğinin) ta kendisidir. Donanım hataları, RAM arızaları veya disk (Storage) kopmaları burada yazar.
* `[3] Config:` Yapılandırma değişikliklerini gösterir.

#### 🔌 4. F12: Kapatma ve Yeniden Başlatma

Bir fiziksel ESXi sunucusunun güç (Power) tuşuna basılı tutarak kapatmak, bir sistem mühendisinin yapacağı en büyük hatadır. İçeride çalışan sanal makinelerin ( VM ) diskleri (vmdk) bozulabilir.

DCUI ekranında F12 tuşuna basıp root şifrenizi girdiğinizde, sistem size "Shut Down" veya "Restart" seçenekleri sunar.

