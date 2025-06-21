---
icon: cassette-betamax
---

# Templating use cases

### 1) Statik Dosya Kopyalama vs. Dinamik Template

* **copy Modülü**: Ansible’daki `copy` modülü, yerel makinedeki bir dosyayı hedef sunucuya **olduğu gibi** kopyalar. Bu yöntem, dosyanın içeriğinde değişken kullanıp “sunucuya göre özelleştirme” yapmanıza izin vermez.
* **template Modülü**: İşte burada devreye “template” gelir. Dosyanız **Jinja2 şablon formatında** hazırlanır ve Ansible, o şablondaki değişkenleri hedef sunucuya ait değerlerle doldurur.

**Örnek İhtiyaç**:\
Web sunucusuna yerleştireceğimiz `index.html` dosyasında, her sunucunun adını (hostname) dinamik olarak göstermek istiyoruz.

### 2) Jinja2 Şablonu Nedir?

Bir `.j2` (veya `.j2` uzantılı) dosyası düşünün. HTML, YAML, vb. herhangi bir metin formatında olabilir. İçinde Jinja2 sözdizimiyle değişken yerleştirebilirsiniz:

```jinja
<!-- index.html.j2 -->
<!DOCTYPE html>
<html>
<body>
This is {{ inventory_hostname }} server.
</body>
</html>
```

Burada `{{ inventory_hostname }}` Ansible’ın otomatik sağladığı bir değişkendir. Hostun adını (ör. “web1”) koyar.

### 3) Ansible’da Template Modülü Kullanmak

* **Playbook Örneği**:

```yaml
- name: Deploy index.html template
  hosts: webservers
  tasks:
    - name: Create dynamic index.html on each webserver
      template:
        src: index.html.j2
        dest: /var/www/html/index.html
```

**Açıklama**:

1. `src: index.html.j2` → Yereldeki **Jinja2 şablonu** (kaynak dosya).
2. `dest: /var/www/html/index.html` → Hedef sunucuda oluşacak gerçek dosyanın konumu.
3. Ansible, `index.html.j2` içindeki `{{ inventory_hostname }}` gibi değişkenleri uygun değerlerle **interpole** (değiştirir) edip kopyalar.

### 4) Nasıl Çalışır?

Playbook çalıştığında:

1. **Envanter** (inventory) veya diğer değişken kaynaklarından (vars, group\_vars, vb.) host bilgileri okunur.
2. **Jinja2 Motoru**: `index.html.j2` dosyasındaki `{{ ... }}` yerlerine host özelindeki değerler konur.
3. **Sonuç**: Her sunucuya **farklı** içerik sahip bir `index.html` kopyalanmış olur.
   * Örnek: web1’de `This is web1 server.`, web2’de `This is web2 server.` yazar.

Bu şekilde `copy` modülüyle ayrı ayrı “index\_web1.html”, “index\_web2.html” gibi dosyalar üretmek zorunda kalmazsınız. Tek bir şablon, **tüm** hostlar için uygun olacak şekilde değişkenlerle çalışır.

### 5) Biraz Daha Detay: Değişkenler, Filtreler, Koşullar

Jinja2 şablon içinde **Ansible değişkenlerini** kullanabilir, hatta Jinja2’ye ait filtreleri (`| default`, `| upper` vs.) veya `if`, `for` gibi yapıları kullanabilirsiniz.

#### a) Default Filtre Örneği

```jinja
<!-- redis.conf.j2 -->
port {{ redis_port | default("6379") }}
```

* `redis_port` tanımlanmazsa varsayılan **6379** kullanılır.

#### b) Döngü Örneği: resolv.conf

```jinja
<!-- resolv.conf.j2 -->
{% for ns in nameservers %}
nameserver {{ ns }}
{% endfor %}
```

* `nameservers` bir liste olduğu varsayılırsa (ör. `[ "8.8.8.8", "8.8.4.4" ]`), döngü her eleman için bir `nameserver` satırı ekler.

### 6) Şablon Dosyası Nereye Konur?

* Basit projelerde, playbook’unuzla aynı dizinde veya “templates/” gibi klasörde tutabilirsiniz.
* **Roles** kullanıyorsanız, `roles/myrole/templates/` altında saklamak best practice’tir.

**Dosya Adı**: `.j2` uzantısı kullanmak (örn. `index.html.j2`, `nginx.conf.j2`) en iyisidir. Hem siz hem Ansible, bunun bir Jinja2 şablonu olduğunu hemen anlarsınız.

### 7) Uygulama Senaryoları

* **Web Sunucu**: `index.html.j2` → Her makinede farklı hostname görüntülemek.
* **Nginx/Apache Config**: `nginx.conf.j2` → Farklı domain adları, portlar, SSL ayarları.
* **Redis/MySQL Config**: `redis.conf.j2`, `my.cnf.j2` → Her database sunucusuna farklı bellek, port ayarları, IP adresleri atama.
* **Network Ayar Dosyaları**: `resolv.conf.j2` ile dinamik DNS sunucuları ekleme.

Temel mantık hep aynı: Bir metin dosyası şablon (`.j2`), içinde değişkenli kısımlar. Ansible, `template:` modülüyle o değişkenleri doldurur ve dosyayı kopyalar.

### 8) Özet

1. **Ansible copy** modülü → Dosyayı **değiştirmeden** kopyalar.
2. **Ansible template** modülü → Dosyayı Jinja2 ile **işler** (değişkenleri yerleştirir), sonra kopyalar.
3. **.j2** uzantılı dosyalar → Jinja2 şablonları. İçinde `{{ varname }}` veya `{% ... %}` gibi ifade kullanabilirsiniz.
4. **inventory\_hostname** veya diğer Ansible değişkenleriyle her host’a özel içerik oluşturabilirsiniz.

<details>

<summary>Use Cases</summary>

Aşağıda, **Nginx ve MySQL** konfigürasyonlarını dinamikleştirmek için bir **gerçekçi kullanım senaryosu (use case)** örneği paylaşıyorum. İki farklı şablon (template) dosyası kullanacağız: biri **nginx.conf.j2** (web sunucusu), diğeri **my.cnf.j2** (MySQL veritabanı). Her sunucunun envanterde belirtilen değişkenlerine göre **farklı** ayarlar almasını sağlayacağız.

***

### 1) Klasör Yapısı (Örnek)

Basit bir Ansible projesi gibi düşünelim:

```
myproject/
├── inventory
├── playbook.yml
├── templates/
│   ├── nginx.conf.j2
│   └── my.cnf.j2
└── group_vars/
    ├── webservers.yml
    └── dbservers.yml
```

* `inventory`: Ansible envanter dosyası (hangi host web, hangi host db).
* `playbook.yml`: Burada görevleri yazacağız (template modülü vb.).
* `templates/`: Şablon dosyalarımız (`nginx.conf.j2`, `my.cnf.j2`).
* `group_vars/`: Web sunucuları (`webservers.yml`) ve DB sunucuları (`dbservers.yml`) için değişken dosyaları.

***

### 2) Envanter (inventory)

Örneğin, iki web sunucusu (`web1`, `web2`) ve iki DB sunucusu (`db1`, `db2`) tanımlayalım:

```ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20
db2 ansible_host=192.168.1.21
```

Böylece `webservers` grubuna dahil host’lar ile `dbservers` grubuna dahil host’ları ayırdık.

***

### 3) group\_vars Dosyaları

#### A) `group_vars/webservers.yml`

Tüm web sunucular için **ortak** veya **farklı** değerleri burada girebiliriz. Örneğin her web sunucunun domain adı, port, SSL ayarı vb.:

```yaml
# group_vars/webservers.yml

# Tüm web sunucuların dinleyeceği port:
nginx_port: 80

# Domain adları host spesifik olabilir, bu durumda envanterde tanımlayabilir ya da host_vars kullanabiliriz.
# Basit örnek: default bir domain verelim:
nginx_domain: "example.com"

# SSL kullanacaksak:
nginx_ssl_enabled: false
```

> Eğer `web1` ve `web2` farklı domain’ler kullanacaksa, `host_vars/web1.yml` ve `host_vars/web2.yml` dosyalarına `nginx_domain` tanımları yapabilirsiniz. Alternatif olarak inventory satırlarında da (`web1 ansible_host=... nginx_domain=web1.example.com`) eklenebilir.

#### B) `group_vars/dbservers.yml`

Veritabanı sunucuları için **farklı bellek (innodb-buffer-pool-size)**, IP adresi, port vb.:

```yaml
# group_vars/dbservers.yml

mysql_bind_address: "0.0.0.0"
mysql_port: 3306
mysql_innodb_buffer_pool_size: "512M"
```

> İki DB sunucusu yine farklı bellek ayarlarına sahip olacaksa, `host_vars/` üzerinden ayrıştırabilirsiniz (örneğin `db1.yml`’de 512M, `db2.yml`’de 1024M).

***

### 4) Template Dosyaları

#### A) `templates/nginx.conf.j2`

Basit bir Nginx konfigürasyon örneği:

```jinja
# templates/nginx.conf.j2
user nginx;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    server {
        listen {{ nginx_port }};

        {% if nginx_ssl_enabled %}
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/{{ inventory_hostname }}.crt;
        ssl_certificate_key /etc/nginx/ssl/{{ inventory_hostname }}.key;
        {% endif %}

        server_name {{ nginx_domain }};

        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
```

* **`{{ nginx_port }}`** → `group_vars/webservers.yml`’den değer okur (ör. 80).
* `{{ nginx_ssl_enabled }}` → true/false. True ise 443 SSL ayarları eklenir.
* `{{ nginx_domain }}` → Domain adı.

#### B) `templates/my.cnf.j2`

MySQL konfigürasyonu örneği:

```jinja
# templates/my.cnf.j2

[mysqld]
bind-address = {{ mysql_bind_address }}
port = {{ mysql_port }}
innodb_buffer_pool_size = {{ mysql_innodb_buffer_pool_size | default("256M") }}

# Diğer ayarlar
max_connections = 100
```

* `bind-address`, `port`, `innodb_buffer_pool_size` yine `group_vars/dbservers.yml` veya host bazlı ayarlardan gelir.
* `| default("256M")` ile tanımlanmamışsa 256M kullanılır.

***

### 5) Playbook (playbook.yml)

Aynı dosyada iki **play** tanımlayabiliriz: biri web sunucuları, diğeri DB sunucuları için. Örnek:

```yaml
---
- name: Configure Nginx on web servers
  hosts: webservers
  become: yes  # root yetkisiyle çalış
  tasks:
    - name: Deploy nginx.conf
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart nginx

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted

- name: Configure MySQL on DB servers
  hosts: dbservers
  become: yes
  tasks:
    - name: Deploy my.cnf
      template:
        src: my.cnf.j2
        dest: /etc/my.cnf
        owner: mysql
        group: mysql
        mode: '0644'
      notify: Restart mysql

    - name: Ensure MySQL is running
      service:
        name: mysqld
        state: started
        enabled: yes

  handlers:
    - name: Restart mysql
      service:
        name: mysqld
        state: restarted
```

**Açıklama**:

1. **İlk Play**
   * Hedef: `webservers` grubu (web1, web2).
   * `template` modülü ile `nginx.conf.j2` dosyasını `/etc/nginx/nginx.conf` olarak kopyalar.
   * Değişkenler: `nginx_port`, `nginx_domain`, `nginx_ssl_enabled` vs.
   * Handler: `Restart nginx` (konfig dosyası değişince Nginx yeniden başlasın).
2. **İkinci Play**
   * Hedef: `dbservers` grubu (db1, db2).
   * `template` modülü ile `my.cnf.j2` dosyasını `/etc/my.cnf` yoluna yerleştirir.
   * Değişkenler: `mysql_bind_address`, `mysql_port`, `mysql_innodb_buffer_pool_size` vs.
   * Handler: `Restart mysql`.

***

### 6) Çalıştırma ve Sonuç

Envanteriniz ve vars dosyalarınız hazır, `playbook.yml` de bu şekilde tanımlı. Komutu verin:

```bash
ansible-playbook -i inventory playbook.yml
```

* **Web Sunucularda**:
  * `/etc/nginx/nginx.conf` dosyası, `nginx.conf.j2` şablonu baz alınarak host’a özel `nginx_domain`, `nginx_port` vb. değerlerle oluşturulur.
  * Nginx restart edilir.
* **DB Sunucularda**:
  * `/etc/my.cnf` dosyası, `my.cnf.j2` şablonundan host’a özel `mysql_bind_address`, `mysql_innodb_buffer_pool_size` ile üretilir.
  * MySQL (mysqld) restart edilir.

Her makine, kendi konfigürasyonuna sahip olur.

***

### 7) Özet

1. **Vars (group\_vars/host\_vars)** üzerinden her rol veya host grubuna özgü değişkenler atıyoruz (domain adı, port, bellek, vb.).
2. **Template (.j2)** dosyalarında `{{ variable }}` veya Jinja2 koşulları/döngüleriyle dinamik kısımları tanımlıyoruz.
3. **Playbook** içinde `template` modülüyle bu şablonları hedef sisteme kopyalıyoruz.
4. Servis restart gibi adımlarla değişiklikler aktif hale geliyor.

Bu yaklaşım sayesinde tek bir şablonla **farklı** sunuculara **farklı** değerler atayabilir, yönetimsel süreci oldukça basitleştirirsiniz.

</details>
