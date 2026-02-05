---
icon: windows
---

# WinRM

#### 1. WinRM Şifre Emisyonu Nedir?

Windows sunuculara bağlanmak için genellikle bir anahtar dosyası değil, Kullanıcı Adı ve Şifre kullanılır. Packer, AWS üzerinde bir Windows makinesi açtığında, Amazon o makine için rastgele, karmaşık bir "Administrator" şifresi üretir.

* Sorun: Bu şifre şifrelenmiştir ve sen onu bilmiyorsun.
* Çözüm (Emission): Packer, senin adına AWS'ye gider, "Bu makinenin şifresini ver" der, o şifreyi çözer (decrypt eder) ve sana sunar. Buna "Password Emission" denir. Yani Packer, sana kapıyı açman için gereken parolayı teslim eder.

#### 2. Linux vs. Windows

Hata ayıklama (Debug) sürecinde iki dünya arasındaki fark şöyledir:

* Linux: Packer sana bir dosya verir (`.pem`). Sen `ssh -i dosya.pem ...` yazıp terminalden bağlanırsın.
* Windows: Packer sana açık bir şifre (text) verir. Sen bilgisayarındaki Uzak Masaüstü Bağlantısı (RDP) uygulamasını açarsın, IP adresini ve Packer'ın verdiği o şifreyi girip, Windows'un o bildiğimiz masaüstü ekranına bağlanırsın.

#### 3. Debugging:&#x20;

Diyelim ki Windows Image'ı hazırlarken IIS (Web Sunucusu) kurulumu patladı.

* `packer build -debug` komutuyla çalıştırdığın için Packer durur.
* Sana ekranda şifreyi yazar.
* Sen RDP ile bağlanıp, "Denetim Masası"na veya "Event Viewer"a (Olay Görüntüleyicisi) girip hatayı görsel olarak incelersin.

#### 4. İzinler ve Bürokraksi (IAM Policy)

Metindeki en kritik teknik uyarı `ec2:GetPasswordData` kısmıdır. Bunu şöyle düşün: Packer senin adına AWS'den şifreyi istiyor ama eğer Packer'a verdiğin yetkilerde (IAM User/Role) "Şifreleri görme yetkisi" yoksa, AWS "Sana şifreyi veremem" der ve süreç başarısız olur.

* Packer ile Windows çalışacaksan, IAM izinlerine bu maddeyi eklemeyi unutma.

#### 5. Ağ Engeli

Tıpkı SSH konusunda olduğu gibi, elinde şifrenin olması kapıya ulaşabileceğin anlamına gelmez. Eğer sunucuyu "Private Subnet" içine kuruyorsan, evindeki bilgisayardan RDP yapamazsın.

* Çözüm: Araya bir RD Gateway (Uzak Masaüstü Geçidi) veya VPN kurman gerekir. Bu, güvenli bir tünel açmak demektir.
