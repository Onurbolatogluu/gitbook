---
icon: temperature-three-quarters
---

# TMPDIR

Packer, eklenti tabanlı bir mimariye sahiptir. Ana süreç ile yardımcı süreçler (Builder, Provisioner) arasındaki veri akışı, Unix Sockets üzerinden sağlanır.

* Packer çalıştırıldığında, eklentilerle konuşmak için işletim sisteminin geçici dizininde (`/tmp` veya `var/tmp`) geçici soket dosyaları oluşturur.
* Kernel Kısıtı: Unix/Linux çekirdek yapısında, `sockaddr_un` yapısı içerisindeki adres yolu (Path) için katı bir karakter sınırı (Genellikle 104-108 bayt) bulunur. Bu sınır donanımsaldır ve aşılması durumunda soket bağlanamaz (Bind Failure).
* Eğer Packer'ın çalıştığı dizin veya sistemin varsayılan `TMPDIR` yolu çok uzunsa, oluşturulan soket dosyasının tam yolu kernel limitini aşar. Sonuç olarak kullanıcıya, kök nedeni belirsiz olan `"plugin exited before we could connect"` hatası döner.

