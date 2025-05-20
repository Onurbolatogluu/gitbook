---
icon: clipboard-list-check
---

# Adhoc Commands

Ad-hoc komutlar, Ansible ile tek seferlik veya hızlı doğrulama, test ya da bilgi toplama işlemleri yapmak istediğinizde kullanabileceğiniz **kısa** ve **pratik** komutlardır. Playbook’lar yerine doğrudan komut satırından (CLI) bir modül çağırarak veya rasgele bir komut (`-a`) çalıştırarak hedef sistemlerde işlem yapabilirsiniz.

### 1) Ad-Hoc Komut Nedir, Neden Kullanılır?

* **Playbook’lar**: Daha uzun vadeli, tekrar kullanılabilir, sürüm kontrolüne uygun otomasyon senaryoları için idealdir.
* **Ad-Hoc Komutlar**: Hızlı testler, tek seferlik işlemler, basit doğrulamalar veya “acil” müdahaleler için mükemmeldir.
  * Örneğin _“Bütün sunuculara ping at, yanıt geliyor mu?”_
  * “Tüm makinelerdeki `/etc/hosts` dosyasını hızlıca göreyim.”
  * “Tek bir satırla disk kullanımını kontrol edeyim.”

Kısacası, _Playbook yazmaya gerek kalmadan_ anında komut gönderip sonucu görebilirsiniz.

### 2) Temel Kullanım

#### (A) Basit Ping Testi

```bash
ansible -m ping all
```

* `-m ping`: “ping” modülünü (Ansible’ın dahili modülünü) kullan.
* `all`: Envanter dosyasında tanımlı _tüm_ host’lara git.
* **Amaç**: SSH bağlantısı ve kimlik doğrulaması doğru mu, host’lar erişilebilir mi test etmek.

Çıktı başarılıysa şöyle görürsünüz:

```
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
web2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**Not**: Bu ping, “ICMP” yerine Ansible modülünün SSH üzerinden gönderdiği “pong” cevabıdır.

#### (B) Arbitrary (Rasgele) Komut Çalıştırma

Mesela tüm host’larda `/etc/hosts` dosyasını görüntülemek:

```bash
ansible -a "cat /etc/hosts" all
```

* `-a`: Bir metin komutunu direkt parametre olarak ver.
* `all`: Yine tüm hostlar.
* Modül belirtmezseniz varsayılan “command” modülü devreye girer.

**Örnek Çıktı**:

```
web1 | CHANGED | rc=0 >>
127.0.0.1    localhost
...

web2 | CHANGED | rc=0 >>
127.0.0.1    localhost
...
```

Böylece aynı anda pek çok makinede basit komutları yürütebilirsiniz.

### 3) Sudo (Become) Kullanımı

Ad-hoc komutlarda da **sudo (become)** kullanabilirsiniz. Mesela paket kurulumu, servis restart gibi yönetici ayrıcalıkları gerektiren işlemleri tek satırla yapmak:

```bash
ansible -m apt -a "name=nginx state=present update_cache=yes" all --become --ask-become-pass
```

* `-m apt`: apt modülünü kullanarak paket yönetimi yap.
* `-a "..."`: Bu modüle verilmesi gereken argümanlar.
* `--become`: Bu işlemi _root_ (sudo) haklarıyla yap.
* `--ask-become-pass`: Sudo parolanızı interaktif olarak gireceksiniz.

### 4) Ad-Hoc Komutları Shell Script İçine Yazma

Tek tek komutları elle girmek yerine, sıralı birkaç ad-hoc komutunuzu bir **shell scripti** haline getirebilirsiniz. Örnek `check_servers.sh`:

```bash
#!/bin/bash

echo "=== Pinging all servers ==="
ansible -m ping all

echo "=== Checking /etc/hosts on all servers ==="
ansible -a "cat /etc/hosts" all
```

1. Dosyanızın başına `#!/bin/bash` ekleyin (shebang).
2.  Scripti çalıştırılabilir hale getirin:

    ```bash
    chmod +x check_servers.sh
    ```
3.  Artık tek komutla tüm işlemler:

    ```bash
    ./check_servers.sh
    ```

Bu yaklaşım özellikle:

* **Çevresel değişkenler** (ENV vars) ayarlayarak,
* **Sıralı** birkaç ad-hoc komutu bir defada çalıştırarak,
* Kısa bir _otomasyon iskeleti_ yaratmanıza olanak tanır.

### 5) Ad-Hoc Komut vs. Playbook

* **Ad-Hoc Komut**:
  * Hızlı test, küçük bir komut.
  * Unutmamak istiyorsanız script’e koyabilirsiniz, ancak yine de uzun vadede yönetimi zordur.
* **Playbook (YAML)**:
  * Daha kalıcı, düzenlenebilir ve denetlenebilir.
  * Geniş ölçekli otomasyon, tekrar kullanım, sürüm kontrolü için tavsiye edilir.

**Pratikte**:

* Bir şeyi hızlıca denemek için ad-hoc (örneğin “tüm makinelerde bir dosyayı sil”).
* Sık tekrar edeceğiniz veya karmaşık işlemleri playbook’a dönüştürmek genellikle en iyi yöntem.

### 6) Özet

1. **Ad-hoc komutlar**, Ansible modüllerini “tek satırlık” biçimde çağırarak çok sayıda sunucuda eş zamanlı işlemler yapmanıza olanak tanır.
2. **Basit testler** (ping), **dosya görüntüleme** veya **paket yönetimi** gibi işlerde fazlasıyla kullanışlıdır.
3. **Sudo** ve diğer Ansible özellikleri (ör. vars, tags vs.) de ad-hoc komutlarla kısmen kullanılabilir.
4. **Shell script’lerle** birkaç komutu arka arkaya çalıştırmak isterseniz, ad-hoc komutlarınızı script’e yazabilirsiniz.
5. **Playbook** hâlâ uzun vadede daha organize ve sürdürülebilir bir yaklaşım sunar, fakat ad-hoc komutlar “hemen şimdi” bir şey yapmak için idealdir.

