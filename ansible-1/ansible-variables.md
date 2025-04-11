---
icon: square-root-variable
---

# Ansible Variables

### 1. Değişken (Variable) Nedir?

Ansible’da değişkenler, her sunucuda veya her görevde _farklı_ olması gereken değerleri saklar. Aynı Playbook ile 100 farklı sunucuda patch işlemi yapacaksanız, bu 100 sunucunun host ismi, kullanıcı adı, parola veya IP adresi gibi değişen bilgilerini **değişkenler** aracılığıyla yönetirsiniz. Playbook’u sadece bir kere yazar, değişen kısımları değişken tanımlarıyla (envanter veya var dosyaları) kontrol edersiniz.

***

### 2. Inventory Dosyasında Değişken Kullanımı

Daha önce **inventory** (envanter) dosyasında `ansible_host`, `ansible_connection`, `ansible_ssh_pass` vb. değişkenler gördük:

**inventory.ini**:

```ini
[web]
Web1 ansible_host=server1.company.com ansible_connection=ssh ansible_ssh_pass=P@ssW http_port=8081 snmp_port=161-162

[db]
db ansible_host=server2.company.com ansible_connection=winrm ansible_ssh_pass=P@s
```

Burada `Web1` için:

* `ansible_host=server1.company.com`
* `ansible_connection=ssh`
* `ansible_ssh_pass=P@ssW`
* `http_port=8081`
* `snmp_port=161-162`\
  gibi **beş** değişken tanımlamış olduk.

**Not:** Sadece `Web1` satırında `http_port` ve `snmp_port` tanımladığımız için, bu değişkenler **yalnızca** `Web1` adlı host için geçerli olur.

**use\_inventory\_vars.yml**:

```yaml
- name: Demo Playbook using inventory variables
  hosts: web  # "web" grubunu hedefliyoruz
  gather_facts: no  # Örnek olarak hız için kapattık, isteğe bağlı

  tasks:
    - name: Show the http_port and snmp_port from inventory
      debug:
        msg: >
          HTTP Port is {{ http_port }}
          SNMP Port is {{ snmp_port }}

    - name: Use the http_port in a mock firewall task
      firewalld:
        port: "{{ http_port }}/tcp"
        permanent: true
        state: enabled
```

#### Açıklamalar

1. **`hosts: web`**
   * Bu playbook, envanter dosyasında `[web]` grubunda tanımlı olan tüm hostlara uygulanır (yani `Web1`).
2. **Değişken Kullanımı: `{{ http_port }}`**
   * `{{ }}` (Jinja2 templating) ile `inventory.ini` dosyasında “Web1” satırında tanımlanmış `http_port` ve `snmp_port` değerlerini çağırıyoruz.
   * Örneğin `debug` modülü, terminalde “HTTP Port is 8081, SNMP Port is 161-162” şeklinde bir mesaj basar.
3. **`firewalld:` Örneği**
   * Burada “`port: "{{ http_port }}/tcp"`” diyerek, envanterden gelen `http_port` değerini (8081) kullanıyoruz.
   * Böylece portu sabit kodlamak yerine envanterde değiştirerek daha esnek yönetim sağlıyoruz.

***

### 3. Playbook’ta Değişken Tanımlama

Playbook içinde, **`vars`** anahtar kelimesiyle değişken tanımlayabilirsiniz:

```yaml
- name: Add DNS server to resolv.conf
  hosts: localhost
  vars:
    dns_server: 10.1.250.10  # Burada değişken tanımladık
  tasks:
    - lineinfile:
        path: /etc/resolv.conf
        line: "nameserver {{ dns_server }}"  # Değişkeni kullanıyoruz
```

1. **Tanımlama**: `vars:` altına `anahtar: değer` şeklinde yazarız.
2. **Kullanım**: `{{ ... }}` (Jinja2 templating) ile değişkenin değerini yerleştiririz.
3. **Neden Faydalı?** Değeri değiştirmek istediğimizde sadece `vars:` bölümünü güncelleriz; Playbook’un kod kısmına dokunmamız gerekmez.

***

### 4. Daha Büyük Senaryolar: Varsayılan Değerleri Ayırma

Özellikle port, IP blokları gibi makineye özgü değerleri envantere yazabilirsiniz. Büyük projelerde, her host veya her grup için değişkenler ayrı YAML dosyalarında saklanır (host\_vars, group\_vars dizinlerinde).

Proje dizininiz şu şekilde olabilir:

```
.
├── inventory.ini
├── host_vars
│   └── web1.yml
└── playbook.yml
```

* `inventory.ini`: Sunucu(lar)ı tanımladığınız envanter dosyası.
* `host_vars/web1.yml`: `web1` adlı host’a özel değişkenlerin saklandığı dosya.
* `playbook.yml`: Playbook’unuz.

**Önemli Nokta**: `host_vars` altındaki YAML dosyasının adı, **envanterdeki host ismi** ile **birebir aynı** olmalıdır (`web1.yml`↔ `web1`).

#### Inventory (inventory.ini)

```ini
[web]
web1 ansible_host=server1.company.com ansible_connection=ssh
```

* `[web]` grubuna **`web1`** adlı host dahil ediyoruz.
* Host’a dair temel bağlantı bilgilerini (IP, SSH vb.) burada tutuyoruz.

#### Host-Specific Vars (host\_vars/web1.yml)

```yaml
http_port: 8081
snmp_port: 161-162
inter_ip_range: 192.0.2.0
```

* `web1` host’una özgü değişkenleri burada tanımladık (port bilgileri, IP range vb.).
* Playbook, `hosts: web` (veya doğrudan `hosts: web1`) dendiğinde bu dosyadaki değişkenler otomatik yüklenecek.

#### Playbook (playbook.yml)

```yaml
- name: Demo using host_vars
  hosts: web   # "web" grubuna ait tüm host’lar (web1) hedeflenir
  gather_facts: no

  tasks:
    - name: Display Host Vars
      debug:
        msg:
          - "HTTP Port is {{ http_port }}"
          - "SNMP Port is {{ snmp_port }}"
          - "Internal Range: {{ inter_ip_range }}"
```

#### Nasıl Çalışıyor?

1. **Ansible**, `hosts: web` dediğimizde önce `inventory.ini` içindeki `web` grubunu bulur.
2. `web` grubunda `web1` adlı host olduğu için, Ansible “`web1`” ismini **host\_vars** dizinine bakarak arar (`host_vars/web1.yml`).
3. `web1.yml` dosyasındaki değişkenleri (ör. `http_port`) yükler.
4. `{{ http_port }}` gibi ifadelere denk geldiğinde bu değerleri kullanır.

#### Playbook’u Çalıştırma

```bash
ansible-playbook -i inventory.ini playbook.yml
```

* `-i inventory.ini`: Varsayılan envanter `/etc/ansible/hosts` değil, `inventory.ini` dosyamızı kullandığımızı belirtir.
* Ansible, `web1.yml` içindeki değişkenlerle `playbook.yml` içindeki `{{ http_port }}` vb. yerleri doldurur ve işlemleri uygular.

#### Sonuç

* **Inventory’de** yalnızca host isimlerini tanımlıyor, temel bağlantı ayarlarını tutuyoruz (IP, SSH vs.).
* **host\_vars** (veya `group_vars`) klasörlerinde, her host (veya grup) için ayrıntılı değişkenleri saklıyoruz.
* **Playbook**, `hosts:` satırında eşleşen hostların **host\_vars** dosyalarındaki değişkenleri otomatik olarak yüklüyor. Bu sayede değişken yönetimi daha düzenli ve ölçeklenebilir hâle geliyor.

Yukarıda host\_vars kullanarak, bir host için değişkenleri kullanmayı gördük. Aynı şekilde, aşağıda, **group\_vars** klasörünü kullanarak bir grup için ortak değişkenleri nasıl tanımlayabileceğinize dair basit bir örnek gösteriyorum. Bu yapı sayesinde, **aynı gruba ait** tüm makineler, **ortak** değişkenleri otomatik olarak yükleyebilir.

#### Klasör Yapısı

Proje dizininiz şu şekilde olabilir:

```
.
├── inventory.ini
├── group_vars
│   └── web.yml
└── playbook.yml
```

* **`inventory.ini`**: Sunucu(lar)ı ve grupları tanımladığınız envanter dosyası.
* **`group_vars/web.yml`**: `web` adlı grup için değişkenlerin saklandığı dosya.
* **`playbook.yml`**: Playbook’unuz.

**Önemli Nokta**: `group_vars` altındaki YAML dosyasının adı, **envanterdeki grup ismi** ile **birebir aynı** olmalıdır (`web.yml` ↔ `[web]` grubu).

#### Inventory (inventory.ini)

```ini
[web]
web1 ansible_host=web01.example.com ansible_connection=ssh
web2 ansible_host=web02.example.com ansible_connection=ssh

[db]
db1 ansible_host=db01.example.com ansible_connection=ssh
```

* **`[web]`** grubunda `web1` ve `web2` adlı iki host var.
* **`[db]`** grubunda `db1` adlı tek host bulunuyor.

### Grup Değişkenleri (group\_vars/web.yml)

```yaml
http_port: 8081
snmp_port: 161-162
inter_ip_range: 192.0.2.0
```

* Bu dosya, **`web`** grubuna ait **ortak** değişkenleri içeriyor.
* `http_port`, `snmp_port` ve `inter_ip_range` bu gruba bağlı **tüm** makineler (`web1` ve `web2`) tarafından otomatik yüklenecek.

#### Playbook (playbook.yml)

```yaml
- name: Demo using group_vars
  hosts: web   # "web" grubuna ait tüm host’lar hedeflenir
  gather_facts: no

  tasks:
    - name: Display group_vars for the web group
      debug:
        msg:
          - "HTTP Port is {{ http_port }}"
          - "SNMP Port is {{ snmp_port }}"
          - "Internal Range: {{ inter_ip_range }}"
```

#### Nasıl Çalışır?

1. **`hosts: web`** dediğimizde Ansible, envanterde `[web]` grubundaki tüm hostları (web1, web2) hedefler.
2. Ansible aynı isimli `group_vars/web.yml` dosyasını otomatik olarak arayıp yükler.
3. Bu dosyadaki değişkenler (`http_port`, `snmp_port`, `inter_ip_range`), playbook içinde **her bir host** için geçerli olur.
4. `{{ http_port }}` gibi değişken çağrılarında bu değerler kullanılır.

#### Playbook’u Çalıştırma

```bash
ansible-playbook -i inventory.ini playbook.yml
```

* `-i inventory.ini`: Varsayılan dışındaki envanter dosyasını işaret eder.
* Ansible, `[web]` grubuna dahil olan hostlara gider ve `group_vars/web.yml` içindeki değişkenleri kullanır.

#### Sonuç

* **group\_vars**’ın avantajı, **gruplara** ait **ortak** değişkenleri tek bir yerde toplayarak bakım kolaylığı sağlamasıdır.
* Yeni bir makine **`web`** grubuna eklendiğinde, `group_vars/web.yml` dosyasındaki değişkenleri **otomatik olarak** devralır.
* Büyük ortamlarda, farklı gruplara (örneğin `[db]`, `[cache]`, `[loadbalancer]`) özel değişkenler için benzer şekilde `group_vars/db.yml`, `group_vars/cache.yml` gibi dosyalar oluşturulabilir.

Bu şekilde, **group\_vars** klasörü üzerinden her gruba ait değişkenleri toplu ve düzenli halde yönetebilirsiniz.

***

### 5. Değişkenleri Kullandığımız Tipik Örnekler

#### 5.1 DNS Eklemek

* Tek satırda “DNS IP” sabit yazmak yerine, `dns_server` değişkenini `vars:` veya envanterde tanımlarsınız.
* `lineinfile` modülü ile `nameserver {{ dns_server }}` satırını ekler veya değiştirirsiniz.

#### 5.2 Firewall Ayarları

Aşağıdaki örnekte, port değerlerini envanter/var dosyasından çekiyoruz ve Jinja2 ile yerleştiriyoruz:

```yaml
- name: Set Firewall Configurations
  hosts: web
  tasks:
    - firewalld:
        port: "{{ http_port }}/tcp"
        permanent: true
        state: disabled
    - firewalld:
        port: "{{ snmp_port }}/udp"
        permanent: true
        state: disabled
    - firewalld:
        source: "{{ inter_ip_range }}/24"
        zone: internal
        state: enabled
```

* `http_port`, `snmp_port`, `inter_ip_range` gibi değişkenler ya inventory’de ya da `web.yml` dosyasında tanımlanmıştır.
* Böylece `snmp_port` bir anda `161-162` yerine `10161-10162` olsaydı, sadece var dosyasını güncellemek yeter.

***

#### Özet

* **Envanter Dosyasında Değişken**: `alias ansible_host=... http_port=...` gibi yazılır, her host veya grup bazında bilgi tutar.
* **Playbook’ta `vars:`**: Daha küçük, hızlı tanımlamalar için idealdir.
* **Ayrı Dosyalar (host\_vars / group\_vars)**: Çok sayıda değişkeni düzenli saklamak için en iyi yöntem.
* **Kullanırken**: `{{ variable_name }}` (Jinja2).
* **Faydası**: Daha dinamik, esnek, tekrar kullanılabilir ve bakımı kolay Ansible kodu elde edersiniz.

Bu temelleri anladıktan sonra, **vars\_files**, **group\_vars/host\_vars** gibi yaklaşımlarla çok daha düzenli ve ölçeklenebilir playbook’lar oluşturabilirsiniz.

