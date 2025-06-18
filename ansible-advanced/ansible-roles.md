---
icon: user-tie-hair-long
---

# Ansible Roles

### 1. Rol fikri ne?

* Nasıl gerçek hayatta biri _doktor_ olsun diye lise → tıp fakültesi → staj → uzmanlık adımlarını tamamlarsak,
* Biz de “boş” bir sunucuyu _mysql‐server_ yapmak için **bir dizi task** çalıştırıyoruz (paket kur, servis başlat, config yap).

Bu task setini tek bir pakete sarıyoruz → buna **role** diyoruz. Sonra playbook’ta sadece:

```yaml
roles:
  - mysql
```

yazmak yetiyor; 1 sunucu da olsa 100 sunucu da olsa aynı reçete uygulanıyor.

### 2. Rol dizin yapısı (best practice)

```
roles/
└─ mysql/
   ├─ tasks/       # ana .yml dosyan (main.yml) burada
   ├─ handlers/    # notify ile tetiklenen restart vb.
   ├─ templates/   # Jinja2 şablonları
   ├─ files/       # düz kopyalanacak dosyalar
   ├─ vars/        # sabit değişkenler (yüksek öncelik)
   ├─ defaults/    # override edilebilir vars (düşük öncelik)
   └─ meta/        # rol bağımlılık bilgisi
```

> İskeleti **elle** oluşturmakla uğraşma:
>
> ```bash
> ansible-galaxy init mysql
> ```

### 3. Kendi rolünü yazma adımları

1. `ansible-galaxy init myrole` → klasör iskeleti oluşur.
2. `roles/myrole/tasks/main.yml` içine task’larını koy.
3. Gerekli template veya vars dosyalarını ilgili klasöre ekle.
4. Playbook dizininde **roles/** klasörü varsa Ansible otomatik bulur.
   * Global yerse **/etc/ansible/roles** (cfg’de `roles_path`).

Örnek hızlı task:

```yaml
# roles/mysql/tasks/main.yml
- name: Install MySQL packages
  yum:
    name: mysql-server
    state: present

- name: Enable & start service
  service:
    name: mysqld
    state: started
    enabled: true
```

Playbook’ta kullanımı:

```yaml
- hosts: db
  roles:
    - mysql
```

### 4. Hazır rol kullanmak (Ansible Galaxy)

1.  **Ara**

    ```bash
    ansible-galaxy search mysql
    ```
2.  **Kur**

    ```bash
    ansible-galaxy install geerlingguy.mysql   # varsayılan path’e
    # veya proje içi:
    ansible-galaxy install geerlingguy.mysql -p ./roles
    ```
3.  **Çağır**

    ```yaml
    roles:
      - geerlingguy.mysql
    ```

Ek parametre geçmek istersen:

```yaml
roles:
  - role: geerlingguy.mysql
    become: yes
    mysql_root_password: supersecret
```

### 5. Rol önceliği & değişken hiyerarşisi (özet)

| Nerede tanımlı?                 | Öncelik    |
| ------------------------------- | ---------- |
| **extra vars** (`-e`)           | 🥇         |
| **play vars**                   | 🥈         |
| **role vars** (`vars/`)         | 🥉         |
| **role defaults** (`defaults/`) | 🚦         |
| group/host vars                 | daha düşük |

Yani role içinde `defaults/main.yml`’a koyduklarını kullanıcı override edebilsin. Zorunlu sabitleri `vars/main.yml`’a koy.

### 6. Rol dizini nereye bakılır?

```bash
ansible-config dump | grep ROLE
```

Çıktıda `DEFAULT_ROLES_PATH` listesi gelir (global + user).\
Proje içi `roles/` klasörü _her zaman_ ilk bakılan yerlerden biridir.

### 7. Mini egzersiz (deneyip pekiştir)

1. `ansible-galaxy init nginx` komutuyla yeni rol oluştur.
2. `tasks/main.yml` içine `apt install nginx && systemctl enable` step’lerini ekle (Ubuntu düşün).
3. `defaults/main.yml`’a `nginx_listen_port: 8080` koy.
4. Template klasörüne bir `nginx.conf.j2` ekle, portu değişkenden çek.
5.  Playbook yaz:

    ```yaml
    - hosts: web
      roles:
        - role: nginx
          nginx_listen_port: 80   # override
    ```
6. Çalıştır, web sunucularında 80 portundan nginx geldiğini test et.

