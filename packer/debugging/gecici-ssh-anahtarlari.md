---
icon: key-skeleton
---

# Geçici SSH Anahtarları

#### 1. Nedir Bu "Geçici" Anahtar?

Packer, AWS veya Azure üzerinde bir sunucu (Instance) oluştururken, o sunucuya bağlanıp komut çalıştırmak zorundadır. Bunun için anlık olarak bir şifreleme anahtarı üretir.

* Normalde: Packer bu anahtarı kendi hafızasında tutar, işini yapar ve kimseye vermez.
* Debug Modunda: Eğer `-debug` komutunu kullandıysan, Packer sana şöyle der: _"Al kardeşim, bu sunucunun anahtarı. Bilgisayarındaki klasöre .pem dosyası olarak bıraktım. Lazım olursa kullanırsın."_
* Windows İçin: Eğer Windows sunucu kuruyorsan, bu anahtar WinRM şifresini çözmek için kullanılır.

#### 2. Ne İşe Yarar?

Diyelim ki Packer çalıştı, sunucuyu açtı ama Provisioner aşamasında (yazılım yüklerken) hata verdi. Normalde sunucu kapanır ve sen hatanın sebebini asla öğrenemezsin.

Bu anahtar sayesinde şunları yapabilirsin:

* Canlı Bağlantı: Terminalini açıp `ssh -i anahtar.pem kullanıcı@ip-adresi` yazarak, o an Packer'ın oluşturduğu sunucunun içine girersin.
* Olay Yeri İncelemesi: "Neden dosya yüklenmedi?", "Disk mi doldu?", "İnternet mi yok?" gibi soruların cevabını sunucunun içinde gezinerek bulursun.
* Cloud-Init Kontrolü: Önceki konularda bahsettiğimiz "Sunucu tam açılmadan işlem yapma" hatasını, içeriden `cloud-init status` yazarak gözünle teyit edebilirsin.

#### 3. Yaşam Döngüsü:&#x20;

Bu anahtarların en önemli özelliği kendini imha etmesidir.

* Oluşturulma: Packer çalışmaya başladığında oluşturulur.
* Kullanım: Sen hata ayıklarken veya Packer işlem yaparken kullanılır.
* İmha: Packer işlemi bittiğinde (başarılı olsun veya olmasın), Packer arkasını temizler ve o `.pem` dosyasını siler. Yani o anahtarı saklayıp "Yarın da kullanırım" diyemezsin. Bu bir güvenlik önlemidir.

#### 4. Dikkat Edilmesi Gereken Bir "Ağ" Detayı

* Senaryo: Packer AWS'de bir sunucu açtı ve sana anahtarı (.pem) verdi. Sen bağlanmaya çalışıyorsun ama bağlanamıyorsun.
* Sebep: Eğer Packer'a sunucuyu "Dış dünyaya kapalı, güvenli bir ağda (Private Subnet)" açmasını söylediysen, elinde anahtar olsa bile o sunucuya internet üzerinden erişemezsin.
* Çözüm: Bu durumda araya bir atlama tahtası (Bastion Host / RD Gateway) koyman veya VPN kullanman gerekir. Yani anahtarın olması, kapıya ulaşabileceğin anlamına gelmez.

#### 5. Neden Önemli?

Normalde Immutable Infrastructure mantığında sunuculara SSH ile girip elle ayar yapmayı sevmeyiz. Ancak Build aşaması (inşaat aşaması) bir istisnadır.

* Zaman Tasarrufu: 40 dakika süren bir kurulumun 39. dakikasında hata alıyorsan, sorunu tahmin etmeye çalışmak yerine bu anahtarla içeri girip bakmak sana saatler kazandırır.

***
