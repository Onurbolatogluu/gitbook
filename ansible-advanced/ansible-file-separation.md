---
icon: folder
---

# Ansible File Separation

**“Ansible File Separation”** konusuna giriyoruz. Hedefimiz: envanteri, değişkenleri ve görevleri (tasks) dosyalara ayırarak **daha düzenli, yeniden kullanılabilir** bir yapı kurmak.

### 1. Sorun ne?

Inventory dosyasını şöyle tuttuğunu düşün:

```ini
[web_servers]
web1 ansible_host=172.20.1.100 dns_server=10.1.1.5
web2 ansible_host=172.20.1.101 dns_server=10.1.1.5
web3 ansible_host=172.20.1.102 dns_server=10.1.1.5
```

* **Değişkenler (vars)** de burada, host listesi de burada.
* Sunucu sayısı artınca dosya “çorba” oluyor.

### 2. Çözüm 1 – **host\_vars** (sunucuya özel değişkenler)

1. **Klasör**: `host_vars/`
2. **Dosya adı**: Sunucunun inventordaki ismi (web1, web2, …).
3. **İçerik**: YAML (anahtar: değer).

```bash
inventory/
├─ hosts          # ana envanter dosyası
├─ host_vars/
│   ├─ web1.yml
│   ├─ web2.yml
│   └─ web3.yml
```

`host_vars/web1.yml`:

```yaml
ansible_host: 172.20.1.100
size: big
```

> Dosya adı **web1.yml** olmalı ki Ansible “bu değişkenler web1’e ait” diye otomatik eşleştirsin; ekstra ayara gerek yok.

### 3. Çözüm 2 – **group\_vars** (gruba ortak değişkenler)

Aynı DNS adresi tüm web sunucularında ortaksa:

1. **Klasör**: `group_vars/`
2. **Dosya adı**: Grubun adı (`web_servers.yml`).

```bash
inventory/
└─ group_vars/
   └─ web_servers.yml
```

`group_vars/web_servers.yml`:

```yaml
dns_server: 10.1.1.5
```

→ Böylece ortak değerleri tek noktada tutarsın, kopyala-yapıştır biter.

### 4. Klasör düzeni önerisi

```bash
project/
├─ inventory/
│   ├─ hosts
│   ├─ host_vars/
│   └─ group_vars/
├─ playbooks/
└─ roles/ ...
```

İstasyondaki her şey (inventory, host\_vars, group\_vars) **inventory** dizinine toplanır; playbook dizini tertemiz kalır.

### 5. Vars dosyası “default” konumda değilse – `include_vars`

Merkezi bir yol düşün: `/opt/apps/common-data/email/info.yml`

```yaml
# info.yml
admin_email: admin@company.com
```

Playbook’ta kullanmak:

```yaml
- hosts: webservers
  tasks:
    - include_vars:
        file: /opt/apps/common-data/email/info.yml
        name: email_data          # değişken namespace'i

    - mail:
        to: "{{ email_data.admin_email }}"
        subject: "Deploy bitti"
        body: "Web sunucusu hazır"
```

Ansible çalışma anında dosyayı okuyup `email_data` sözlüğü içine yükler.

### 6. Envanteri “son hâliyle” görmek – `ansible-inventory -y`

```bash
ansible-inventory -i inventory/ -y
```

* Tüm host & group değişkenlerini **öncelik sırasına göre** birleştirip YAML basar.
* “Bu host hangi değeri nereden aldı?” sorusunu debug etmek için süper.

### 7. Görevleri bölmek – `include_tasks`

Tek playbook’ta DB + Web kuruyorsun ama bazen sadece DB lazım:

```bash
tasks/
├─ db.yml
└─ web.yml
```

`deploy.yml`:

```yaml
- hosts: webservers
  tasks:
    - include_tasks: tasks/db.yml   # sadece DB kur
    - include_tasks: tasks/web.yml  # web de kurulsun
```

Başka bir playbook yazıp yalnız `db.yml`’i include edersen saf DB kurulumu elde edersin. **Task’lar kopyalanmıyor, “import” ediliyor.**

### 8. Mini rehber – Değişken önceliği (en yüksek aşağıda)

1. **Extra vars** (`-e "key=val"`)
2. **Playbook içi vars**
3. **Host vars (host\_vars/)**
4. **Group vars (group\_vars/)**
5. **Inventory satırı**

→ Aynı isimli değişken çakışırsa yüksek öncelikli olan kazanır.

#### Özet

* **host\_vars/\<hostname>.yml** → O host’a özel değerler
* **group\_vars/\<groupname>.yml** → Gruba ortak değerler
* **include\_vars** → Değişken dosyası nerede olursa olsun yükle
* **include\_tasks** → Görev dosyalarını modüler kullan
* `ansible-inventory -y` → “Gerçek” envanteri gözden geçir
