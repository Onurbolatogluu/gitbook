---
icon: location-crosshairs
---

# SSH keys to managed nodes

### 1) SSH Key Nedir ve Neden Önemlidir?

* **SSH Key’ler**, makinenize kullanıcı adı ve şifre girmeden (passwordless) bağlanmanızı sağlayan bir “anahtar” sistemidir.
* İki dosyadan oluşur:
  1. **Private Key**: Sizin makinenizde (yerel) kalmalı ve asla başka biriyle paylaşılmamalıdır.
  2. **Public Key**: Hedef (remote) makinenin belirli bir dosyasında saklanır; bu sayede o makine, sizin Private Key’inizle eşleştiğini doğrular.

Bu ikiliyi bir “kilit ve anahtar” gibi düşünebilirsiniz. **Public Key** uzak makinede “kilit” görevi görür. Sadece “uyumlu Private Key” (sizin anahtarınız) o kilidi açabilir.

### 2) SSH Key’lerinizi Nasıl Oluşturursunuz?

1. **Kontrol Makinenize** (yani Ansible kurulu olan makineye) giriş yapın.
2.  Terminalde şu komutu çalıştırın:

    ```bash
    ssh-keygen
    ```
3. Komut size birkaç soru sorar (örneğin, key’i nereye kaydetmek istediğiniz veya bir parola belirlemek isteyip istemediğiniz gibi). Eğer varsayılan seçeneklerle devam ederseniz, `~/.ssh/id_rsa` (özel anahtar) ve `~/.ssh/id_rsa.pub` (açık anahtar) isimli iki dosya oluşur.

### 3) Public Key’i (Açık Anahtar) Uzak Makineye Nasıl Kopyalarsınız?

#### Yöntem 1: Elle Kopyalama

1.  **Public Key** içeriğine bakın:

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```
2.  Çıktıyı kopyalayın ve hedef makinede (örneğin `web1` sunucusunda) şu dosyaya ekleyin:

    ```
    ~/.ssh/authorized_keys
    ```

    Dosya yoksa oluşturabilirsiniz. Ardından aşağıdaki gibi kontrol edin:

    ```bash
    cat ~/.ssh/authorized_keys
    ```

    Public Key metniniz orada olmalı.
3. Artık Ansible kontrol makinenizdeki **özel anahtar** (id\_rsa) ile, hedef makineye şifresiz erişebilirsiniz.

#### Yöntem 2: `ssh-copy-id` Aracı

SSH copy-id, public key’inizi hedef makineye otomatik olarak ekleyen kolay bir araçtır:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@hedef_makine
```

Önce uzak makinenin şifresini sorar (ilk bağlantı için). Kopyalama bittiğinde şifreye gerek kalmaz.

### 4) Ansible Ortamında SSH Keys Kullanımı

1. **Kontrol Makinesi**: SSH anahtarlarınızı barındıran makine.
2. **Yönetilen Makineler (Managed Nodes)**: Public Key’inizi authorized\_keys dosyalarına eklediğiniz sunucular.

Ansible, hedef sistemlere bağlanırken SSH kullanır. Dolayısıyla, **private key**’iniz ile hedef makinelere parolasız erişebilirsiniz.

#### Envanter (hosts) Dosyanızı Güncelleme

Eskiden şöyle bir envanter kullanıyor olabilirsiniz:

```ini
web1 ansible_host=192.168.1.100 ansible_ssh_pass=Passw0rd
web2 ansible_host=192.168.1.101 ansible_ssh_pass=Passw0rd
```

Parolayla kimlik doğrulamayı bırakıp SSH Key’e geçmek için artık `ansible_ssh_pass` kısmını kaldırabilirsiniz. Ayrıca eğer **root** kullanıcısı dışında farklı bir kullanıcıyla (örnek: `ansible`) bağlanacaksanız, bunu belirtmeniz gerekir:

```ini
[webservers]
web1 ansible_host=192.168.1.100 ansible_user=ansible
web2 ansible_host=192.168.1.101 ansible_user=ansible
```

* Eğer **private key** (id\_rsa) dosyanız varsayılan `~/.ssh/` yolunda ise, Ansible bunu otomatik algılar.
*   Eğer özel bir yerde saklıyorsanız, şöyle belirtmeniz yeterli:

    ```ini
    web1 ansible_host=192.168.1.100 ansible_user=ansible ansible_ssh_private_key_file=/path/to/id_rsa
    ```

### 5) Sonraki Adımlar

1. **Şifre Tabanlı Girişleri Kapatma**: Sunucularınızda her şey sorunsuz çalıştıktan sonra, ekstra güvenlik için `PasswordAuthentication no` ayarlayarak parolalı girişi engelleyebilirsiniz.
2. **Otomatikleştirme**: Yüzlerce sunucunuz varsa, `ssh-copy-id` veya Ansible modülleri (örneğin, `authorized_key`) ile bu işlemi toplu halde yapabilirsiniz.
3. **Güvenlik İpuçları**: Private key’inizi mutlaka korunaklı bir yerde saklayın; paylaşılan dizinlerde bulundurmayın.

### 6) Özet

* **SSH Anahtarları** “kilit & anahtar” mantığıyla çalışır. Private Key sizde kalır, Public Key uzak makinede `authorized_keys` içine eklenir.
* **Parolasız Bağlantı**: Anahtarlar doğru eşleşiyorsa parola girmenize gerek kalmaz.
* **Ansible Entegrasyonu**: Ansible, normal SSH bağlantısını kullanır, dolayısıyla bu adımları yaptıktan sonra envanter dosyanızda kullanıcı ve key bilgilerini ayarlamanız yeterli.
* **Güvenlik**: Parolasız erişim sağlamak, doğru yapılandırıldığında (örneğin password-based login’i kapatarak) çok daha güvenli ve pratik bir yöntemdir.

