---
icon: plug-circle-check
---

# Introduction to Ansible Plugins

### 1. Ansible Plugin Nedir?

* **Tanım**: Ansible plugin, Ansible’ın **kendi çekirdek davranışını** veya **işlevselliğini** özelleştiren/yeni yetenekler ekleyen **küçük kod parçalarıdır**.
* **Neden Önemli?** Ansible zaten birçok “hazır özellik” sunar, fakat karmaşık ihtiyaçlarınız olduğunda veya standart özellikler yetersiz kaldığında **plugin** yazarak/kurarak **Ansible’ı genişletebilirsiniz**.

**Örnek**: Bir bulut sağlayıcısında (AWS, GCP vb.) sürekli değişen sunucuları otomatik keşfetmek mi istiyorsunuz? **Dinamik envanter eklentisi** (inventory plugin) kullanılabilir.

### 2. Plugin Türleri

Ansible’da çok sayıda **plugin türü** vardır. Her biri farklı bir alana odaklanır:

1. **Inventory Plugin (Dinamik Envanter Eklentisi)**
   * Ansible’ın envanteri (host listesi) normalde bir “ini” veya “yaml” dosyasında statik olabilir.
   * **Inventory plugin** sayesinde Ansible, bulut sağlayıcı API’lerinden, veritabanlarından veya başka sistemlerden **canlı** bilgi alarak **envanteri dinamik** olarak oluşturur.
   * **Örnek**: AWS, Azure vb. buluttan çalışan instance’ları otomatik çekip “inventory” haline getirir.
2. **Module Plugin (Modül Eklentisi)**
   * Mevcut modüller yeterli gelmiyorsa, kendiniz bir modül geliştirip **özelleştirilmiş işlemler** yapabilirsiniz.
   * **Örnek**: Kendi veri tabanınızı veya API’nizi çağırarak sunucu başlatsın, konfig yapsın.
3. **Action Plugin (Eylem Eklentisi)**
   * Belirli karmaşık işlemleri _tek bir yüksek seviyeli adım_ olarak tanımlamak için kullanılır.
   * **Örnek**: Yük dengeleyici (load balancer) yönetimi. Tek bir “action”la “SSL sertifika ayarla, health check ekle, routing kuralı koy” gibi alt adımları _arka planda_ halledebilir.
4. **Callback Plugin (Geribildirim/Etkileşim Eklentisi)**
   * Ansible playbook’un **çalışma aşamalarına** (start, task tamamlandı, error vb.) bağlanarak **olaylara tepki** vermenizi sağlar.
   * **Örnek**: Her görev bittiğinde log tutmak, Slack’e bildirim atmak ya da özel rapor oluşturmak.
5. **Lookup Plugin**
   * Harici kaynaklardan (veritabanı, API, dosya vb.) veri çekerek playbook içinde değişken gibi kullanmayı sağlar.
   * **Örnek**: “with\_url” ile birden çok URL’den bilgi çekmek, “with\_mongodb” ile veritabanındaki dokümanları döngülemek.
6. **Filter Plugin**
   * Değişkenleri, çıktıları, verileri **dönüştürmek**, formatlamak, ufak işlem yapmak için ek fonksiyonlar ekler.
   * **Örnek**: Veriyi uppercase yapmak, string bölmek, JSON parse etmek vb.
7. **Connection Plugin**
   * Ansible’ın uzak sunucularla nasıl iletişim kuracağını tanımlar. (SSH, WinRM, Docker vs.)

### 3. Neden Gerekli?

#### Örnek Senaryolar

1. **Dinamik Envanter**: Bulut üzerinde sürekli yenilenen instance’ları manuel envanterde takip etmek zor. **Inventory plugin** yazıp/kullanarak bulut API’den otomatik çekersiniz.
2. **Özel Modül**: AWS, GCP, Azure vb. bulutlarda “varsayılan modüller” yetmeyebilir. Kendi “module plugin” oluşturup spesifik API çağrıları yaparsınız (ör. özel parametrelerle instance açmak).
3. **Kompleks Load Balancer Yönetimi**: Bir “action plugin” yazarsınız, playbook’ta sadece “loadbalancer: config.yaml” dersiniz. Plugin tüm alt adımları (sertifika ekleme, health check vs.) halleder.

#### Faydaları

* Ansible’ı **kendi ihtiyaçlarınıza** göre esnetir.
* Kod tekrarını azaltır, **daha üst seviye** yapılarla çalışmanızı sağlar.
* **Bakımı** ve **geliştirmesi** kolay olur: karmaşık logic’i plugin içine gömersiniz.

### 4. Nasıl Kullanılır?

* Birçok plugin **halihazırda** Ansible çekirdeğinde mevcuttur (resmî eklentiler).
* Kendi plugin’inizi yazacaksanız:
  * Belirli dizin yapısında (örn. `plugins/inventory/`, `plugins/modules/`) bir Python dosyası oluşturup Ansible plugin API’sini implement edersiniz.
* **Örnek**: Dinamik envanter için “my\_cloud\_inventory.py” dosyası, bulut API’sine bağlanır, host listesini JSON formatında döndürür.

#### Plugin Example,

### 1. Dizin Yapısı

Ansible, **filter plugin**’lerini (ve diğer plugin türlerini) belirli dizinlerde arar. Basit bir projede şu şekilde bir yapı oluşturabiliriz:

```
my-ansible-project/
├── playbook.yml
└── filter_plugins/
    └── myfilters.py
```

* `filter_plugins/` klasörü: Burada yazdığınız Python dosyaları **filter plugin** olarak tanınır.
* `myfilters.py`: Plugin kodunu içerecek dosya.

***

### 2. Plugin Kodu (myfilters.py)

Aşağıdaki Python kodunu `filter_plugins/myfilters.py` içine koyun. Bu kod, bir **FilterModule** sınıfı tanımlar ve “reverse\_string” adında yeni bir filtre ekler.

```python
# filter_plugins/myfilters.py

class FilterModule(object):
    '''A simple filter plugin that reverses a given string'''

    def filters(self):
        return {
            'reverse_string': self.reverse_string
        }

    def reverse_string(self, value):
        """Reverses the given string."""
        if not isinstance(value, str):
            # Eğer string değilse, str'e dönüştürmeye çalışalım veya hata verelim
            value = str(value)
        return value[::-1]  # Python'da stringi tersine çeviren dilimleme
```

#### Açıklamalar

* **`FilterModule`**: Ansible’ın beklediği bir sınıf adı.
* **`filters(self):`**: Bu fonksiyon, plugin’in hangi filtreleri sağladığını bir **sözlük** (dictionary) halinde döndürür.
  * Key: Filtre Adı (`reverse_string`)
  * Value: Filtre fonksiyonu (`self.reverse_string`)
* **`reverse_string`** fonksiyonu: Parametre olarak `value` alır ve `value[::-1]` ile tersi string döndürür.

***

### 3. Playbook Kullanımı

Artık `reverse_string` filtresi, bu proje içinde **her yerde** kullanılabilir. Örnek bir playbook:

```yaml
# playbook.yml
- name: Demo Filter Plugin
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Show reversed string
      debug:
        msg: "{{ 'Hello Ansible' | reverse_string }}"
```

* Burada `'Hello Ansible'` string’ini **`reverse_string`** filtresi ile tersine çeviriyoruz.
* Eğer plugin doğru yüklendiyse, çıktıda `"elbisnA olleH"` görürsünüz.

***

### 4. Çalıştırma

```bash
ansible-playbook playbook.yml
```

* Ansible, `filter_plugins/` klasörünü otomatik tanıyacak ve **`myfilters.py`** dosyasındaki filtreleri yükleyecek.
* Ardından, `reverse_string` filtresini bularak `"Hello Ansible"` metnini ters çevirecek.

***

### 5. Sonuç

* **Filter Plugin**: Ansible’daki verileri veya string’leri dönüştürmek için basit bir eklenti yazmak işte bu kadar kolay.
* Benzer mantıkla **inventory plugin**, **callback plugin**, **module plugin** gibi diğer plugin türlerini de yazabilirsiniz.
* Bu örnekle “plugin” kavramının **nasıl çalıştığını** ve **hangi dosya yapısına** yerleştirildiğini görmüş oldunuz.

