---
icon: paintbrush
---

# Cloud-init

Otomasyon süreçlerinde en sık karşılaşılan kronik sorun, Packer'ın işletim sistemi tam olarak hazır hale gelmeden (Boot Complete) komut göndermeye çalışmasıdır.

* Modern Linux dağıtımları (özellikle Ubuntu), açılışta `cloud-init` servisini asenkron olarak başlatır. Bu servis arka planda paket veritabanını güncellerken veya sistem ayarlarını yaparken, Packer eş zamanlı olarak `apt-get install` gibi bir komut gönderirse, paket yöneticisi kilitli olduğu için (APT Lock) süreç başarısız olur ("No candidate version found").
* Provisioner adımının en başına, işletim sisteminin başlatma rutinlerinin tamamlandığını teyit eden bir Bekleme Bariyeri (`cloud-init status --wait`) eklenmelidir.
