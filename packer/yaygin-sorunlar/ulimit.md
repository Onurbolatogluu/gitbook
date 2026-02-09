---
icon: virus-slash
---

# ulimit

* Şablonda tanımlanan her `builder`, `provisioner` ve `post-processor`; ana Packer sürecinden ayrılarak kendi başına çalışan bağımsız bir işletim sistemi süreci (OS Process) olarak başlatılır.
* Her yeni süreç ve bu süreçler arasındaki iletişim kanalları (Pipe/Socket), işletim sistemi çekirdeğinde birer Dosya Tanımlayıcısı (File Descriptor - FD) tüketir. Karmaşık ve çok bileşenli derleme süreçlerinde, varsayılan kullanıcı limiti (`ulimit -n`) hızla dolabilir.
* Sistem, yeni süreç başlatamaz hale geldiğinde `fork/exec: too many open files` hatası fırlatılır. Çözüm, Kernel seviyesinde kullanıcı limitlerinin (`ulimit -Sn`) artırılmasıdır.

