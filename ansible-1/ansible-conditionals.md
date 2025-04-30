---
icon: command
---

# Ansible Conditionals

### 1. Neden Koşullar Gerekli?

Diyelim ki:

* Debian tabanlı (Ubuntu gibi) sunucularda paketleri **apt** ile,
* RedHat tabanlı (CentOS gibi) sunucularda paketleri **yum** ile

kurmanız gerekiyor. Normalde iki ayrı playbook yazılabilir: _biri Debian için_, _biri RedHat için_. Ancak **koşullar** (conditionals) kullanarak **tek** bir playbook içinde hangi görevlerin hangi OS üzerinde çalışacağını belirleyebilirsiniz.

### 2. `when` (Clause) ile Koşullu Görevler

#### 2.1 Örnek: Farklı OS Aileleri

```yaml
- name: Install Nginx on multiple OS families
  hosts: all
  tasks:
    - name: Install Nginx on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian" and ansible_distribution_version == "16.04"

    - name: Install Nginx on RedHat or SUSE
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat" or ansible_os_family == "SUSE"
```

* `ansible_os_family` ve `ansible_distribution_version`, **Ansible Facts** tarafından sağlanan hazır değişkenlerdir. (Örn. “Debian”, “RedHat”, “SUSE”)
* `when: ansible_os_family == "Debian"` diyerek, sadece OS “Debian” olan makinelerde bu görevi çalıştırır.
* `and`, `or` gibi mantıksal operatörlerle daha karmaşık koşullar kurabilirsiniz.

#### 2.2 Dikkat Noktaları

* **Eşitlik Kontrolü**: `==` kullanmalısınız (tek `=` değil).
* **Facts’ın Toplanması**: Bu değişkenlerin (`ansible_os_family` vb.) hazır olması için “Gather Facts” açık olmalı veya OS bilgisi bir şekilde envanterden gelmelidir.

### 3. Döngülerde (Loops) Koşullar Kullanmak

Bazen bir **liste** (ör. `packages`) içindeki öğeleri sırayla kurmanız, ancak içlerinden sadece “gerekli” olanları yüklemek istersiniz. Bunun için de `when:`i döngü (`loop:`) içinde kullanabilirsiniz.

#### Örnek:

```yaml
- name: Install software packages
  hosts: all
  vars:
    packages:
      - name: nginx
        required: true
      - name: mysql
        required: true
      - name: apache
        required: false

  tasks:
    - name: Install "{{ item.name }}"
      apt:
        name: "{{ item.name }}"
        state: present
      loop: "{{ packages }}"
      when: item.required
```

* `loop: "{{ packages }}"`: Üç öğeyi (nginx, mysql, apache) sırayla işler.
* `when: item.required`: `required == true` olan paketlerde görevi çalıştırır, `false` olanı **geçer**.
* Böylece “apache” `required: false` olduğu için yüklenmez.

### 4. Önceki Görev Çıktısına Göre Koşullar

Ansible’da bir görevdeki çıktıyı (output) `register: sonuc` ile saklayıp **sonraki** görevde `when: ...` kullanarak karar verebilirsiniz.

#### Örnek: Servis Durumunu Kontrol Etmek

```yaml
- name: Check service status and send alert
  hosts: localhost
  tasks:
    - name: Check status of httpd service
      command: service httpd status
      register: result

    - name: Send email alert if httpd service is down
      mail:
        to: admin@example.com
        subject: "Service Alert"
        body: "Httpd Service is down"
      when: result.stdout.find('down') != -1
```

* İlk görev `service httpd status` çalıştırıp **çıktıyı** `result` adlı değişkende saklıyor.
* İkinci görev, `result.stdout` içindeki metinde `'down'` kelimesini arıyor (`find('down')`).
  * `find('down')` `-1` döndürmezse, demek ki kelime bulundu → “Servis down” demek, o durumda mail gönderiyor.

**Mantık**: “Eğer çıktı içinde 'down' varsa, e-posta at.”

### 5. İpuçları ve Özet

* **`when:`** cümlesi, bir görevin sadece belli bir koşul **doğru (true)** olduğunda çalışmasını sağlar.
* **Kullanım Senaryoları**:
  1. **Farklı OS’lerde farklı modül kullanmak** (Debian vs RedHat).
  2. **Bir döngüde** sadece bazı öğeleri işlemek (örn. `item.required == true`).
  3. **Önceki görevdeki kaydedilen çıktıya** göre sonraki görevi koşturmak veya atlamak.
* **Mantıksal Operatörler**: `and`, `or`, `==`, `!=`, `>` vb.
* **String Metotları**: `stdout.find('down') != -1` gibi Python benzeri ifadeler kullanabilirsiniz.
* **Koşul sağlanmazsa**: Görev “SKIPPED” olarak atlanır, playbook durmaz veya hata vermez.

Bu sayede **tek bir playbook** ile, sistemlerinizin farklı senaryolarına göre **dinamik** şekilde uygun görevleri çalıştırabilirsiniz. Daha temiz, bakımı kolay ve esnek bir Ansible otomasyonu elde edersiniz.

