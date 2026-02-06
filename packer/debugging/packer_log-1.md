---
icon: sim-card
---

# PACKER\_LOG=1

#### 1. `PACKER_LOG=1` Nedir?

Normalde Packer bir "Kara Kutu" gibidir. İçeri komut girer, dışarı Image çıkar. Ama arada bir şeyler patlarsa ne olduğunu göremezsin. Bilgisayarının terminaline `export PACKER_LOG=1` yazdığında (Windows'ta `set` ile), Packer "Verbose Mode"a geçer. Yani arka plandaki her fısıltıyı, her ağ bağlantısını, her dosya transferini ekrana basar.

#### 2. Neden Bu Kadar Karmaşık Görünüyor?

* Packer tek parça bir program değildir. Builder ayrı bir programcık, Provisioner ayrı bir programcık olarak çalışır.
* Bunlar kendi aralarında RPC (Remote Procedure Call) denilen bir yöntemle konuşurlar.
* Klasik Debug burada işe yaramaz. Çünkü ortada tek bir program yok, bir sürü küçük parçanın trafiği var. Bu yüzden logları okumak zorundasın.

#### 3. Logları Okuma Sanatı&#x20;

Packer çok yetenekli olduğu için aynı anda 3 farklı işi (Paralel) yapabilir. Bu durum logların karışmasına neden olur.

* Sorun: Ekrana akan yazılarda, 1. işin logunun hemen altına 3. işin logu düşebilir.
* Sıraya aldanmamamız, Timestamp'e (Zaman Damgasına) bakmak gerekiyor. Yani satırın başındaki saate bakarak olayların gerçek sırasını çözebilirsin.
* Ekranda loglar çok hızlı akar ve kaybolur. `PACKER_LOG_PATH`  değişkenini kullanarak bu akışı bir `.txt` dosyasına kaydetmek, sonra o dosyayı sakince incelemek en iyi yöntemdir.

#### 4. Hangi Hataları Yakalarız?

* SSH Bağlantı Sorunları: "Authentication failed" hatası aldığında, Packer'ın hangi kullanıcı adıyla, hangi key dosyasıyla gitmeye çalıştığını loglarda görürsün.
* Cloud-Init: Önceki konularda konuştuğumuz "Makine hazır olmadan komut gönderme" hatasını loglardaki zamanlamadan (Timing) anlarsın.
* Eğer bilgisayarın "Too many open files" diyorsa, Packer'ın aynı anda işletim sisteminin izin verdiğinden fazla dosya açmaya çalıştığını bu loglardan yakalarsın.

***

Özetle: `PACKER_LOG=1`, Packer'ın "Detaylı Loglama" (Verbose Logging) özelliğidir. Normalde ekranda sadece işlemin özeti görünürken, bu mod açıldığında arka plandaki tüm teknik süreçler (yapılan API çağrıları, SSH bağlantı detayları ve dosya transferleri) en ince ayrıntısına kadar görünür hale gelir. Karmaşık bir hata aldığınızda sorunun asıl kaynağını bulmak için bu ham verilere ihtiyaç duyarsınız.
