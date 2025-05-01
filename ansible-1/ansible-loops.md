---
icon: arrow-rotate-left
---

# Ansible Loops

### 1. Neden Loops Kullanıyoruz?

Ansible’da bir görevi (örn. kullanıcı oluşturma, paket kurma gibi) birden fazla öğe (örn. birçok kullanıcı, birçok paket) için uygulamak istediğinizde, her öğe için ayrı bir “task” kopyalamak zorunda kalmazsınız. **Loops** kullanarak **tekrarlayan** görevlerden kurtulur, **daha az ve okunaklı** kod yazarsınız.

**Örnek**: Kullanıcı oluşturma işini tek satırda tanımlayıp, listede yer alan tüm kullanıcılar için **otomatik** tekrarlanmasını sağlayabilirsiniz.

### 2. Basit Bir Örnek: Düz Liste (“loop:”)

#### 2.1 Tek Öğe İçin Görev

```yaml
- name: Create a single user
  hosts: localhost
  tasks:
    - user:
        name: joe
        state: present
```

Bu şekilde tek bir kullanıcı “joe” oluşturuluyor.

#### 2.2 Birden Çok Öğe Olursa (Kopyalama Sorunu)

Beş, on, yirmi kullanıcı oluşturmak istersek aynı görevi tekrar tekrar kopyalamak zorunda kalırız:

```yaml
    - user:
        name: joe
        state: present
    - user:
        name: george
        state: present
    - user:
        name: ravi
        state: present
    ...
```

Bu **uzun, tekrarlı** ve **bakımı zor**.

#### 2.3 `loop:` Kullanın

```yaml
- name: Create multiple users
  hosts: localhost
  tasks:
    - user:
        name: "{{ item }}"
        state: present
      loop:
        - joe
        - george
        - ravi
        - mani
        - kiran
```

* **loop** parametresine bir liste (array) veriyoruz: `[joe, george, ravi, ...]`
* **Ansible**, bu listeyi **tek tek** döngüden geçirir, her yinelemede `item` adlı özel bir değişkende (örn. `joe`, sonra `george`, vb.) tutar.
* `name: "{{ item }}"` → her yinelemede yeni kullanıcı adı.

**Sonuç**: Daha **kısa** ve **temiz** bir playbook!

### 3. Liste Elemanları Daha Karmaşık Olursa (Dictionaries)

Bazen yalnızca kullanıcı adı değil, ek bilgiler (örn. UID) de vermek isteyebilirsiniz. Bu durumda döngü öğeleri birer **dictionary** (sözlük) olabilir:

```yaml
- name: Create users with specific UID
  hosts: localhost
  tasks:
    - user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        state: present
      loop:
        - { name: "joe",    uid: 1010 }
        - { name: "george", uid: 1011 }
        - { name: "ravi",   uid: 1012 }
        - { name: "mani",   uid: 1013 }
```

* Her **döngü öğesi**, `{ name: "...", uid: "..." }` gibi iki alan içerir.
* Ansible her yinelemede `item` değişkenini bu dictionary ile doldurur. `item.name` → “joe”, `item.uid` → 1010 vb.

#### Nasıl Çalıştığını Görselleştirmek

Döngü, bu playbook’u “ardışık” bir şekilde çalıştırır ve her seferinde `item`’a farklı değer atar:

1. `item = {name: "joe", uid: 1010}`
2. `item = {name: "george", uid: 1011}`
3. `item = {name: "ravi", uid: 1012}`
4. `item = {name: "mani", uid: 1013}` ... ve her seferinde tek bir “user” görevini uygular.

### 4. “with\_items” vs “loop”

Eskiden Ansible’da döngü yapmak için `with_items` sıklıkla kullanılırdı:

```yaml
- user:
    name: "{{ item }}"
    state: present
  with_items:
    - joe
    - george
    - ravi
```

**Günümüzde** `loop:` önerilen yaklaşım.\
Ancak eski playbook’larda `with_items`, `with_file`, `with_url` vb. görebilirsiniz. **Mantık** aynıdır, “her öğe için görevi tekrarla” fikrini uygular.

### 5. Diğer “with\_” Yönergeleri

#### `with_file`, `with_url`, `with_mongodb` vb.

* `with_file`: Birden çok dosya yolunu döngüyle işler.
* `with_url`: Birden çok URL üzerinde işlem yapar.
* `with_mongodb`: Birden çok MongoDB veritabanına bağlanmak için.

Bu “with\_\*” yöntemleri, Ansible’ın **lookup plugin** yapısına dayanır. Örneğin `with_file` ile bir dizi dosyayı tek tek okuyabilir, `with_url` ile bir dizi URL’yi ziyaret edebilirsiniz.

### 6. Özet

1. **loop:** → Bir liste üzerinde döngü kurar.
2. `item`: Döngüdeki her öğeyi temsil eder (string, dictionary vb.).
3. Sade bir liste → `item` doğrudan string olur. Dictionary listesi → `item.name`, `item.uid` vb. alanlara erişirsiniz.
4. **with\_items** gibi eski yöntemler benzer amaçla kullanılır, **loop** daha güncel ve önerilir.
5. Diğer “with\_” direktifleri (like `with_file`, `with_url`) → Spesifik data kaynağına iterasyon uygular.

***

### Lookup Plugin

Ansible’daki **lookup plugin**’ler, “harici bir kaynaktan veya farklı bir veri biçiminden” bilgiyi alıp Ansible içerisinde kullanmanızı sağlayan ufak eklentilerdir. Örneğin `with_file`, `with_url`, `with_k8s` gibi “with\_” direktiflerini gördüğünüzde, aslında bunlar birer **lookup plugin** çağırır ve iterasyona (döngüye) uygun veriyi döndürür.

### 1. Lookup Plugin Nedir?

* **Amaç**: Ansible’ın normalde “inventory” veya “vars” gibi basit yerlerden okuduğu veriyi daha geniş kaynaklardan (dosya sistemleri, veri tabanları, API’ler, bulut servisleri, vs.) dinamik olarak çekebilmenizi sağlamak.
* **Çalışma Mantığı**: Her “with\_\*” direktifi (ör. `with_dict`, `with_file`, `with_k8s`) kendi özel plugin’ini kullanır. Bu plugin, belirtilen kaynaktan döngü yapabileceğiniz bir veri listesi veya veri sözlüğü üretir.

**Örnek**:

* `with_filetree` → Belirttiğiniz bir dizin ağacını (file tree) tarar, orada bulduğu dosyalar hakkında Ansible’da iterasyon yapmayı sağlar.
* `with_dict` → Elinizdeki sözlük (dictionary) yapısını döngüleyip key/value çiftlerini `item.key`, `item.value` gibi alanlar halinde kullanmanıza izin verir.
* `with_sequence` → Otomatik sayı listesi (örn. 1’den 10’a) oluşturmak ve her birinde ayrı işlem yapmak için kullanılır.

#### Nasıl Çalıştığını Düşünmek

1. Görevinizde `with_XXX` (örn. `with_dict`) parametresi yazarsınız.
2. Ansible, “XXX” ismindeki **lookup plugin**’ini çağırır.
3. Plugin, belirtilen kaynaktan (dictionary, environment variables, vs.) verileri toplar ve bir liste veya sözlük haline getirip Ansible’a döndürür.
4. Ansible, bu listede veya sözlükteki öğeler üzerinde **döngü** (loop) işlemi yapar. Her öğeyi `item` veya benzer bir değişkene atar.

### 2. Örnek: `with_dict`

```yaml
- name: Process dictionary items
  hosts: localhost
  tasks:
    - debug:
        msg: "Key={{ item.key }}, Value={{ item.value }}"
      with_dict:
        my_var1: "Hello"
        my_var2: "World"
        my_var3: "Test"
```

* Bu görevde, `with_dict` plugin’i, `{ my_var1: "Hello", my_var2: "World", my_var3: "Test" }` sözlüğünü **key-value çiftleri** listesine dönüştürür.
* Her turda `item.key` ve `item.value` geliyor (örn. `my_var1` / `Hello`, `my_var2` / `World`).

### 3. Sık Kullanılan Lookup Plugin’leri

* **`with_filetree`**: Belirlediğiniz dizin altındaki dosya/yapıları iterasyona sokar.
* **`with_env`**: Ortam değişkenlerini (`$ENV_VAR`) okuyup döngü kurar.
* **`with_inventory_hostnames`**: Envanterdeki host isimlerine döngü.
* **`with_password`**: Parolalar oluşturmak/okumak için.
* **`with_sequence`**: Bir sayı dizisi (1..N) üzerinde döngü kurmak için.
* **`with_subelements`**: İç içe listeler (nested lists) üzerinde döngü.

**Not**: Hepsi benzer şekilde çalışır: Farklı kaynaklardan verileri okur, Ansible’a bir liste döndürür, Ansible her öğeyle bir tur görev çalıştırır.

### 4. Özet

* **lookup plugin**: Ansible’ın “dış kaynak / özel format” verilerini alması ve döngü kurması için gerekli küçük eklentilerdir.
* _with\_ direktifleri_\*: Bu plugin’leri **kullanım şekli**. Örnek: `with_dict`, `with_file`, `with_url` vb.
* **Mantık**: Plugin veriyi toplar → Ansible bir liste elde eder → her öğe `item` şeklinde döngüde kullanılır.
* **Yararları**: Daha esnek, dinamik playbook’lar yazmanızı sağlar (farklı veri kaynaklarından kolay veri çekebilirsiniz).

Bu sayede, Ansible’da “with\_\*” yazdığınızda aslında bir “lookup plugin”’i çağırdığınızı bilmelisiniz. Böylece veri kaynağını plugin aracılığıyla döngüye sokuyor, her turda `item` değişkeni ile o öğeyi kullanıyorsunuz.

