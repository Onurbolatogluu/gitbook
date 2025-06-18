---
icon: hands-asl-interpreting
---

# Configure error handling

### 1. Tek sunucuda hata senaryosu

* Ansible task’ları sırayla çalıştırır.
* Bir task **FAILED** olursa playbook **hemen durur**.
* Tek makinede bu aslında iyi bir şey: Daha fazla bozmadan durur.

```yaml
- hosts: server1
  tasks:
    - name: Install dependencies
      # ...
    - name: Run web-server
      # hata -> playbook stop
```

***

### 2. Çoklu sunucuda (default davranış)

* Her task **tüm hostlar** üzerinde biter, sonra sonraki task’a geçer.
* Bir host’ta task patlarsa, **o host listeden çıkar**, kalan host’larda devam eder.
* Ama playbook sonuna kadar sürer, raporda “FAILED” hostları görürsün.

### 3. “Hiç taviz yok” dersi — `any_errors_fatal: true`

Dağıtımın **her yerde tutarlı** olmak zorundaysa:

```yaml
- name: Deploy web app
  hosts: webservers
  any_errors_fatal: true   # biri bile düşerse herkesi durdur
  tasks:
    - name: Start DB
      service:
        name: mysql
        state: started
```

> Bir host, bir task’ta çuvalladı mı? Ansible **tüm playbook’u** iptal eder.

### 4. “Belli bir eşiğe kadar sorun yok” — `max_fail_percentage`

Yüzlerce sunucuya rollout yapıyorsun, %5’i patlayabilir, dert etme; ama **%30’dan fazlası** patlarsa muhtemelen _gömülü hatan_ var:

```yaml
- hosts: big_farm
  max_fail_percentage: 30   # eşiği belirle
```

* Task başına hesaplar; toplam fail yüzdesi %30’u aşarsa playbook stop.

### 5. Kritik olmayan task’ı yok say — `ignore_errors: yes`

SMTP server dengesiz; “notification e-mail” task’ı çalışsa güzel, çalışmazsa da playbook çökmesin:

```yaml
- name: Send notification mail
  mail:
    to: admin@example.com
    subject: Done
  ignore_errors: yes
```

### 6. Çıktıya bakıp “fail” dedirt — `failed_when`

Log’larda “ERROR” varsa task’ı bilinçli olarak fail edelim:

```yaml
- name: Check server log
  command: cat /var/log/server.log
  register: log_out

- name: Fail if error in log
  debug:   # veya gerçek işi burada yaparsın
    msg: "Errors found!"
  failed_when: "'ERROR' in log_out.stdout"
```

> Task teknik olarak _başarılı_ çalışsa bile (`rc = 0`), koşul sağlanırsa Ansible onu **FAILED** sayar.
>
>
>
> * **`'ERROR' in log_out.stdout`** → Python’daki “içinde mi?” testi.
>   * Metnin **herhangi bir yerinde** büyük harfle **`ERROR`** kelimesi geçiyorsa **True** döner.
>   * Geçmiyorsa **False** döner.

### 7. Blok + Rescue tekrar (error handling’in süper gücü)

```yaml
- name: Deploy stack
  hosts: appservers
  tasks:
    - block:
        - name: Install packages
          yum: name=httpd state=present
        - name: Start service
          service: name=httpd state=started
      rescue:
        - name: Mail admins
          mail:
            to: admin@example.com
            subject: "Deploy FAILED on {{ inventory_hostname }}"
            body: "{{ ansible_failed_task.name }} task'ında hata!"
      always:
        - debug:
            msg: "HTTPD block finished (success or fail)"
```

* **block:** normal görevler.
* **rescue:** block içindeki _herhangi_ bir task fail ederse **hemen** çalışır.
* **always:** her koşulda (başarılı/başarısız) en sonda çalışır (cleanup, log, vs.).

| Ne?                       | Nasıl?                       |
| ------------------------- | ---------------------------- |
| **Anında stop**           | `any_errors_fatal: true`     |
| **Yüzdelik eşik**         | `max_fail_percentage: <int>` |
| **Task hatasını yok say** | `ignore_errors: yes`         |
| **Koşullu fail**          | `failed_when:`               |
| **Try/Catch benzeri**     | `block / rescue / always`    |

