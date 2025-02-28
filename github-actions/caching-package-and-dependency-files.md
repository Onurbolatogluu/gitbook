---
icon: screen-users
---

# Caching Package and Dependency Files

**GitHub Actions Cache**, CI/CD süreçlerinde sıkça indirilen bağımlılıkları veya oluşturulan (build) dosyaları **tekrar kullanarak** iş akışlarını **hızlandırmak** için kullanılır.

### 1) Caching Package & Dependency Files

* **Amacı**: Her yeni workflow çalışmasında (run) tekrar bağımlılık indirmek (npm install, pip install, bundle install vb.) yerine, daha önce indirilmiş paketleri bir **cache**’ten almak ve işleri hızlandırmak.
* **Nasıl Yapılır?**
  *   Pek çok resmi “setup” action (örn. `setup-node`, `setup-python`, `setup-ruby`, `setup-java`) **“cache”** özelliği sunar. Örneğin:

      ```yaml
      - uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
      ```
  * Bu sayede gem’ler (Ruby paketleri) otomatik cache’lenir. Benzer şekilde `setup-node` action’ında `cache: npm` yazabilirsin.
  * Ya da **el ile** `actions/cache@v3` action’ı kullanıp `path` ve `key` belirleyerek kendi paket klasörünü cache’lersin.

#### Kullanım Örneği,

```yaml
name: Simple 2-Job Cache Example
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out
        uses: actions/checkout@v3

      - name: Cache Build Output
        id: build-cache
        uses: actions/cache@v4
        with:
          path: dist
          key: ${{ runner.os }}-dist
        # 1) "dist" klasörünü "runner.os-dist" anahtarıyla sakla veya geri yükle.
        #    Cache varsa "cache-hit" => "true", yoksa "false" döner.

      - name: Dist is already cached (skip build)
        if: steps.build-cache.outputs.cache-hit == 'true'
        run: echo "dist folder is found in cache, skipping build."
        # 2a) Cache bulunmuşsa, bu adımda sadece bilgi veriyoruz
        #     (burada build komutlarını atlıyoruz).
      
      - name: Build (only if cache not found)
        if: steps.build-cache.outputs.cache-hit != 'true'
        run: |
          echo "No dist cache found. Installing and building..."
          npm install
          npm run build
        # 2b) Cache yoksa (cache-hit != true), dist'i sıfırdan oluşturuyoruz.

  deploy:
    runs-on: ubuntu-latest
    needs: build   # build job’u bitmeden deploy başlamaz
    steps:
      - name: Check out
        uses: actions/checkout@v3

      - name: Restore Build Output
        uses: actions/cache@v4
        with:
          path: dist
          key: ${{ runner.os }}-dist
        # 3) Aynı anahtarla dist klasörü bulunursa, bu job da indirebilir.

      - name: Deploy
        run: echo "Deploying dist folder..."
        # 4) dist klasöründeki dosyaları kullan. 
        #    Build job’da güncel oluşturulduysa cache'e yüklendi, 
        #    şimdi buradan geri yüklenir.
```

#### Nasıl Çalışıyor?

1. **Cache Build Output (Build Job)**
   * “actions/cache@v4”, `dist` klasörünü `runner.os-dist` adlı bir cache anahtarıyla arar/oluşturur.
   * **Varsa** → “`cache-hit: true`”.
   * **Yoksa** → “`cache-hit: false`”.
2. **Dist is already cached (skip build)**
   * “if: steps.build-cache.outputs.cache-hit == 'true'” → Cache bulunduysa bu step çalışır.
   * Sadece “dist folder is found in cache…” mesajını basar, `npm install` vs. yapmaz.
3. **Build (only if cache not found)**
   * “if: steps.build-cache.outputs.cache-hit != 'true'” → Cache yoksa bu step devreye girer.
   * `npm install` + `npm run build` ile “dist” klasörünü oluşturur.
   * Job sonunda, yeni “dist” klasörü cache sunucusuna kaydedilir.
4. **Restore Build Output (Deploy Job)**
   * “deploy” job, aynı `path: dist` ve `key: runner.os-dist` kullanarak **dist** klasörünü **tekrar** cache sunucusundan yükler.
   * Dolayısıyla “dist” klasörü orada hazır olduğu için “Deploy” adımında kullanabilir.

**Özet**:\
Bu ek step’le beraber, **Build job** içinde de açıkça “cache varsa atla, yoksa build” mantığını görebiliyorsun.

