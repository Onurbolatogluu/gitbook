---
icon: bugs
---

# packer build -debug

#### 1. Step-by-Step

Normalde Packer, "Bas gaza" modunda çalışır; her şeyi olabildiğince hızlı ve aynı anda (Paralel) yapmaya çalışır. Ama `-debug` parametresini eklersen Packer "Manuel Vitese" geçer:

* Paralellik Biter: Her şey sıraya girer, teker teker yapılır.
* Onay Bekler: Packer her adımdan sonra durur (Pause) ve sana sorar: "Şimdi Provisioner çalıştıracağım, hazır mısın? Enter'a basarsan devam edeceğim."
* Faydası: O an hata oldu mu, dosya kopyalandı mı, sunucu ayakta mı? Hepsini adım adım kontrol edersin.

#### 2. Ephemeral SSH Key

Burası işin en havalı kısmı. Özellikle AWS gibi Bulut (Cloud) ortamlarında çalışırken en büyük sorun şudur: "Packer sunucuyu açtı ama ben içine giremiyorum, şifresini/anahtarını bilmiyorum."

`-debug` modunda Packer burada şöyle bir avantaj sağlar:

* Geçici Anahtar: Sunucuyu açtığı an, senin bilgisayarındaki klasöre geçici bir .pem dosyası bırakır.
* Terminalden `ssh -i anahtar.pem ...` yazarak o an çalışan sunucunun içine girebilirsin.
* Canlı Müdahale: İçeri girip "Neden Nginx kurulmadı?" diye dosyalara bakabilirsin.
* İş bitince Packer "Çıkıyoruz arkadaşlar" der ve o geçici anahtarı otomatik olarak siler.

#### 3. Windows ve RDP

Eğer Linux değil de Windows sunucu hazırlıyorsan (Windows Image), işler SSH ile değil RDP (Uzak Masaüstü) ve WinRM ile yürür. Packer bu modda, Windows'un admin şifresini çözer ve sana ekranda gösterir. Böylece Uzak Masaüstü ile bağlanıp sorunu görsel olarak inceleyebilirsin.

#### 4. Log vs Debug (Fark Nedir?)

Metin, önceki konuyla bu konuyu şöyle kıyaslıyor:

* `PACKER_LOG=1` (Pasif): Güvenlik kamerası kayıtlarını izlemek gibidir. Olay olup bittikten sonra "Ne olmuş?" diye bakarsın. Sadece izleyicisin.
* `-debug` (Aktif): Olay yerine gidip müdahale etmek gibidir. Süreci durdurabilir, içeri girebilir, gidişatı inceleyebilirsin.

***

Özetle: `packer build -debug` komutu, Packer'ın o hızlı ve kapalı kutu sürecini, senin kontrolünde ilerleyen bir Laboratuvar Ortamına çevirir. Özellikle "Kodum neden çalışmıyor?" dediğin anlarda, sunucunun içine girip bakman için sana anahtarı (SSH Key) teslim eden en güçlü araçtır.
