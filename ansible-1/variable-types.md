---
icon: text-size
---

# Variable Types

### 1. String (Metin) Değişkenleri

* **Tanım**: Metinsel ifadeleri saklamak için kullanılır.
*   **Örnek**:

    ```yaml
    username: "admin"
    ```

    Burada `username` adlı değişkenin değeri bir **string** metnidir.
* **Nerede Tanımlanabilir?**
  * Playbook içerisinde `vars:` altında
  * Envanter dosyasında (örn. `web1 ansible_host=... role="frontend"`)
  * Komut satırından `--extra-vars "username=admin"` şeklinde

**Kullanım Senaryosu**: Sunucu adları, kullanıcı isimleri, dosya yolları, mesaj metinleri vb. **metin tabanlı** bilgilerin saklanması.

***

### 2. Number (Sayı) Değişkenleri

* **Tanım**: Tamsayı (integer) veya ondalıklı (float) değerleri tutar.
*   **Örnek**:

    ```yaml
    max_connections: 100
    max_load: 1.5
    ```

    Burada `max_connections=100` (int), `max_load=1.5` (float).
* **Matematiksel İşlemler**\
  Ansible görevlerinde, bu değerler üzerinde toplama, çıkarma, karşılaştırma gibi işlemler yapılabilir.

**Kullanım Senaryosu**: Port numaraları, kaynak limitleri (CPU, bellek), tekrar sayıları (retry, max\_attempts) vb.

***

### 3. Boolean (Mantıksal) Değişkenleri

* **Tanım**: **true** veya **false** (doğru/yanlış) değeri saklar (ayrıca `yes/no`, `on/off` gibi truthy-falsy formatlarını da Ansible anlar).
*   **Örnek**:

    ```yaml
    debug_mode: true
    maintenance_mode: false
    ```
* **Kullanım**: Koşullu ifadelerde (`when:`) veya görevlerin etkin/pasif hale getirilmesinde sıkça kullanılır.

**Kullanım Senaryosu**: Özellikle `when:` şartlarında, “Eğer `debug_mode` açıksa ek log yaz, değilse yazma” gibi kontrol akışları.

***

### 4. List (Dizi) Değişkenleri

* **Tanım**: Sıralı bir koleksiyon. İçinde her türlü veri tipi (string, number, dictionary vs.) bulunabilir.
*   **Örnek**:

    ```yaml
    packages:
      - nginx
      - postgresql
      - git
    ```
* **Erişim**:
  * Tüm liste: `{{ packages }}`
  * Tek bir öğe: `{{ packages[0] }}` → `nginx`
* **Döngülerde Kullanım**: `loop: "{{ packages }}"` ile listedeki her bir öğe için görev yürütülebilir.

**Kullanım Senaryosu**: Toplu paket kurulumu, sunucu listeleri, çoklu parametre değerleri, vs.

***

### 5. Dictionary (Sözlük) Değişkenleri

* **Tanım**: Anahtar (key) - değer (value) çiftlerinden oluşan koleksiyon.
*   **Örnek**:

    ```yaml
    user:
      name: "admin"
      password: "secret"
    ```

    Burada `user.name` ve `user.password` şeklinde erişilebilir.
*   **Erişim**:

    ```yaml
    debug:
      msg: "User = {{ user.name }}, Password = {{ user.password }}"
    ```
* **Genişletilebilir**: Değerler de **list**, **dictionary** veya farklı türde veri olabilir.

**Kullanım Senaryosu**: Bir host veya servise ait detaylı bilgileri tek yapıda saklamak (örneğin kullanıcı bilgisi, servis konfigürasyonu, network ayarları gibi).

***

### 6. Değişkenleri Playbook’ta Kullanma

#### Örnek Kurulum Senaryosu

```yaml
- name: Install and Configure Services
  hosts: web
  vars:
    debug_mode: true
    packages:
      - nginx
      - php-fpm
      - git
    server_config:
      port: 8080
      doc_root: "/var/www/html"
  tasks:
    - name: Print debug info if debug_mode is true
      debug:
        msg: "We are in debug mode!"
      when: debug_mode

    - name: Install required packages
      package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"

    - name: Configure the service
      template:
        src: myconfig.j2
        dest: /etc/myconfig.conf
      vars:
        http_port: "{{ server_config.port }}"
        root_path: "{{ server_config.doc_root }}"
```

1. **Boolean Örneği**: `debug_mode` değişkeni `when: debug_mode` ifadesiyle koşullu görev çalıştırır.
2. **List Örneği**: `packages` içinde `nginx`, `php-fpm`, `git` tanımlı, `loop` ile sırayla kurulurlar.
3. **Dictionary Örneği**: `server_config` adlı sözlükten `port` ve `doc_root` alınarak `template` modülünde kullanılır.

{% hint style="info" %}
`item`, Ansible’ın **loop (döngü)** mekanizmasında **otomatik** olarak kullanılan bir “geçici değişken”dir. Listenin her bir elemanı, sırayla `item` isminde saklanır.
{% endhint %}

***

### 7. Sonuç ve İpuçları

* **Tür Seçimi**: Mümkünse veriyi doğru türe göre saklayın. Örneğin sayısal değerler `number`, evet/hayır durumları `boolean`, birden çok paket `list`, iç içe parametreler `dictionary`.
* **Esneklik**: Doğru tür kullanımı, playbook’larınızı daha okunabilir ve bakımı kolay hale getirir.
* **Erişim Söz Dizimi**:
  * String, number, boolean → Direkt `{{ variable }}`
  * List → `{{ list_name }}`, `{{ list_name[index] }}`
  * Dictionary → `{{ dict_name.key }}`, `{{ dict_name['key'] }}`
* **Yaml Formatı**: Ansible, YAML düzenine duyarlı olduğu için girintilere dikkat etmek önemlidir.

Bu şekilde, Ansible’ın **String**, **Number**, **Boolean**, **List** ve **Dictionary** değişken türlerini anladınız. Farklı veri türlerini uygun senaryolarda doğru kullanmak, **daha temiz, esnek ve yeniden kullanılabilir** Ansible kodları yazmanıza yardımcı olur.

