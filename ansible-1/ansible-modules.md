---
icon: bowl-chopsticks-noodles
---

# Ansible Modules

### 1. Ansible Modülleri Nedir?

* **Ansible**’da her “işlemi” yapmak için bir **modül** (module) vardır. Örneğin bir paketi kurmak, bir kullanıcı eklemek, bir servis başlatmak için ayrı modüller kullanırsınız.
* Modüller, Ansible’ın **temel** “inşa taşları”dır. Her “task” (görev) yazarken, hangi modülü kullanacağınızı belirler, gerekli parametreleri verirsiniz.

**Örnek:**

```yaml
- name: Start httpd service
  service:
    name: httpd
    state: started
```

Bu görevde “service” modülünü kullanıp, “name” ve “state” parametrelerini giriyoruz.

### 2. Modül Kategorileri

#### 2.1 System Modülleri

* **Amaç**: İşletim sistemi düzeyinde değişiklikler yapmak (kullanıcı eklemek, servis çalıştırmak, iptables ayarlamak, disk mount etmek vb.).
* **Örnek**: `user`, `group`, `service`, `firewalld`, `mount` gibi modüller.

#### 2.2 Command Modülleri

* **Amaç**: Uzak sunucuda komut/script çalıştırmak.
* **Örnek**:
  * `command`: Basit komut çalıştır.
  * `script`: Yerel bir script’i uzak makineye kopyalayarak çalıştırır.
  * `expect`: Etkileşimli komutlar (parola soran vb.) için yanıt verebilir.

#### 2.3 File Modülleri

* **Amaç**: Dosya ve dizin işlemleri, içerik değiştirme, kopyalama, sıkıştırma vb.
* **Örnek**:
  * `copy`: Bir dosyayı kaynaktan hedefe kopyalar.
  * `lineinfile`: Bir dosyada belli bir satırı ekler veya değiştirir.
  * `find`, `replace`, `archive`, `unarchive` vb.

#### 2.4 Database Modülleri

* **Amaç**: Veritabanı yönetimi (MySQL, PostgreSQL, MongoDB, vb.).
* **Örnek**: `mysql_db`, `postgresql_db`, `mongodb` vb. Veritabanı oluşturma, silme veya konfigürasyon değiştirme.

#### 2.5 Cloud Modülleri

* **Amaç**: AWS, Azure, GCP, VMware gibi bulut/virtualization ortamlarında kaynak yönetimi (instance oluşturma, silme, network ayarları vb.).
* **Örnek**: `ec2`, `azure_rm`, `vmware_guest`, `docker_*` vb.

#### 2.6 Windows Modülleri

* **Amaç**: Windows makineleri yönetmek (dosya kopyalama, servis yönetimi, registry değişikliği vb.).
* **Örnek**: `win_copy`, `win_command`, `win_service`, `win_regedit`, vb.

### 3. Bazı Özel Modüllere Derin Bakış

#### 3.1 `command` Modülü

* **İşlev**: Uzak sunucuda bir komut çalıştırır.
*   **Örnek Task**:

    ```yaml
    - name: Execute date command
      command: date
    ```
* **Ek Parametreler**:
  * `chdir`: Komut öncesi dizin değiştirme
  * `creates`: Belirtilen dosya/dizin zaten varsa komutu çalıştırma
  * `removes`: Belirtilen dosya/dizin yoksa komutu çalıştırma

**Bu üç parametre** (`chdir`, `creates`, `removes`), **`command` modülü** (ve benzeri bazı modüller) için “komut çalıştırmadan önce” ek kontroller veya davranışlar tanımlamanızı sağlar. Ne anlama geldiklerine bakalım:

1. **`chdir` (Change Directory)**
   * Komutu çalıştırmadan önce **hangi dizine geçileceğini** (cd komutu gibi) belirtir.
   *   Örneğin:

       ```yaml
       - name: List files in /etc
         command: ls
         args:
           chdir: /etc
       ```

       Bu, `ls` komutunu `/etc` dizini içinde çalıştırır (yani `cd /etc; ls` gibi düşünün).
2. **`creates`**
   * Belirttiğiniz dosya veya dizin **zaten varsa**, komut **çalıştırılmaz**.
   * Yani `creates=/tmp/done_file` derseniz, `/tmp/done_file` _varsa_ o komut “Zaten yapılacak bir şey yok” diyerek SKIPPED olur.
   *   **Örnek**:

       ```yaml
       - name: Create folder if it doesn't exist
         command: mkdir /myfolder
         args:
           creates: /myfolder
       ```

       Eğer `/myfolder` zaten varsa, `mkdir` çalışmaz. Yoksa da komut çalışıp klasör oluşturur.
3.  **`removes`**

    * Belirttiğiniz dosya/dizin **yoksa**, komut **çalıştırılmaz**.
    * Ters mantık: “Bu dosya/dizin var olmadan bu komutu çalıştırma.”
    *   Örnek:

        ```yaml
        - name: Remove folder if it exists
          command: rm -rf /myfolder
          args:
            removes: /myfolder
        ```

        Yani `/myfolder` zaten yoksa, bir şey yapmaz; varsa komutu çalıştırıp siler.



* **Free Form**: `command:` modülünde komutu serbest şekilde yazabilirsiniz (örn. `command: "cat /etc/resolv.conf"`).

#### 3.2 `script` Modülü

* **İşlev**: Kontrol makinesindeki bir script’i (örn. `myscript.sh`) otomatik olarak uzak sunucuya kopyalar ve orada çalıştırır.
*   **Örnek Task**:

    ```yaml
    - name: Run a local script
      script: /path/to/local_script.sh --arg1 --arg2
    ```
* Böylece yüzlerce sunucuda script’i manuel kopyalamaya gerek kalmaz.

#### 3.3 `service` Modülü

* **İşlev**: Bir servisi **başlat**, **durdur**, **yeniden başlat**, veya **enable**/**disable** yapmak.
* **Parametreler**:
  * `name`: Servis adı (örn. `nginx`, `httpd`)
  * `state`: `started`, `stopped`, `restarted`, `reloaded`
* **Idempotency**: `started` demek “Bu servis _çalışır durumda_ olmalı” - eğer zaten çalışıyorsa ekstra işlem yapmaz. Bu yaklaşıma “idempotent” denir.

#### 3.4 `lineinfile` Modülü

* **İşlev**: Bir dosyada belirli bir satırı eklemek/değiştirmek, yoksa eklemek.
*   **Örnek**:

    ```yaml
    - name: Add DNS to resolv.conf
      lineinfile:
        path: /etc/resolv.conf
        line: "nameserver 10.1.250.10"
    ```
* Tekrar tekrar çalıştırılsa da sadece **bir kez** ekler (idempotent davranış).

### 4. Idempotency (Neden `started` ve `start` Farklı?)

Ansible modülleri çoğunlukla **idempotent** olacak şekilde tasarlanmıştır. Yani aynı playbook’u birçok kez çalıştırsanız da sistemin son durumu tutarlı kalır.

* `service: state=started`
  * Eğer servis kapalıysa açar, açıksa dokunmaz. (Fark: “start” ifadesi ile her seferinde restart gibi işlem yapmıyoruz.)

Bu sayede konfigürasyonunuzu defalarca çalıştırıp her seferinde **uygun halde** kalmasını sağlarsınız.

{% embed url="https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html" %}
