---
icon: comment-question
---

# Frequently Asked Questions

### 1) YAML’da Boolean (True/False) Yazım Şekli

Ansible, boolean değerleri (**true/false**) farklı şekillerde kabul eder:

* `yes`, `no`
* `true`, `false`
* `True`, `False`
* `TRUE`, `FALSE`

Örnek:

```yaml
gather_facts: yes
# ya da
gather_facts: true
# Tümü Ansible tarafından 'true' olarak yorumlanır.
```

Aynı şekilde “hayır” değeri için `no`, `false`, `False`, `FALSE` de kullanılabilir. Hepsi aynı anlama gelir, sadece yazım stilleri farklıdır.

**Tavsiye**: Projenizde bir stil seçip ona tutarlı şekilde devam edin.

### 2) Üç Tane Tire (---) Dosyanın Başı

YAML dosyalarının başına bazen `---` (üç tire) konur:

```yaml
---
- name: My play
  hosts: all
  tasks:
    - debug:
        msg: "Hello"
```

* **Opsiyonel** bir belirtimdir ve tek bir YAML dokümanı içinde **zorunlu değildir**.
* **Amaç**: Çoklu YAML dosyalarını tek bir yerde birleştirirken, her dokümanın başlangıcını netleştirmektir.

Ansible, `---` eklenip eklenmemesine pek takılmaz. Küçük projelerde kullanmasanız da olur.

### 3) Değişken Kullanımında Çift Süslü Parantez (\{{ \}})

Ansible, değişkenleri **Jinja2** şablon sistemiyle işler. Normalde bir değişkenin değerini kullanmak istediğinizde:

```yaml
msg: "DNS Server IP: {{ dns_server_ip }}"
```

* **`{{ dns_server_ip }}`** → Değişkenin değeri buraya yerleştirilir.

#### Nerede \{{ \}} Kullanmıyoruz?

1.  **`var` Parametresi** (debug modülünde):

    ```yaml
    - debug:
        var: dns_server_ip
    ```

    Burada `var:` kendisinin değişkene ihtiyacı olduğunu bildiği için `{{ }}` yazmanıza gerek yok.
2.  **`when` Koşulu**:

    ```yaml
    when: ansible_os_family == "Debian"
    ```

    `when` ifadesi değişkeni otomatik tanır. Süslü parantez gerekmez.
3.  **`loop` / `with_items`** Kullanınca:

    ```yaml
    with_items: "{{ some_list }}"
    ```

    Burada `loop` veya `with_items` ifadesi içinde **değişkeni çift süslü parantezle** yazmalısınız. Çünkü o ifadenin neyin liste olduğunu anlaması için bu gerekli.

**Özet**:

* `debug: var=` ve `when:` → **süslü parantezsiz**
* normal mesaj ekleme veya `loop` → **süslü parantezli**

### 4) Mesaj Başına Değişken Gelince Tırnak

Eğer değerin **tamamı** sadece bir değişkense, başta `{{ }}` ile başladığı için Yaml’ın bunu string olarak doğru algılaması adına çift veya tek tırnak kullanmanız gerekir. Örnek:

```yaml
msg: "{{ dns_server_ip }}"
```

Başka metinle birlikteyse zorunlu değil:

```yaml
msg: "DNS Server IP: {{ dns_server_ip }}"
```

ya da

```yaml
msg: DNS Server IP: {{ dns_server_ip }}
```

(Çift tırnak zorunlu olmaktan çıkar.)

### 5) `ansible_ssh_pass` mi, `ansible_password` mı?

Önceden `ansible_ssh_pass` kullanılıyordu, ancak artık **`ansible_password`** tercih ediliyor. Eski playbook veya dokümanlarda `ansible_ssh_pass` görebilirsiniz, ama **yeni projelerde** `ansible_password` daha doğru ve güncel yaklaşım.

### Sonuç

1. **Boolean değerler** (yes/no/true/false) Ansible için eşdeğer.
2. **Üç tire (---)** opsiyonel, genellikle birden fazla YAML dokümanı birleştirmek içindir.
3. **`{{ }}`** (çift süslü parantez) değişken interpolasyonu için kullanılır. Ancak `when:` ve `debug: var=` gibi belirli durumlarda gerekmez.
4. **`ansible_password`** tercih edin; `ansible_ssh_pass` eski yöntemin adı.

