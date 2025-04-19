---
icon: screwdriver
---

# Variable Scoping

Ansible’da değişkenlerin **erişim alanı** (scope) farklı seviyelerde olabilir. İster envanter dosyasında (host bazında) tanımlayın, ister bir play içerisindeki `vars:` bölümünde tanımlayın, ister komut satırında `--extra-vars` olarak geçin – hepsi ayrı kapsam kuralları içerir.

**Üç temel kapsam (scope) vardır**:

1. **Host Scope** (Host’a Özel Değişkenler)
2. **Play Scope** (Bir Play İçinde Tanımlanan Değişkenler)
3. **Global Variables** (Tüm Playbook Boyunca Geçerli Değişkenler)

Her birinin nasıl çalıştığını görelim.

### 1. Host Scope (Host’a Özel Kapsam)

* **Tanım**: Host bazında tanımlanan değişkenler, _sadece_ o host’ta çalışan görevler (tasks) için geçerlidir.
* **Nerede Tanımlanır?**
  * Envanter dosyasında (INI formatında) `host satırı` şeklinde (örn. `web2 dns_server=10.5.5.4`)
  * `host_vars` dizininde o host’un adına ait YAML dosyasında (ör. `host_vars/web2.yml`)

**Örnek**:

```ini
web1 ansible_host=192.168.1.101
web2 ansible_host=192.168.1.102 dns_server=10.5.5.4
web3 ansible_host=192.168.1.103
```

* Burada `web2` için `dns_server=10.5.5.4` tanımlanmış.
* `web1` veya `web3` üzerinde bu değişken **tanımsız** (undefined) olacaktır.

**Playbook’ta Kullanım**:

```yaml
- name: Print DNS server
  hosts: all
  tasks:
    - debug:
        var: dns_server
```

* Çıktı:
  * `web2` → `dns_server=10.5.5.4`
  * `web1`, `web3` → “VARIABLE IS NOT DEFINED!”

**Neden?** Çünkü `dns_server` yalnızca `web2` host’una ait bir değişken.

### 2. Play Scope (Bir Play İçinde Tanımlanan Değişken)

* **Tanım**: `vars:` satırıyla veya benzer şekilde bir Play içinde tanımlanan değişkenler, _sadece_ o play boyunca geçerli olur. Başka play’lere (aynı playbook içinde olsa bile) aktarılmaz.
* **Örnek**:

```yaml
- name: Play1
  hosts: web1
  vars:
    ntp_server: 10.1.1.1
  tasks:
    - debug:
        var: ntp_server

- name: Play2
  hosts: web1
  tasks:
    - debug:
        var: ntp_server
```

**Çıktı**:

* **Play1** → `ntp_server=10.1.1.1`
* **Play2** → `ntp_server=VARIABLE IS NOT DEFINED!`

Çünkü `Play2` içerisinde `ntp_server` tekrar tanımlanmadı. Play1’de tanımlanan değişken, Play2’ye miras kalmaz.

### 3. Global Variables (Tüm Playbook Boyunca Erişim)

* **Tanım**: Tüm playbook boyunca (birden fazla play olsa bile) erişilebilen değişkenler.
* **Nasıl Tanımlanır?**
  * **En yaygın yöntem**: Komut satırında `--extra-vars "key=value"` şeklinde verilirse, playbook’un tamamında kullanılır.
  * Bazı durumlarda (daha ileri konularda), Ansible’ın “global vars” mantığıyla her play’e ulaşan değişkenler düzenlenebilir. Ancak tipik senaryo, extra vars’tır.

**Örnek** Komut Satırı:

```bash
ansible-playbook myplaybook.yml --extra-vars "ntp_server=10.1.1.1"
```

* Bu komut çalıştığında, `ntp_server` değeri _playbook’taki tüm play’ler_ için tanımlı hale gelir.
* İki ayrı play’de de `debug: var=ntp_server` yaptığınızda, aynı değeri görürsünüz.

### Özet Tablosu

| Kapsam (Scope)  | Nerede Tanımlanır?                                                                               | Geçerlilik Alanı                                                |
| --------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------- |
| **Host Scope**  | <p>- Envanterde host satırı (INI)<br>- <code>host_vars/host.yml</code></p>                       | Yalnızca _o host_ üzerinde çalışan görevler                     |
| **Play Scope**  | - Play içinde `vars:` veya `vars_files:` vb.                                                     | Yalnızca _o play_ boyunca geçerli (sonraki play’ler göremez)    |
| **Global Vars** | <p>- Komut satırı <code>--extra-vars</code> (en sık)<br>- (İleri kullanımda başka yöntemler)</p> | Tüm playbook boyunca (birden fazla play olsa bile) herkese açık |

