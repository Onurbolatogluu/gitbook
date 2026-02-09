---
icon: windows
---

# WinRM Bağlantı Hataları

WinRM bağlantısının kurulabilmesi için ön koşul, Packer'ın hedef makinenin yönetici şifresini elde edebilmesidir.

* AWS gibi bulut sağlayıcıları, Windows sunucusu açıldığında rastgele bir şifre üretir ve bunu şifreli (Encrypted) olarak saklar.
* Packer, bu şifreyi çözmek için AWS API'sine çağrı yapar. Eğer Packer'ın çalıştığı kimlik (User/Role) `ec2:GetPasswordData` iznine sahip değilse, şifre alınamaz ve WinRM bağlantısı "Authentication Failed" hatasıyla (veya zaman aşımıyla) sonuçlanır.
* &#x20;Windows işletim sisteminin açılması, Sysprep süreçlerinin tamamlanması ve WinRM servisinin dinlemeye başlaması (Listening) zaman alır. Packer, sunucu durumu "Running" olsa bile WinRM portu cevap verene kadar beklemelidir. Erken yapılan bağlantı denemeleri Connection Refused ile sonuçlanır. Packer, Linux'taki gibi "bağlan ve bekle" yapmaz; bunun yerine "bağlanana kadar dene" stratejini izler.

