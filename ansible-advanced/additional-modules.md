---
icon: hammer-brush
---

# Additional Modules

🔧 1. **Package Modülü**

Hedef sunucuya yazılım yüklemek (örneğin web sunucusu, nginx, apache2, vs.).

**Yol 1: Dağıtıma özel modüller:**

```yaml
- name: Install httpd on CentOS
  yum:
    name: httpd
    state: present
```

```yaml
- name: Install apache2 on Ubuntu
  apt:
    name: apache2
    state: present
```

**Yol 2: Ortak modül (`package`) — Ansible 2.0+:**

```yaml
- name: Install web server using package module
  package:
    name: httpd
    state: present
```

> ⚠️ Ancak dikkat: Paket adları farklı olabilir!
>
> * CentOS’ta: `httpd`
> * Ubuntu’da: `apache2`

Bu yüzden, **koşullu ifadeler (when)** ya da **değişkenler** kullanmak mantıklıdır. Bunu ilerleyen konularda göreceğiz.

### 🛠️ 2. **Service Modülü**

Bir servisi başlatmak, durdurmak veya otomatik başlatılmasını sağlamak.

#### Örnek:

```yaml
- name: Start and enable httpd service
  service:
    name: httpd
    state: started
    enabled: yes
```

> * `state: started`: Servis hemen başlatılır.
> * `enabled: yes`: Sunucu yeniden başladığında servis otomatik açılır.

> * .

***

### 🔥 3. **Firewalld Modülü**

Port açmak, protokollere izin vermek vs.

#### Örnek:

```yaml
- name: Allow HTTP on port 8080
  firewalld:
    port: 8080/tcp
    zone: public
    state: enabled
    permanent: yes
    immediate: yes
```

> * `permanent: yes`: Kural kalıcı olur (reboot sonrası da geçerli).
> * `immediate: yes`: Kural **hemen** uygulanır.
> * Aksi takdirde reboot sonrası uygulanır ama hemen değil!

### 📦 4. **LVM Modülleri**

#### Adımlar:

1. `lvg`: LVM Volume Group (VG) oluşturur.
2. `lvol`: Logical Volume (LV) oluşturur.

#### Örnek:

```yaml
- name: Create volume group
  lvg:
    vg: vg1
    pvs: /dev/sdb1

- name: Create logical volume
  lvol:
    vg: vg1
    lv: data
    size: 2g
```

### 🧷 5. **FileSystem ve Mount Modülleri**

Diskin dosya sistemini oluşturmak ve mount etmek.

#### Örnek:

```yaml
- name: Create ext4 filesystem
  filesystem:
    fstype: ext4
    dev: /dev/sdb1

- name: Mount the filesystem
  mount:
    path: /opt/app
    src: /dev/sdb1
    fstype: ext4
    state: mounted
```

### 📁 6. **File Modülü**

```yaml
- name: Create directory
  file:
    path: /opt/app
    state: directory
    mode: '0755'
    owner: root
    group: root
```

> * `state: directory`: klasör oluşturur.
> * `state: touch`: boş dosya oluşturur.

### 📦 7. **Archive ve Unarchive Modülleri**

#### Sıkıştırmak:

```yaml
- name: Compress a directory
  archive:
    path: /var/www
    dest: /tmp/web.gz
```

#### Açmak:

```yaml
- name: Extract on remote host
  unarchive:
    src: /tmp/web.gz
    dest: /opt/app
    remote_src: yes
```

> * `remote_src: yes` → Dosya hedef sunucudaysa kullanılır.
> * `remote_src: no` (varsayılan) → Dosya kontrol sunucudaysa hedefe kopyalar.

### ⏰ 8. **Cron Modülü**

#### Örnek:

```yaml
- name: Run script every day
  cron:
    name: daily health check
    job: /usr/local/bin/health.sh
    minute: "10"
    hour: "8"
```

> * `minute: "*/2"` → Her 2 dakikada bir çalıştırır.
> * `weekday: "1"` → Sadece Pazartesi.
> * `*` işareti → herhangi bir zaman dilimi.

### 👤 9. **User, Group ve Authorized Key Modülleri**

#### Kullanıcı oluşturmak:

```yaml
- name: Create user
  user:
    name: devuser
    uid: 1050
    shell: /bin/bash
```

#### Grup oluşturmak:

```yaml
- name: Create group
  group:
    name: devgroup
```

#### SSH anahtarı eklemek:

```yaml
- name: Add SSH key to user
  authorized_key:
    user: devuser
    key: "{{ lookup('file', '/home/control/.ssh/id_rsa.pub') }}"
```

