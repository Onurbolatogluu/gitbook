---
icon: play-pause
---

# Use conditionals to control play execution

### 1. “when” anahtar sözcüğü

Ansible’da bir **task** yalnızca belli bir şart sağlanıyorsa çalışsın istiyorsan:

```yaml
when: <şart>
```

_Şart_ kısmı Jinja2 ifadesidir ve **True** dönerse task çalışır.

***

### 2. Birden fazla Linux dağıtımı için tek playbook

#### Problem

Debian tabanlı makinelerde `apt`, Red Hat tabanlılarda `yum` kullanman gerekiyor.\
İki ayrı playbook yerine tek dosyada halletmek için built-in **fact**’lerden yararlanırız:

| Built-in fact                  | Ne döner? (Örnek)      |
| ------------------------------ | ---------------------- |
| `ansible_os_family`            | `"Debian"`, `"RedHat"` |
| `ansible_distribution_version` | `"20.04"`, `"9"`       |

#### Çözüm

```yaml
- name: Install NGINX
  hosts: all
  tasks:
    - name: Install NGINX on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Install NGINX on Red Hat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"
```

> 🔑 **Çift eşittir (`==`)** ile karşılaştırmayı unutma.

***

### 3. Koşulları birleştirmek (and / or)

```yaml
when: ansible_os_family == "Debian" and ansible_distribution_version == "16.04"
```

```yaml
when: ansible_os_family == "RedHat" or ansible_os_family == "SUSE"
```

* `and` → ikisi de doğru olmalı
* `or` → herhangi biri doğruysa yeter

***

### 4. Döngülerde koşul (loop + when)

Elinde kurman gereken paket listesi olsun:

```yaml
vars:
  packages:
    - { name: nginx,  required: true  }
    - { name: mysql,  required: true  }
    - { name: apache, required: false }
```

Task:

```yaml
- name: Install "{{ item.name }}"
  apt:
    name: "{{ item.name }}"
    state: present
  loop: "{{ packages }}"
  when: item.required == true   # Sadece gerekli olanları kur
```

> Döngü “üç task” gibi çalışır; her iterasyonda `item` objesi değişir.\
> `when` satırı her item için ayrı değerlendirildiği için _**apache**_ es geçilir.

***

### 5. Önceki task’ın çıktısına göre karar (register)

1. **register** ile sonucu sakla
2. Sonraki task’ta o değişkeni incele

```yaml
- name: Check status of httpd
  command: service httpd status
  register: result

- name: Send alert e-mail if httpd is down
  mail:
    to: admin@company.com
    subject: Service Alert
    body: Httpd service is down
  when: result.stdout.find('down') != -1
```

* `result.stdout` komutun çıktısıdır.
* `find()` döndürdüğü indeks `-1` değilse “down” kelimesi geçiyor demektir → mail at.

***

### 6. Küçük ipuçları

| İpucu                                                                                                    | Neden önemli?                                     |
| -------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `ansible_facts` çıktısını görmek için `ansible -m setup` veya playbook içinde `debug: var=ansible_facts` | Koşul yazmadan önce hangi fact’ler geldiğini gör. |
| Birden çok “when” satırı yazmak yerine Jinja2’de parantez kullan                                         | Karmaşık koşulları okunur kılar.                  |
| Boole karşılaştırırken `true/false` **küçük harf**                                                       | YAML boolean’ı.                                   |
| Fact caching aç (`gather_facts: yes` + cache plugin)                                                     | Büyük envanterde performans kazanırsın.           |

***

`when` anahtar sözcüğü sayesinde tek playbook’la farklı dağıtımları, farklı durumları yönetebiliyorsun. Loops + conditionals + registered vars kombinasyonu da seni “if-else hell”’inden kurtarıp **idempotent** ve temiz otomasyon sağlar.



