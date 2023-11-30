---
description: Ansible
---

# 👉 Girizhah

![](../.gitbook/assets/ansible\_works.jpg)

* Python ve ruby dilleri ile geliştirilmiştir.
* 2015 'de Redhat tarafından satın alınıp, desteklenen open source bir projedir.
* IT otomasyon aracıdır.
* Sistem configuration/application deploy etmemizi sağlar.
* IT taskları ve orkestrasyonu sağlar.
* Kesintisiz deployment yeteneğine sahiptir.
* Genelde deployment işlemleri için SSH kullanır.
* Herhangi bir agent 'a ihtiyaç duymaz.
* Node'lara SSH ile bağlanırız.
* Farklı metotlar ile bağlantı kurabiliriz. SSH şart değil.
* AD-HOC commands ile node'lara, ansible 'ın desteklediği komutları çalıştırabiliriz.
  * Ping
  * Reboot
  * Dizin listelemek
  * Dosya oluşturmak
  * Dizin silmek
  * Kopyalamak
  * Permission ayarları
  * Paket yükleme
  * vb..

Ansible node'lara bağlandıktan sonra, modül olarak adlandırılan küçük programları node'lara gönderir. Yani çalıştırmak için push işlemi yapar. Node üzerinde, bu modülleri çalıştırır. İşi bittiğinde node 'da bu modül konfigürasyonu bulunmaz. Ansible otomasyon işlerini tanımlamak için ad-hoc komutları ve playbooks 'ları kullanır. Bunlar imperative ve declarative şeklindedir.
