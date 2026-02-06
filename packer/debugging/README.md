---
icon: cloud-showers-water
---

# Debugging

#### 1. Interaktif Modlar

Packer'ın hızını kesip kontrolü senin eline verdiği komutlardır.

* `-debug` : Packer'ı bu parametreyle çalıştırırsan, her bir adımdan sonra durur ve sana "Enter'a basarsan devam edeceğim" der.
  * _Avantajı:_ Süreci kare kare izlemeni sağlar.
  * _Önemli Özellik:_ AWS gibi bulut ortamındaysan, Packer o an çalışan geçici makineye bağlanman için sana özel, geçici bir SSH Anahtarı (.pem dosyası) üretir ve bulunduğu klasöre bırakır. Sen de o anahtarla makineye girip içeride ne olup bittiğine bakabilirsin.
* `-on-error=ask` : Normalde Packer hata alınca makineyi siler. Bu komutu kullanırsan, hata anında durur ve sana sorar: "Hata oldu, temizleyip çıkayım mı, yoksa tekrar mı deneyeyim?". Bu, hatayı canlı canlı incelemek için mükemmeldir.

#### 2. Logging

Bazen ekranda gördüğün hata mesajı yetmez. Arka planda hangi API çağrısının yapıldığını veya hangi dosyanın kopyalanırken takıldığını görmek istersin.

* `PACKER_LOG=1`: Bu Environment Variable'ı (Ortam Değişkeni) tanımlarsan, Packer sana çok detaylı loglar verir.
* Metinde geçen "Sırasız görünme" uyarısı şudur: Packer aynı anda birden fazla işi (Paralel) yapabildiği için, log satırları bazen karışık zamanlarda ekrana düşebilir. Tarih/Saat damgasına dikkat etmek gerekir.

#### 3. En Sık Karşılaşılan Tuzak: "Race Condition"

Yeni başlayanların %90'ının yaşadığı ve dökümanda Cloud-init başlığıyla geçen sorun şudur:

* Senaryo: Packer AWS'de makineyi açar. Makine "Ping" vermeye başlar başlamaz Packer "Hadi yazılım kuralım" der.
* Sorun: Ama makine arka planda hala kendi iç ayarlarını (Cloud-init) yapıyordur. Packer komut gönderdiğinde "Ben daha hazırım değilim" hatası alırsın veya paket yöneticisi kilitlidir.
* Çözüm: `cloud-init status --wait` komutu, Packer'a "Acele etme, makine tam olarak kendine gelene kadar bekle" demektir.

#### 4. Sistem Kaynakları

Packer çok agresif ve hızlı çalışan bir araçtır. Eğer aynı anda 50 tane Image oluşturmaya çalışırsan, bilgisayarının veya sunucunun dosya açma limitlerine (ulimit) takılabilirsin.

***

Özetle: Packer ile Debugging yapmak; bir hata oluştuğunda sistemi hemen çöpe atmayıp, geçici anahtarlar ve loglar yardımıyla "ameliyat masasında" sorunu teşhis etme sürecidir. Unutma, amacımız çalışan sunucuyu tamir etmek değil, fabrikanın (Packer Template'in) hatalı üretim yapmasını engellemektir.
