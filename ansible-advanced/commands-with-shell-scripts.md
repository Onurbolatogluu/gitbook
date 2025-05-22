---
icon: scroll-torah
---

# Commands with shell scripts

### 1) Ad-Hoc Komut Nedir?

* **Ad-hoc komut**: Ansible modüllerini tek satırda çağırmanızı sağlayan hızlı bir yol. Playbook yazmaya gerek kalmadan, örneğin “tüm sunucuları ping’le” veya “/etc/hosts dosyasını göster” gibi basit işler için kullanılır.
*   Komut satırından (CLI) şu şekilde çalıştırırsınız:

    ```bash
    ansible -m ping all
    ansible -a 'cat /etc/hosts' all
    ```

    _İlki ping modülüyle SSH bağlantısını test eder, ikincisi tüm makinelerde /etc/hosts dosyasını görüntüler._

### 2) Neden Shell Script Oluşturuyoruz?

Bazen, ad-hoc komutları **art arda** çalıştırmak veya belirli **ortam değişkenlerini** (env vars) otomatik ayarlamak isteyebilirsiniz. Elbette her seferinde elle yazmak mümkün, ancak daha verimli bir yol:

1. **Tekrar eden** birkaç ad-hoc komutu sıralı şekilde çalıştırmak istiyorsunuz.
2. **Konfigürasyon değişkenlerini** (ör. `ANSIBLE_GATHERING`, `ANSIBLE_CONFIG`) her seferinde elle girmek istemiyorsunuz.
3. **Takım arkadaşlarınız** aynı komutları çalıştıracaksa, onlara tek bir script verip, tek komutla tüm işlemleri yaptırabilirsiniz.

### 3) Örnek Shell Script

Diyelim ki:

1. Ortam değişkeni `ANSIBLE_GATHERING`’i `explicit` yapacaksınız (fact toplama davranışı).
2. Ardından tüm host’lara ping atmak,
3. `/etc/hosts` dosyasını göstermek,
4. Son olarak küçük bir playbook çalıştırmak istiyorsunuz.

Script’inizde (`my_script.sh`) şöyle olabilir:

```bash
#!/bin/bash

# 1) Ortam değişkenini ayarla
export ANSIBLE_GATHERING=explicit

# 2) Ping modülünü kullanarak tüm sunucuları test et
ansible -m ping all

# 3) /etc/hosts içeriğini görüntüle
ansible -a "cat /etc/hosts" all

# 4) Bir playbook'u çalıştır
ansible-playbook deploy.yml
```

**Not**: `#!/bin/bash`, bu dosyanın bir “bash” script’i olduğunu belirten “shebang” satırıdır.

### 4) Script’i Nasıl Çalıştırırım?

#### Yöntem 1: `sh` ile

```bash
sh my_script.sh
```

#### Yöntem 2: Doğrudan Yürütülebilir (Executable) Hale Getirme

1.  Dosyayı yürütülebilir yapın:

    ```bash
    chmod 755 my_script.sh
    ```
2.  Ardından çağırın:

    ```bash
    ./my_script.sh
    ```

### 5) Özet Akış

1. **Ad-hoc komutunuzu** normalde tek satırda çalıştıracağınız kod olarak düşünün.
2. Bunları **art arda** sıralamak istiyorsanız veya **ortam değişkenleri** ayarlamak istiyorsanız, bir **shell script** yazmanız faydalı olur.
3. Bu script’i başka projelerde veya ekip arkadaşlarınızla paylaşarak **standart bir işlem akışı** oluşturabilirsiniz.
