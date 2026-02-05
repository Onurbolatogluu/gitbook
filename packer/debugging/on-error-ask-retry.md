---
icon: xmark-large
---

# -on-error=ask/retry

#### 1. `-on-error=ask` (Bekle, Hemen Çöpe Atma!)

Normalde Packer, bir hata oluştuğunda (örneğin bir dosya yüklenemediğinde) "Ben başarısız oldum" der ve ortalığı temizleyip (oluşturduğu sunucuyu silip) kapanır. Bu bayrağı kullandığında Packer şöyle davranır:

* Durum: Hata oluştu!
* Aksiyon: Packer makineyi silmez. Durur ve sana sorar: _"Patladık, ne yapalım? Temizleyip çıkayım mı, yoksa durayım mı?"_
* Faydası: Sen "Dur" dersin. Makine hala açık olduğu için içine girer (SSH ile), hatayı gözünle görür, belki manuel olarak düzeltmeyi denersin. Hatanın sebebini inceleme şansın olur.

#### 2. `-on-error=retry` (Tekrar Dene)

Bazen hatalar anlıktır (İnternet kopmuştur, paket sunucusu cevap vermemiştir). Bu seçenekle Packer'a şunu dersin: _"Eğer takılırsan, hemen pes etme, o adımı tekrar dene."_

#### 3. Proaktif vs. Reaktif Yaklaşım

Burada güzel bir karşılaştırma var:

* `-debug` : Süreç başlar başlamaz her adımda durur. "Adım adım gidelim, hata yapmayalım" mantığıdır. Yolculuğun başından itibaren kontrol sendedir.
* `-on-error` : Süreç normal hızında akar. Sadece kaza yaparsa devreye girer. "Hızlı git ama kaza yaparsan dur" mantığıdır.

#### 4. Operasyonel Kolaylık (SSH Anahtarı)

Yine önceki konularda değindiğimiz SSH Key (.pem dosyası) burada da devreye girer. Hata anında Packer durduğunda (`ask` modunda), sana geçici bir anahtar verir.

* Sen terminalden `ssh -i anahtar.pem ...` diyerek o an hata vermiş makineye bağlanırsın.
* _"Neden `apt-get install` çalışmadı?"_ diye loglara içeriden bakarsın.
* Özellikle Cloud-init (Sunucunun ilk açılış ayarları) bitmeden işlem yapmaya çalıştıysan, bunu sistemin içindeyken çok net görürsün.

***

Özetle:

* `-on-error=ask`: Hata olunca "Sakın silme, inceleyeceğim" demek için.
* `-on-error=retry`: "Belki şanssızlıktır, bir daha dene" demek için kullanılır. Amacımız; saatler süren Build işlemlerinin en sonunda çıkan ufak bir hata yüzünden tüm süreci çöpe atmamak ve zaman kazanmaktır.
