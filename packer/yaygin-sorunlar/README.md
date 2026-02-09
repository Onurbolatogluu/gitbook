---
icon: exclamation
---

# Yaygın Sorunlar

Otomasyon süreçlerinde en sık karşılaşılan sorun, Packer'ın işletim sistemi tam olarak hazır hale gelmeden (Boot Complete) komut göndermeye çalışmasıdır.

* Özellikle Ubuntu gibi bulut imajlarında, `cloud-init` servisi arka planda sistem paketlerini güncellerken, Packer eş zamanlı olarak `apt-get install` komutunu gönderirse kilitlenme (Lock) veya "Paket bulunamadı" hatası oluşur.
* Çözüm: Provisioner adımının en başına, işletim sisteminin hazır olduğunu teyit eden bir bekleme mekanizması (`cloud-init status --wait`) eklenmelidir.&#x20;

#### 2. Sistem Kaynak Yönetimi

Packer, mimari olarak her bir eklentiyi (Builder, Provisioner) ayrı bir İşletim Sistemi Süreci (Process) olarak başlatır.

* Çok sayıda paralel derleme (Parallel Builds) yapıldığında, işletim sisteminin izin verdiği maksimum açık dosya limiti (`ulimit -n`) aşılabilir. Bu durum "Too many open files" hatasıyla sonuçlanır.
* Çözüm: Unix tabanlı sistemlerde `ulimit` parametrelerinin artırılması, kaynak darboğazını çözer.

#### 3. IPC ve Soket İletişimi

* Unix sistemlerinde soket dosya yolları için karakter sınırı (genellikle 100 civarı) bulunmaktadır. Eğer Packer'ın çalışma dizini veya `TMPDIR` yolu çok uzunsa, soket oluşturulamaz ve eklenti başlatılamaz.
* Çözüm: Geçici dizin değişkeni (`TMPDIR`), daha kısa bir dizine (Örn: `/tmp`) yönlendirilerek soket yolu kısaltılmalıdır.

#### 4. Entegrasyon Karmaşıklığı

* Ansible'ın standart çalışma mantığındaki "Grup Değişkenleri" (Group Vars) veya karmaşık Jinja2 makroları, Packer'ın izole çalışma ortamında (Chroot/Proxy) her zaman beklenildiği gibi davranmayabilir.
* Ansible rollerinin Packer uyumlu hale getirilmesi (Variables'ın rol içine taşınması) ve "Playbook" yapısının basitleştirilmesi gerekebilir.

#### 5. Ağ Topolojisi ve Erişim Kontrolü

* Packer, derleme sırasında sunucuya SSH/WinRM ile bağlanmak zorundadır. Eğer sunucu internete kapalı bir ağdaysa bağlantı zaman aşımına uğrar.
* Ara sunucular (Bastion Host) veya Windows ortamları için RD Gateway kullanımı gereklidir. Ayrıca AWS tarafında `ec2:GetPasswordData` gibi IAM izinlerinin eksiksiz tanımlanması zorunludur.

#### 6. Gözlemlenebilirlik ve Log Analizi

* Birden fazla "Thread" aynı anda log bastığı için, hata mesajları karışık sırada görünebilir.
* &#x20;`PACKER_LOG=1` ile detaylı loglama açılmalı ve analiz sırasında Zaman Damgaları (Timestamps) referans alınarak olay örgüsü yeniden kurulmalıdır.
