---
icon: simplybuilt
---

# Ansible Playbooks

### 1. Ansible Playbook Nedir?

* **Playbook**, Ansible’ın **temel orkestrasyon dili**dir.
* YAML formatında yazılır.
* İçinde, **hangi sunucularda (hosts)** hangi **işlemlerin (tasks)** yapılacağını tanımlarsınız.

#### Örnek Senaryolar

* Tek bir sunucuda `date` komutu çalıştırmak
* Yüzlerce sunucuda paket kurmak, servis başlatmak
* Özel script’leri çalıştırmak, disk yapılandırmak, VM’leri otomatik oluşturmak vb.



### 2. Yapıtaşları: Play → Tasks → Modules

#### Play

* **Playbook**, aslında bir **liste** (array) şeklinde yazılır.
* Bu listenin **her bir elemanına** **“Play”** denir.
* Her “Play”, **hangi sunuculara** işlem yapılacağını (hosts) ve o sunucularda hangi **tasks** sırasıyla çalıştırılacağını belirtir.

#### Tasks

* Bir Play içindeki “tasks” listesi, adım adım yapılacak **eylemleri** ifade eder.
* **Sıralama önemlidir**: İlk task biter, ikinci task başlar.
* Örnek task’lar: Komut çalıştırma, paket kurma, servis başlatma, script çalıştırma.

#### Modules

* Her “task” bir **modül** kullanır (`command`, `yum`, `service`, `script` vb.).
* **Modül**, “nasıl yapılacağını” bilen kod parçacığıdır. Örneğin `yum` modülüyle paket kurulur, `service` modülüyle servis başlatılır.



### 3. Basit Bir Örnek Playbook

Aşağıdaki playbook’ta **tek bir “play”** var, bu play **“localhost”** üzerinde 4 task çalıştırıyor:

```yaml
- name: Play 1
  hosts: localhost
  tasks:
    - name: Execute command 'date'
      command: date

    - name: Execute script on server
      script: test_script.sh

    - name: Install httpd service
      yum:
        name: httpd
        state: present

    - name: Start web server
      service:
        name: httpd
        state: started
```

1. **Play’in Adı**: “Play 1” (isteğe bağlı bir açıklama).
2. **hosts: localhost**: Bütün task’lar “localhost” üzerinde çalışır.
   * Envanterde “localhost” tanımlıdır.
   * İsterseniz `hosts: web` veya `hosts: db` diyerek gruplar hedefleyebilirsiniz (envanterde tanımlı olmalı).
3. **Tasks**:
   * `command: date` (sistem tarihi)
   * `script: test_script.sh` (bir script dosyası çalıştırır)
   * `yum:` (httpd paketini kurar)
   * `service:` (httpd servisini başlatır)

**Önemli**: Task’lar listesinde hangi sırada yazarsanız, o sırada çalışır. Örneğin httpd paketi kurulduktan sonra servis başlatılması mantıklı.



### 4. Birden Fazla “Play” İçeren Playbook

Bir playbook’ta birden fazla “play” tanımlayarak, farklı aşamalarda farklı sunuculara işlem yaptırabilirsiniz:

```yaml
- name: Play 1
  hosts: localhost
  tasks:
    - name: Execute command 'date'
      command: date

- name: Play 2
  hosts: web
  tasks:
    - name: Install httpd service
      yum:
        name: httpd
        state: present
    - name: Start web server
      service:
        name: httpd
        state: started
```

* Play1 → `hosts: localhost`, `date` komutu
* Play2 → `hosts: web`, sunucuya `httpd` kurar ve başlatır

Bunlar **sırayla** çalışır: önce Play1 biter, sonra Play2 başlar.



### 5. Playbook Nasıl Çalıştırılır?

```bash
aansible-playbook playbook.yml
```

* `playbook.yml`: Yukarıdaki örnekleri kaydettiğiniz YAML dosyası.
*   Eğer envanter dosyanız (inventory) varsayılan `/etc/ansible/hosts` dışında ise, `-i` parametresiyle belirtmeniz gerekir:

    ```bash
    ansible-playbook -i my_inventory.ini playbook.yml
    ```
*   Daha fazla seçenek için:

    ```bash
    ansible-playbook --help
    ```



### 6. Modüller (Modules) ve Dokümantasyon

Ansible, **yüzlerce modül** ile gelir (örn. `copy`, `file`, `lineinfile`, `shell`, `apt`, `yum`, `service`, `user` vb.).

*   Bilgi almak için:

    ```bash
    ansible-doc -l
    ```

    Tüm mevcut modülleri listeler.
* `ansible-doc <modül_adı>` komutuyla belirli modülün kullanımı incelenebilir.



### 7. Özet

* **Playbook**: Ansible’ın “ne yapılacak, hangi sırayla, hangi sunucularda” sorusuna cevaptır.
* **YAML** formatında yazılır, **liste** yapısıyla bir veya birden çok “play” içerebilir.
* **Her Play**: “hosts” + bir “tasks” listesi
  * “tasks” içinde sırasıyla **modülleri** çağırırsınız (komut, servis, paket kurma vb.).
* **Çalıştırma**: `ansible-playbook playbook.yml` ile.

Bu şekilde, Ansible Playbook’ların temel yapısını öğrendiniz. Sonraki adımlarda, farklı modüllerle ve daha gelişmiş özelliklerle (değişkenler, koşullu görevler, loop’lar vb.) playbook’ları zenginleştirip otomasyonunuzu genişletebilirsiniz.

