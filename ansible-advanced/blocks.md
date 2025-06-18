---
icon: block-brick
---

# Blocks

“**Blocks**” konusu Ansible’da hayat kurtaran, ama ilk bakışta “göz korkutucu” görünen bir özellik. Kısaca: **aynı ayarları paylaşan birden fazla task’ı tek çatı altında toplamak**, üstüne de **try/catch**‐benzeri hata yakalama eklemek için kullanıyoruz.

### 1. Block nedir?

```yaml
- block:
    # buraya task’ları koyarsın
  rescue:
    # block içindeki herhangi bir task patlarsa burası çalışır
  always:
    # patlasa da patlamasa da en sonda mutlaka çalışır
```

* **block:** → Asıl işlerin olduğu kısım.
* **rescue:** → Bir task “FAILED” olursa devreye girer.
* **always:** → Başarı/başarısızlık ne olursa olsun çalışır (log, temizlik vb.).

> **Hatırla:** `ansible_failed_task` ve `ansible_failed_result` değişkenleri rescue/always içinde otomatik gelir.

### 2. Neden block kullanıyoruz?

1. **Tekrardan kurtul**
   * `become_user`, `when`, `tags` gibi direktifleri blok seviyesine yazarsın, altında duran bütün task’lar miras alır.
2. **Hata yakalama**
   * Blok içindeki ilk başarısız task’tan sonra “rescue” bölümüne atlar; buraya e-posta atmaktan rollback’e kadar ne istersen koyarsın.
3. **Okunabilirlik**
   * MySQL task’larını bir blokta, Nginx task’larını başka blokta tutunca göz hemen hangi kısım ne yapıyor anlıyor.

### 3. Örnek: MySQL + Nginx kurulumu

#### 3.1 Tekrar dolu eski hâl

```yaml
- hosts: server1
  tasks:
    - name: Install MySQL
      yum:
        name: mysql-server
        state: present
      become_user: db-user
      when: ansible_facts['distribution'] == 'CentOS'

    - name: Start MySQL
      service:
        name: mysql-server
        state: started
      become_user: db-user
      when: ansible_facts['distribution'] == 'CentOS'

    - name: Install Nginx
      yum:
        name: nginx
        state: present
      become_user: web-user
      when: ansible_facts['distribution'] == 'CentOS'

    - name: Start Nginx
      service:
        name: nginx
        state: started
      become_user: web-user
      when: ansible_facts['distribution'] == 'CentOS'
```

#### 3.2 Block’lu temiz hâl

```yaml
- hosts: server1
  tasks:
    # --- DB BLOĞU ------------------------
    - block:
        - name: Install MySQL
          yum:
            name: mysql-server
            state: present

        - name: Start MySQL
          service:
            name: mysql-server
            state: started
      become_user: db-user
      when: ansible_facts['distribution'] == 'CentOS'
      rescue:
        - name: Notify DB install failure
          mail:
            to: admin@company.com
            subject: "DB ERROR on {{ inventory_hostname }}"
            body: "Task {{ ansible_failed_task.name }} failed: {{ ansible_failed_result.msg }}"
      always:
        - debug:
            msg: "DB block finished (success or fail)"

    # --- WEB BLOĞU -----------------------
    - block:
        - name: Install Nginx
          yum:
            name: nginx
            state: present

        - name: Start Nginx
          service:
            name: nginx
            state: started
      become_user: web-user
      when: ansible_facts['distribution'] == 'CentOS'
```

**Ne değişti?**

* `become_user` ve `when` tekrarı bitti; bir kere yazdım block üstüne.
* DB blokta hata olursa rescue tetikleniyor, Nginx tarafına hiç geçmiyor (istersen `ignore_errors: yes` ile davranışı değiştirebilirsin).
* always bölümü “try-finally” gibi garanti çalışıyor.

***

* **Block = grup + hata yönetimi**
* **rescue** sadece hata anında, **always** her durumda koşar.
* Ortak direktifler → block seviyesine; task üstündeki kalabalık azalır.
