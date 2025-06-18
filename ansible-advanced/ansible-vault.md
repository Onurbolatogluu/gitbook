---
icon: vault
---

# Ansible Vault

### 1. Vault nedir, ne dert çözer?

* **Sorun:** Inventory veya vars dosyanda `ansible_ssh_pass=Passw0rd` gibi çıplak şifreler duruyor; repo’ya push edince herkes görebilir.
* **Çözüm:** `ansible-vault` ile dosyayı **AES-256** ile şifreler, sadece doğru vault parolasını giren okuyabilir/kullanabilir.

### 2. En basit akış – mevcut dosyayı şifrele

```bash
$ ansible-vault encrypt inventory   # veya secret_vars.yml
New Vault password: ********
Confirm New Vault password: ********
Encryption successful
```

* Dosyanın içi artık şöyle görünür:

```yaml
$ANSIBLE_VAULT;1.1;AES256
6637323134633863323435323765333865613366633930303031393832383864...
```

Okunmaz, playsiz.

#### Playbook çalıştırırken:

```bash
ansible-playbook site.yml -i inventory --ask-vault-pass
# şifreyi sorar, doğruysa devam eder
```

### 3. Şifreyi her seferinde yazmak yok mu?

#### 3.1 Parola dosyası (basit ama riskli)

```bash
echo 'MySup3rVault!' > .vault_pass.txt
chmod 600 .vault_pass.txt

ansible-playbook site.yml -i inventory --vault-password-file .vault_pass.txt
```

> `.vault_pass.txt`’i **git’e ekleme**! `.gitignore` şart.

#### 3.2 Dinamik parola script’i (güvenli yol)

```bash
# pwd_loader.py
#!/usr/bin/env python3
import os, requests
print(requests.get("https://secrets.example.com/vaultpass").text)
```

```bash
ansible-playbook site.yml --vault-password-file pwd_loader.py
```

Script şifreyi API, KeePass, AWS Secrets Manager… nereden istersen çeker.

### 4. Sık kullanılan vault komutları

| Komut                            | İşlev                                                       |
| -------------------------------- | ----------------------------------------------------------- |
| `ansible-vault encrypt file.yml` | Dosyayı şifrele                                             |
| `ansible-vault decrypt file.yml` | Dosyanın şifresini kaldır                                   |
| `ansible-vault view file.yml`    | İçeriği salt-okunur görüntüle                               |
| `ansible-vault edit file.yml`    | Şifreli dosyayı temp editörde aç (kayıt = yeniden şifreler) |
| `ansible-vault rekey file.yml`   | Parolayı değiştir                                           |

> `view` ve `edit` de `--ask-vault-pass` veya `--vault-password-file` ister.

### 5. Yalnızca **bir değişkeni** şifrelemek (dosyanın tamamını değil)

```bash
ansible-vault encrypt_string 'MySup3rDBpass!' --name 'db_password'
```

Çıktı:

```yaml
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          333461366432323839383938376538303466343262366231656164633164623565...
```

Bunu direkt vars dosyana yapıştır. Böylece dosyanın geri kalanı düz metin kalır.

### 6. Birden çok kasa (Vault ID) kullanmak

Prod ve Dev için farklı şifre ? Kullan:

```bash
ansible-vault encrypt --vault-id dev@prompt dev_vars.yml
ansible-vault encrypt --vault-id prod@prompt prod_vars.yml
```

Playbook:

```yaml
vars_files:
  - dev_vars.yml   # ortamına göre
```

Çalıştırırken Ansible hangi vault-id lazımsa onun parolasını sorar.

### 7. Küçük demo—şifreli vars dosyasıyla rol kullanımı

```
roles/
└─ mysql/
   └─ defaults/main.yml          # boş
   └─ vars/main.yml              # şifreli dosya
```

`roles/mysql/vars/main.yml`’i şifrele:

```bash
ansible-vault encrypt roles/mysql/vars/main.yml
```

Playbook:

```yaml
- hosts: db
  roles:
    - mysql
```

Çalıştır:

```bash
ansible-playbook db.yml --ask-vault-pass
```

Ansible önce roldeki vars’ı açar, sonra görevler çalışır.

#### Özet

* **Vault ⇒ dosya/variable şifreleme** (AES-256).
* Çalıştırırken parola vermek zorundasın (`--ask-vault-pass` veya `--vault-password-file`).
* Dosya tamamını ya da tek tek stringleri şifreleyebilirsin.
* Production’da parola script’i veya secret manager kullanmak en güvenlisi.

