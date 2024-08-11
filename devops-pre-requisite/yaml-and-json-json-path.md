# 🟦 YAML & JSON - JSON Path

### YAML Nedir?

**Y**AML **A**in't **M**arkup **L**anguage'in kısaltmasıdır ve JSON ya da XML gibi veri formatlarını temsil etmek için kullanılan insan tarafından okunabilir bir veri dilidir. "YAML Ain't Markup Language" ifadesi, YAML'ın bir işaret dili olmadığını vurgular. Bunun yerine, veri yapılarını tanımlamak ve saklamak için kullanılır.

* YAML, sade ve anlaşılır bir sözdizimi sunar. Genellikle JSON veya XML'den daha okunabilir ve yazması daha kolaydır.
* YAML, veri yapısını ifade etmek için girintileme (indentation) kullanır. Bu, hiyerarşileri ve iç içe geçmiş yapıları göstermek için kullanılır.
* YAML, genellikle anahtar-değer çiftlerini kullanarak veri yapılarını ifade eder.
* YAML, uygulamaların yapılandırma dosyalarını yazmak için yaygın olarak kullanılır. Örneğin, Docker, Kubernetes, Ansible gibi araçlar YAML formatını kullanır.

```yaml
# YAML'da Temel Sözdizimi ve Yapılar

# 1. Anahtar-Değer Çiftleri:
# Aşağıdaki örnek, basit bir anahtar-değer çiftlerini gösterir.
name: Onur           # 'name' anahtarının değeri 'Onur'
age: 30              # 'age' anahtarının değeri '30'
city: Istanbul       # 'city' anahtarının değeri 'Istanbul'

# 2. Array:
# Bu örnek, bir listeyi nasıl tanımlayacağınızı gösterir.
fruits:
  - Apple           # Listenin ilk elemanı 'Apple'
  - Orange          # Listenin ikinci elemanı 'Orange'
  - Banana          # Listenin üçüncü elemanı 'Banana'

# 3. Dictionaries:
# Sözlükler, iç içe geçmiş veri yapıları oluşturmak için kullanılır.
person:
  name: Onur        # 'name' anahtarının değeri 'Onur'
  age: 30           # 'age' anahtarının değeri '30'
  address:          # 'address' anahtarı, iç içe geçmiş bir sözlük içerir
    city: Istanbul  # 'city' anahtarının değeri 'Istanbul'
    zip: 34000      # 'zip' anahtarının değeri '34000'

# 4. Karmaşık Yapılar:
# Bu örnek, karmaşık yapıların ve birden fazla kişinin bilgilerini içeren bir listeyi gösterir.
employees:
  - name: John Doe  # İlk çalışanın adı 'John Doe'
    age: 25         # Yaşı '25'
    department: IT  # Departmanı 'IT'
  - name: Jane Doe  # İkinci çalışanın adı 'Jane Doe'
    age: 28         # Yaşı '28'
    department: HR  # Departmanı 'HR'

```



### JSON Nedir?

**JSON** (JavaScript Object Notation), veri alışverişi ve veri depolama için kullanılan, insan tarafından okunabilir, yaygın olarak kullanılan bir veri formatıdır. JSON, özellikle web tabanlı uygulamalarda sunucu ile istemci arasında veri iletimi için popülerdir.&#x20;

* JSON, veri yapılarını anahtar-değer çiftleri olarak temsil eder.
* JSON, birden fazla değeri bir dizi olarak saklayabilir.

**JSON Yapısı:**

JSON, iki temel yapı taşına sahiptir:

1. **Nesneler (Objects):** Anahtar-değer çiftlerinden oluşur ve `{}` süslü parantezleri içinde tanımlanır.
2. **Diziler (Arrays):** Birden fazla değeri liste olarak saklar ve `[]` köşeli parantezleri içinde tanımlanır.

```json
{
  "name": "Onur",
  "age": 30,
  "isStudent": false,
  "skills": ["Python", "JavaScript", "Docker"],
  "address": {
    "city": "Istanbul",
    "zip": "34000"
  }
}

```

* `name`, `age`, ve `isStudent` anahtarları scalar değerleri tutar (string, number, boolean).
* `skills` anahtarı bir dizi (array) tutar.
* `address` anahtarı bir nesne (object) tutar ve bu nesne içinde başka anahtar-değer çiftleri bulunur.

#### JSON Path Nedir?

**JSONPath**, JSON verileri içinde belirli veri parçalarını sorgulamak ve çekmek için kullanılan bir sorgulama dilidir. JSON veri yapılarında gezinmek ve belirli öğeleri seçmek için kullanılır.

**JSONPath'in Temel Kullanımı:**

* **(`.`):** JSON objeleri içinde belirli bir anahtarın değerine erişmek için kullanılır.
* **(`[]`):** JSON dizilerinde belirli bir indeksteki öğeye erişmek için kullanılır.
* Belirli koşullara uyan JSON öğelerini seçmek için filtreleme yapılabilir.

```json
# 1. Basit Erişim:
$.name
# Çıktı: "Onur"

# 2. Dizi Öğesine Erişim:
$.skills[0]
# Çıktı: "Python"

# 3. Nesne İçinde Erişim:
$.address.city
# Çıktı: "Istanbul"

# 4. Filtreleme (Dizi İçinde Arama):
$.skills[?(@ == 'Docker')]
# Çıktı: "Docker"

```



