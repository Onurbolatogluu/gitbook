---
icon: shield-cross
---

# Introduction to Templating

### 1. Templating Nedir?

Templating (şablonlama), “içinde **değişken** yerler olan bir iskelet dosya” ile “bu değişkenlere atanacak **gerçek değerleri**” birleştirerek, istenen **tam metin** (ör. bir konfig dosyası, HTML sayfası, e-posta vs.) oluşturma işidir.

**Örnek**: CEO’nun herkese gönderdiği davetiye e-postası bir şablondur (`Hi {{ employee_name }}...`). E-posta gönderirken `employee_name` yerine “John”, “Mary” gibi farklı değerler konur.

### 2. Jinja2 Nedir, Ansible’da Nasıl Kullanılır?

* **Jinja2**: Python tabanlı bir templating motorudur. Ansible, şablon dosyalarını (`.j2` uzantılı) Jinja2 ile yorumlayarak değişkenleri yerleştirir.
* Ansible Playbook’ta “template:” modülü ile Jinja2 şablonunu **uzak sunucuya** render edip kopyalayabilirsiniz. Bu sayede “dinamik” konfig dosyaları veya scriptler oluşturabilirsiniz.

### 3. Basit Değişken Yerleştirme

```
# my_template.j2 (örnek şablon)
The name is {{ my_name }}!
```

Playbook’ta:

```yaml
- name: Simple Templating
  hosts: localhost
  vars:
    my_name: "Bond"
  tasks:
    - name: Render a template
      template:
        src: my_template.j2
        dest: /tmp/output.txt
```

1. **`my_template.j2`** içinde `{{ my_name }}` kullandık.
2. Vars (`my_name: "Bond"`) Ansible tarafından şablona yerleştirilir.
3. Sonuç dosyada: `The name is Bond!`

***

### 4. Filtreler ve Dönüşümler

Jinja2’de `|` (pipe) ile çeşitli filtreler kullanabilirsiniz.

1.  **Metin Dönüşümleri**

    ```
    The name is {{ my_name | upper }} 
    ```

    * `my_name` değerini **büyük harfe** çevirir (`"BOND"`).
    * Ayrıca `lower`, `title`, `replace("Bond","Bourne")` gibi filtreler de var.
2.  **Varsayılan Değer**

    ```
    Hello {{ first_name | default("James") }} {{ last_name }}
    ```

    * Eğer `first_name` tanımlı değilse `"James"` kullanır, aksi halde `first_name`’in değerini alır.
3.  **Liste İşlemleri**

    ```
    Minimum: {{ nums | min }}
    Maximum: {{ nums | max }}
    Unique:  {{ nums | unique }}
    ```

    * `nums` bir liste olduğunda, `min`, `max`, `unique` gibi filtrelerle listeyi işleyebilirsiniz.



### 5. Koşullar ve Döngüler

Jinja2, “\{% ... %\}” blok sözdizimiyle **if**, **for** gibi yapıların kullanımını sağlar.

1.  **Döngü (for)**

    ```django
    {% for item in my_list %}
    - {{ item }}
    {% endfor %}
    ```

    * Bu, `my_list` içindeki elemanları tek tek yazar.
2.  **Koşul (if)**

    ```django
    {% if number == 2 %}
    This is 2
    {% else %}
    Not 2
    {% endif %}
    ```

    * “number” değişkeni 2 ise “This is 2” yazar, değilse “Not 2”.

**Örnek Geniş** – Bir şablon:

```django
# config_file.j2
[mysqld]
datadir={{ datadir }}
{% if innodb == true %}
innodb_buffer_pool_size=1G
{% endif %}
```

Varsayılan: `innodb=true, datadir="/var/lib/mysql"`

*   Render sonunda:

    ```
    [mysqld]
    datadir=/var/lib/mysql
    innodb_buffer_pool_size=1G
    ```

***



Bu senaryoda, bir veritabanı sunucusunda (`db_servers`) MySQL konfigürasyon dosyasını Jinja2 şablonuyla nasıl otomatik oluşturabileceğimizi göreceksiniz. Ardından dosya güncellenirse, MySQL servisini yeniden başlatacak bir handler tetikliyoruz.

### 1. Dizini ve Dosyaları Hazırlama

Projenizde (örneğin `my-ansible-project/` klasörünüzde) bir **playbook** (örn. `playbook.yml`) ve bir **templates** klasörü oluşturabilirsiniz:

```
my-ansible-project/
├── playbook.yml
└── templates/
    └── config_file.j2
```

* `templates/config_file.j2`: Jinja2 şablon dosyası (içinde değişken kullanabilirsiniz).
* `playbook.yml`: Ansible Playbook, hangi hostlara ve nasıl uygulayacağınızı belirtiyorsunuz.

***

### 2. Jinja2 Şablonu (config\_file.j2)

Örnek bir MySQL konfig dosyası şablonu:

```jinja
[mysqld]
datadir={{ datadir }}

{% if innodb %}
innodb_buffer_pool_size=1G
{% endif %}

user=mysql
port=3306

# Bu sadece bir örnek, ihtiyacınıza göre satırlar ekleyebilirsiniz.
```

Açıklama:

* `{{ datadir }}` ve `{{ innodb }}` gibi değerler, **Playbook veya vars** içindeki değişkenlere göre doldurulacak.
* `innodb` değişkeni **true** ise `innodb_buffer_pool_size=1G` satırı yazılacak, yoksa atlanacak.

***

### 3. Playbook Yapısı (playbook.yml)

Örnek bir playbook:

```yaml
- name: Templating Demo
  hosts: db_servers
  gather_facts: no
  vars:
    innodb: true
    datadir: "/var/lib/mysql"

  tasks:
    - name: Apply MySQL config from template
      template:
        src: config_file.j2
        dest: /etc/mysql/my.cnf
      notify: Restart MySQL

  handlers:
    - name: Restart MySQL
      service:
        name: mysql
        state: restarted
```

1. **hosts: db\_servers**
   * `db_servers` grubunda tanımlı sunuculara bu işlemleri uyguluyoruz.
2. **vars**
   * `innodb` ve `datadir` değişkenlerini tanımlıyoruz. (Örneğin `innodb: true` dedik, bu şablonun if bloğunu etkinleştirecek.)
3. **tasks**
   * **template** modülü:
     * `src: config_file.j2`: Ansible controller (bu makine) üzerindeki `templates/` klasöründeki Jinja2 şablon dosyamız.
     * `dest: /etc/mysql/my.cnf`: Uzaktaki hedeften (db sunucusu) nereye kopyalanacağını gösterir.
   * `notify: Restart MySQL` => Bu görev bir değişiklik yaptığında (“changed”), aşağıdaki `handlers`bölümündeki “Restart MySQL” tetiklenecek.
4. **handlers**
   * `Restart MySQL`: Bir “service” modülü ile MySQL servisini yeniden başlatır.
   * Sadece **template** görevinde gerçekten dosya içeriği değiştiyse (“changed” olarak raporlanırsa) handler tetiklenir.

***

### 4. Çalıştırma ve Mantık

```bash
ansible-playbook playbook.yml
```

* Ansible, `config_file.j2` dosyasını okumak için **vars** (`innodb: true`, `datadir: "/var/lib/mysql"`) değerlerini şablona uygular.
* Oluşan **render edilmiş** metin `/etc/mysql/my.cnf` olarak db sunucusuna kopyalanır.
  *   Örnek final içerik:

      ```
      [mysqld]
      datadir=/var/lib/mysql
      innodb_buffer_pool_size=1G
      user=mysql
      port=3306
      ```
* Eğer dosya içerik olarak önceki hâlinden farklıysa (gerçek bir değişiklik olduysa), **“changed”** olarak işaretlenir, dolayısıyla `notify: Restart MySQL` devreye girer.
* Handler, MySQL servisini restarted duruma getirir.

**Sistematik Faydası**:

* Şablonun içinde Jinja2 ile “if/for/default vs.” yazabilir, parametrelere göre konfig dosyalarını esnekçe oluşturabilirsiniz.
* “dosya değişti mi?” kontrolleri ve “servis restart” tetikleme mantığı size ek kod yazmadan, Ansible’ın built-in “change detection” özelliğiyle otomatikleşir.

***

### Özet

Bu örnekte, **Ansible template** modülü ile bir MySQL konfig dosyasını Jinja2 şablonundan oluşturup hedef sunucuya kopyaladık. Varsayılan ayarlarla ya da playbook içi vars’larla (`innodb`, `datadir` gibi) **dinamik** bir dosya yaratıp, eğer bir değişiklik olduysa MySQL’i **otomatik** yeniden başlattık.

Özellikle **koşullar** (if) ve **değişkenler** (`{{ ... }}`) sayesinde tek bir `.j2` şablon dosyasını birçok farklı senaryoda kullanabilirsiniz. Bu, büyük yapılandırma yönetiminde kod tekrarını azaltır ve yönettiğiniz sistemlerde **konsistensi** ve **kolay bakımı** sağlar.

