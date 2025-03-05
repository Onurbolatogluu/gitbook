---
icon: user-doctor-hair
---

# Job Matrix Configuration

**Job Matrix** (matris stratejisi), GitHub Actions’da tek bir job tanımını **birden fazla kombinasyonda** koşturmak için kullanılır. Örneğin farklı işletim sistemlerinde (Windows, Ubuntu) ve farklı sürümlerde (Node.js 10, 12, 14) testleri çalıştırmak istiyorsan, **tek bir job**u matris ile tanımlarsın; GitHub Actions **otomatik** olarak o kombinasyonlar için ayrı job’lar başlatır.&#x20;

Aşağıdaki workflow parçasında, **tek bir job** tanımı içinde **matris stratejisi** kullanılıyor. Böylece çeşitli kombinasyonlarda (farklı OS ve farklı Node.js sürümleri) test/build çalıştırabilirsin.&#x20;

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [ubuntu-22.04, ubuntu-20.04]
        version: [10, 12, 14]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
          
      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
        # 4) Tüm kombinasyonlarda testlerimizi koşturuyoruz
```

* **Matrix Tanımı**
  * `os: [ubuntu-22.04, ubuntu-20.04]` → İki işletim sistemi değeri.
  * `version: [10, 12, 14]` → Üç Node.js sürüm numarası.
  * GitHub Actions bu alanları **“çarpım”** şeklinde değerlendirir. Toplamda 2 (OS) × 3 (version) = **6** kombinasyon.
* **runs-on: $\{{ matrix.os \}}**
  * Her kombinasyonda “`matrix.os`” farklı olacağı için, **iki OS değerine göre** job “ubuntu-22.04” veya “ubuntu-20.04” runner’ında çalışır.
  * Mesela:
    * combination #1: os=ubuntu-22.04, version=10
    * combination #2: os=ubuntu-22.04, version=12
    * combination #3: os=ubuntu-22.04, version=14
    * combination #4: os=ubuntu-20.04, version=10
    * combination #5: os=ubuntu-20.04, version=12
    * combination #6: os=ubuntu-20.04, version=14
* **Node Sürüm Kurulumu**
  * `actions/setup-node@v4` action’ında “`node-version: ${{ matrix.version }}`” diyorsun.
  * Bu, her alt-job’da **farklı bir Node.js versiyonunu** (10, 12, 14) kurup kullanır.
* **Nasıl Çalışır?**
  * GitHub Actions, “example\_matrix” job’unu 6 alt-job’a bölerek paralel (veya kısıtlı kaynaksa sırasıyla) çalıştırır. Her alt-job “matrix.os” ve “matrix.version” kombinasyonunu kullanır.
  * Örnek olarak alt-job #1 → “ubuntu-22.04 + Node 10”, alt-job #2 → “ubuntu-22.04 + Node 12”, vb.
  * Böylece senin kodun/diyagramın hem Ubuntu 22.04/20.04’te hem de Node 10/12/14 sürümlerinde test edilebilir.



