---
icon: circle-caret-down
---

# Expressions - 2

#### 1) Metin İşleme Fonksiyonları

* **contains(search, item)**
  * Belirtilen `search` dizesinde `item` ifadesi geçip geçmediğini kontrol eder.
  * Örnek:
    * `if: ${{ contains('Hello world', 'llo') }}` → `true` döner, çünkü “llo” “Hello world” içinde geçiyor.
    * Bir YAML satırında:

```yaml
if: ${{ contains(github.event.head_commit.message, '[deploy]') }}
```

Böylece commit mesajında `[deploy]` geçiyorsa job ya da step çalışır.



**startsWith(searchString, searchValue)**

* `searchString` değeri, `searchValue` ile başlıyor mu diye bakar.
* Gerçek senaryoda branch kontrolü:

```yaml
if: ${{ startsWith(github.ref, 'refs/heads/feature-') }}
```

Eğer branch adı `feature-` ile başlıyorsa step’i çalıştır.



**endsWith(searchString, searchValue)**

* `searchString` değeri, `searchValue` ile bitiyor mu diye bakar.
* Örnek:
  * Tag kontrolü:

```bash
if: ${{ endsWith(github.ref, '/v1.0.0') }}
```

Branch ya da tag `'v1.0.0'` ile bitiyorsa çalıştır.



#### 2) Dize Birleştirme ve Biçimlendirme

* **format(string, replaceValue0, replaceValue1, ...)**
  * Metin içindeki `{0}`, `{1}`, `{2}` alanlarını sırasıyla girilen değerlerle değiştirir.
  * Örnek:
    * `format('Hello {0} {1}', 'Mona', 'Octocat')` → `"Hello Mona Octocat"`.
    * Bir “echo” komutunda kullanmak için:

```yaml
run: echo "${{ format('Deploying version {0} to {1}', '1.2.3', 'production') }}"
```



**join(array, optionalSeparator)**

* Array (dizi) elemanlarını birleştirir, istersen araya özel bir ayırıcı (separatör) koyar.
* Örnek:
  * `join(github.event.issue.labels.*.name, ', ')` → Issue üzerindeki label isimlerini virgülle birleştirir.
  * Ya da manuel array: `join(['apple','orange','banana'], ' | ')` → `"apple | orange | banana"`.

#### 3) JSON İşleme

* **toJSON(value)**
  * Verilen değişkeni ya da nesneyi JSON formatına çevirir (string olarak).
  * Örnek:
    * `if: ${{ toJSON(job) }}` → O anki job nesnesini JSON’a döndürür (daha çok debug amaçlı).
    * Step içinde:

```yaml
run: echo '${{ toJSON(github.event) }}'
```

Bu şekilde event içeriğini JSON formatında görürsün.



**fromJSON(value)**

* JSON formatındaki bir string’i GitHub Actions içinde bir nesneye dönüştürür.
* Örnek:
  * `fromJSON('{"name": "Mona", "role": "Octocat"}').role` → `"Octocat"`.
  * Eğer `client_payload`’da karmaşık bir JSON varsa bu fonksiyonla parse edebilirsin.

#### 4) Dosya Hash Fonksiyonu

* **hashFiles(path)**
  * Belirtilen dosya/dosyaların içeriğini hash’leyip tek bir hash döndürür.
  * Örnek:
    * `hashFiles('**/package-lock.json', '**/Gemfile.lock')` → Bu dosyaların toplu içeriğinin bir karmasını (hash) üretir.
    * Genelde cache kullanırken “cache key” üretmek için kullanılır. Örnek:

```yaml
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: node_modules
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```

