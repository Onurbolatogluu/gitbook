---
icon: warehouse
---

# Registering Variables and Variable Precedence

Bu konular, Ansible’da değişkenlerin nasıl tanımlandığını, aynı değişkenin farklı yerlerde tanımlanması hâlinde hangi değerinin geçerli olacağını ve bir görev (task) sonucunu daha sonra kullanmak üzere nasıl saklayabileceğinizi anlatır.

### 1. Variable Precedence (Değişken Önceliği)

#### 1.1 Aynı Değişkenin Farklı Yerlerde Tanımlanması

Ansible’da aynı isimli bir değişkeni (örneğin `dns_server`) hem **envanter dosyasında** (inventory), hem **grup değişkeni**(group\_vars), hem **host değişkeni** (host\_vars), hatta **playbook içinde** veya **komut satırında** (`--extra-vars`) tanımlayabilirsiniz. Peki Ansible hangisini kullanacak?

* **En Düşük Öncelik**: Role defaults (rollerde `defaults/main.yml`)
* **Grup Değişkenleri**: `[web_servers:vars]` gibi
* **Host Değişkenleri**: `web2 dns_server=10.5.5.4` gibi
* **Playbook İçinde Tanımlanan Vars**: `vars:` altında
* **Komut Satırı (Extra Vars)**: `ansible-playbook ... --extra-vars "dns_server=10.5.5.6"`

**Kural**: Extra Vars en yüksek önceliğe sahiptir. Yani, **aynı isimli değişken** birden çok yerde tanımlanırsa, **daha yüksek öncelikli tanım** var olan diğer tanımları geçersiz kılar.

**Örnek Senaryo**

* **Grup Değişkeni**: `[web_servers:vars] dns_server=10.5.5.3`
* **Host Değişkeni**: `web2 ansible_host=172.20.1.101 dns_server=10.5.5.4`
  * Bu durumda, `web2` makinesi için **10.5.5.4** değeri geçerli olur (host vars, grup vars’ı ezer).
* **Playbook İçinde**: `vars: dns_server=10.5.5.5`
  * Bu tanım ise tüm envanter tanımlarının üzerindedir, yani `dns_server=10.5.5.5` en geçerli değer olur.
* **Komut Satırı**: `ansible-playbook playbook.yml --extra-vars "dns_server=10.5.5.6"`
  * Her şeyi geçersiz kılar, en yüksek öncelik olarak 10.5.5.6 kullanılacaktır.

### 2. Registering Variables (Görev Çıktılarını Kaydetme)

Bazen bir görevin (task) çıktısını **daha sonra** kullanmak istersiniz. Örneğin bir komutun döndürdüğü sonucu, ya da bir modülün verdiği veriyi başka bir adımda işlemek için saklamanız gerekebilir. Bu durumda **`register`** direktifi kullanılır.

#### 2.1 Basit Örnek

```yaml
- name: Check /etc/hosts file
  hosts: all
  tasks:
    - name: Read /etc/hosts
      shell: cat /etc/hosts
      register: result

    - name: Print the result
      debug:
        var: result
```

1. `shell: cat /etc/hosts` komutu çalışır.
2. `register: result` ifadesiyle, komutun çıktısı `result` adlı değişkende saklanır.
3. `debug: var: result` satırı, `result` değişkenindeki tüm bilgiyi ekrana basar (örneğin `stdout`, `rc`, `start`, `end`, vb.).

**Sık Karşılaşılan Alanlar (Shell Modülü Örneğinde)**

* `stdout`: Komutun standart çıktısı (örneğimizde `/etc/hosts` içeriği)
* `rc`: Return code (0 → Başarılı, 0 dışı genelde hata)
* `stderr`: Standart hata akışı (varsa)
* `stdout_lines`: `stdout`’u satır bazında liste hâlinde tutar

#### 2.2 Yalnızca İstediğiniz Bilgiyi Yazdırma

Eğer tüm çıktıyı değil, sadece belli bir kısım görmek istiyorsanız:

```yaml
- debug:
    var: result.stdout
```

Bu şekilde, sadece komutun çıktısını ekrana basarsınız (örneğin `/etc/hosts` içeriğini).

#### 2.3 Kapsam (Scope) ve Kullanım Süresi

* **Host Seviyesi**: `register` ile kaydedilen değişken, sadece **o host** üzerinde geçerlidir. Farklı bir host için aynı görev çalıştığında, kendi `result` değeri olur.
* **Playboyunca Geçerli**: Kayıtlı değişkeni, o play’in devamındaki adımlarda kullanabilirsiniz. İhtiyaç varsa aynı host için **bir sonraki play** içinde de kullanılabilir (ayrı bir play’de “hosts:” aynı host ise).
* **Birden Fazla Host**: “hosts: all” derseniz, her host için “result” farklı değer tutar. Onlara `hostvars[‘hostname’].result` şeklinde de erişmek mümkün.

### 3. Ek Bilgi: Verbose Mod (-v)

Görevlerin çıktısını incelemek istiyor ama playbook’u düzenlemek istemiyorsanız:

```bash
ansible-playbook playbook.yml -v
```

* `-v` (veya `-vvv` gibi daha fazlası) çıktının ayrıntılı (verbose) görünmesini sağlar.
* Görevlerin komut, stdout, rc, vb. bilgilerini konsolda görürsünüz.

***

### Complex ENV Variable Used:

### Proje Klasör Yapısı

Örneğin şöyle bir dizin yapısı oluşturalım:

```
myproject/
├── inventory.ini
├── group_vars/
│   └── web_servers.yml
├── host_vars/
│   └── web2.yml
├── roles/
│   └── myrole/
│       └── defaults/
│           └── main.yml
└── myplaybook.yml
```

* **`inventory.ini`**: Burada `web1`, `web2` gibi host’ları tanımlayacağız. Gerekirse host bazında değişken de ekleyebiliriz.
* **`group_vars/web_servers.yml`**: `web_servers` adlı gruba ait ortak değişkenler.
* **`host_vars/web2.yml`**: `web2` adlı host’a özgü değişkenler.
* **`roles/myrole/defaults/main.yml`**: Bir rolde tanımlanan **defaults** (en düşük öncelik).
* **`myplaybook.yml`**: Playbook içinde `vars:` de tanımlayabiliriz. Ayrıca komut satırında `--extra-vars` kullanıldığında en yüksek öncelik orada olur.

Bu örnek, en azından “role defaults → group vars → host vars → play vars → extra vars” sıralamasını kodla göstermemizi sağlayacak.

***

### 1. Role Defaults (En Düşük Öncelik)

**`roles/myrole/defaults/main.yml`**:

```yaml
# En düşük öncelikli ayar
dns_server: "10.5.5.1"  # Varsayılan DNS
```

Bu dosya, “myrole” adlı rolün “defaults” dizininde bulunuyor. Ansible’da role defaults, envanterde veya playbook’ta tanımlanmış benzer değişkenlere göre **en düşük önceliğe** sahiptir.

***

### 2. Inventory Dosyası (Group, Host Tanımları)

**`inventory.ini`**:

```ini
[web_servers]
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101
```

* `[web_servers]` adında bir grup var.
* `web1` ve `web2` adında iki host bu gruba dâhil.

Şimdilik burada doğrudan `dns_server` tanımlamadım, onu `group_vars` veya `host_vars` dosyalarında göstereceğim.

***

### 3. Group Vars (Grup Değişkenleri)

**`group_vars/web_servers.yml`**:

```yaml
# Bu grup değişkeni, tüm web_servers hostlarına geçerli olur
dns_server: "10.5.5.3"
```

* `[web_servers]` grubuna ait varsayılan `dns_server` değeri `10.5.5.3`.
* Bu, eğer aynı değişken host düzeyinde veya playbook’ta yeniden tanımlanmadıysa, “web1” ve “web2” için geçerli olacaktır.

***

### 4. Host Vars (Belirli Bir Host’a Özel)

**`host_vars/web2.yml`**:

```yaml
# Yalnızca web2'ye özel tanım
dns_server: "10.5.5.4"
```

* Bu tanım, `web2` için **group\_vars**’taki `dns_server=10.5.5.3` değerini geçersiz kılar.
* Artık `web2` için `dns_server=10.5.5.4` kullanılırken, `web1` hâlen 10.5.5.3 kullanmaya devam eder.

***

### 5. Playbook İçinde Değişken (Play Vars)

**`myplaybook.yml`**:

```yaml
- name: Demo of variable precedence
  hosts: web_servers
  vars:
    dns_server: "10.5.5.5"  # Playbook düzeyinde tanımlanan dns_server
  roles:
    - myrole
  tasks:
    - name: Show the final dns_server
      debug:
        msg: "DNS Server (final) is {{ dns_server }}"
```

Bu playbook’ta:

* `vars:` bölümünde `dns_server: "10.5.5.5"` olarak tanımladık.
* Bu tanım, role defaults (10.5.5.1), group vars (10.5.5.3) ve host vars (10.5.5.4) üzerinde **daha yüksek önceliğe**sahiptir.
* Dolayısıyla, `web1` ve `web2` dâhil tüm host’lar için “10.5.5.5” değeri geçerli olacaktır. (Eğer playbook’ta o host’a özel bir override yapılmadıysa.)

***

### 6. Komut Satırında `--extra-vars` (En Yüksek Öncelik)

Eğer bu playbook’u şu şekilde çalıştırırsak:

```bash
ansible-playbook -i inventory.ini myplaybook.yml --extra-vars "dns_server=10.5.5.6"
```

* `--extra-vars "dns_server=10.5.5.6"` parametresi, **her şeyin önünde** gelir.
* Böylece final değer **10.5.5.6** olur ve playbook içindeki `10.5.5.5` dahil, diğer tüm tanımlar geçersiz kalır.

***

### Sonuç (Özet)

* **Role Defaults** (`roles/myrole/defaults/main.yml`) → `dns_server = 10.5.5.1`
* **Group Vars** (`group_vars/web_servers.yml`) → `dns_server = 10.5.5.3`
* **Host Vars** (`host_vars/web2.yml`) → `dns_server = 10.5.5.4` (sadece `web2` için)
* **Playbook Vars** (`myplaybook.yml`) → `dns_server = 10.5.5.5` (tüm web\_servers host’larına hükmeder)
* **Extra Vars** (`--extra-vars "dns_server=10.5.5.6"`) → En üst düzeyde, her şeyi ezer

**Çalışma Anında:**

1. **Role Defaults** en düşük öncelikte kalır.
2. **Group Vars** (10.5.5.3) bunu geçersiz kılar.
3. **Host Vars** (10.5.5.4) “web2” için group vars’ın üstüne yazar (web1 yine 10.5.5.3).
4. **Playbook Vars** (10.5.5.5) hepsini geçersiz kılar, tüm host’lar 10.5.5.5 olur.
5. **Extra Vars** parametresi verilirse (10.5.5.6), her şeyi ezer, tüm host’lar 10.5.5.6 olur.
