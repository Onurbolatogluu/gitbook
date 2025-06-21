---
icon: puzzle
---

# Jinja2 Basics

### 1) Templating (Şablonlama) Nedir?

* **Şablonlama**, bir “kalıp belge” (template) ile “değişken verileri” birleştirerek özel çıktılar üretmenizi sağlar.
  * Örnek: CEO, bütün çalışanlara aynı davet mektubunu yollar. Tek fark, mektuptaki isimler ve aile üyelerinin isimleridir.
  * Mektubun ana gövdesi değişmez (şablon), fakat “isim” veya “davet metni” gibi kısımlar değişkendir.
* Bilgisayar dünyasında: Web sayfalarını dinamik oluşturmak (HTML + veritabanı verileri) veya Ansible’da sunucu ayar dosyalarını (konfigürasyon) dinamikleştirmek için kullanılır.

**Basit Fikir**:

1. Bir şablon hazırlarsınız.
2. Ansible’a “Şu değişkenleri kullanarak bu şablonu doldur.” dersiniz.
3. Sonuç, kullanıma hazır özelleştirilmiş bir dosya/metin olur.

### 2) Jinja2 Nedir?

* **Jinja2**, Python’da yazılmış popüler bir şablonlama motorudur.
* Ansible, Jinja2’yu kullanarak `{{ variable }}` gibi ifadeleri yorumlar.
* Jinja2, “metin kalıbını” alır ve içindeki değişkenleri sizin sağladığınız değerlerle değiştirerek çıktı oluşturur.

### 3) Temel Kullanım: Değişken Ekleme (Interpolation)

**En basit** Jinja2 örneği:

```jinja
My name is {{ my_name }}.
```

* `{{ my_name }}`: `my_name` adında bir değişken var, Jinja2 onu burada metne yerleştirir.
* Örnek değer: `my_name = "Bond"` ise çıktıda `My name is Bond.` görürsünüz.

### 4) Filtreler (Filters)

Bazen değişkeni direkt kullanmak yerine, özel işlemler yapmak isteyebilirsiniz. Jinja2’de “filtre”ler (`|` sembolüyle) kullanılır:

1.  **Küçük/Büyük Harf Dönüşümü**:

    ```jinja
    {{ my_name|upper }}  # hepsini BÜYÜK harfe çevirir
    {{ my_name|lower }}  # hepsini küçük harfe çevirir
    {{ my_name|title }}  # Baş harfleri büyük (Title Case)
    ```
2.  **replace** (Metin değişimi):

    ```jinja
    {{ my_name|replace("Bond", "Bourne") }}
    ```

    * Bu, “Bond” yazısını “Bourne” ile değiştirir.
3.  **default** (Varsayılan Değer):

    ```jinja
    {{ first_name|default("Unknown") }}
    ```

    * `first_name` tanımlı değilse “Unknown” yazısını kullanır. Tanımlıysa kendi değerini alır.
4. **Diziler/Listelerle İlgili Filtreler**:
   *   `min`, `max`, `unique`, `union`, `intersect`, `random` gibi

       ```jinja
       {{ numbers|min }}
       {{ numbers|max }}
       {{ numbers|unique }}
       {{ numbers1|union(numbers2) }}
       ```
   *   `join` (listeyi birleştirip tek string yapmak):

       ```jinja
       {{ mylist|join(", ") }}
       ```

       * Örnek: `["apple", "banana", "cherry"]` → `"apple, banana, cherry"`

### 5) Koşullar (Conditionals) ve Döngüler (Loops)

Jinja2, tıpkı bir mini programlama dili gibi _if_, _for_ gibi yapıları da destekler. Burada süslü parantez yerine `%` sembolleri kullanılır:

#### a) For Loop

```jinja
{% for i in range(1, 4) %}
Number: {{ i }}
{% endfor %}
```

* `range(1, 4)` → `[1,2,3]` üretir.
* Her döngüde `i`’yi ekrana basar.

#### b) If Statement

```jinja
{% if i == 2 %}
  Number is TWO
{% endif %}
```

* Koşul doğruysa ilgili metin/komut çalışır.

> **Dikkat**: Döngüler ve koşullarda `{{ }}` değil `{% %}` kullanırız. Çünkü `{{ }}` tek satırlık değişken göstermek içindir, `{% %}` ise “yapı (block)” tanımlar.

### 6) Ansible’da Jinja2 Nasıl Kullanılır?

* **Şablon dosyası** (`.j2` uzantısıyla) oluşturursunuz.
*   Ansible Playbook’ta `template` modülüyle bu dosyayı hedef sisteme kopyalarken değişken değerlerini yerleştirir:

    ```yaml
    - name: Create my config from template
      template:
        src: myconfig.j2
        dest: /etc/myconfig.conf
    ```
* `myconfig.j2` içinde `{{ my_var }}` veya döngüler, koşullar kullanabilirsiniz.
* Playbook’taki veya vars dosyalarındaki değişkenler, şablonda otomatik olarak yerleştirilir.

**Örnek Template** (`myconfig.j2`):

```jinja
[mysqld]
datadir = {{ datadir|default("/var/lib/mysql") }}
buffer_pool = {{ buffer_pool|default("256MB") }}

{% if replication_enabled %}
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
{% endif %}
```

* Eğer `replication_enabled` değişkeni **true** ise, `server-id` satırı eklenir; yoksa eklenmez.

### 7) Özet

1. **Templating**: Metin dosyanızı (ör. konfigürasyon, HTML) değişkenlerle doldurmak.
2. **Jinja2**, Ansible’ın kullandığı güçlü şablonlama motoru:
   * `{{ }}` → Değişken ekleme
   * `| filtre` → Değer dönüştürme (upper, lower, replace vb.)
   * `{% if %}` / `{% for %}` → Koşullar ve döngüler
3. **Ansible’da template modülü** ile `.j2` dosyanızı hedefe gönderirken değişkenler otomatik yerleştirilir.

Jinja2 sayesinde **tekrar tekrar** aynı yapılandırma dosyalarını farklı ortamlarda (farklı IP adresi, farklı kullanıcı adı, vs.) kolayca üretebilir, manuel düzenleme ihtiyacını ortadan kaldırırsınız. Bu, Ansible’ın otomasyon gücünü önemli ölçüde artırır.

