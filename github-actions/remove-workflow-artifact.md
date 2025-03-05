---
icon: brain-circuit
---

# Remove workflow Artifact

### 1) Artifact Nedir?

* Bir **workflow** (iş akışı) içinde oluşturduğun dosyaları “Upload artifact” step’iyle GitHub’a yüklersen, bu dosyalar “Artifacts” bölümünde listelenir.
* İş akışı tamamlandığında, repo → Actions → ilgili workflow run sayfasından bu artifaktları indirebilirsin.

### 2) Varsayılan Saklama Süresi (Retention)

* GitHub, **varsayılan olarak** bu artifaktları **90 gün** saklar.
* Bu süre dolduğunda otomatik silinir.
* “Retention” süresini istersen YAML’da (ör. `retention-days`) veya depo ayarlarından özelleştirebilirsin.

### 3) Nereden Silinir?

1. **GitHub Actions** sekmesine git.
2. İlgili workflow run’ı seç (veya “All workflows” görüp run listesinden seç).
3. Sayfanın altında “Artifacts” bölümü bulunur.
4. Yanında (örneğin çöp kutusu simgesi) “Delete” butonu görebilirsin. Tıklayıp onaylarsan kalıcı silinir.

### 5) Özet

* Artifaktları **workflow bitince** GitHub saklar (varsayılan 90 gün).
* İstersen UI’de ilgili run sayfasından **manuel** silip diskte yer açabilirsin.
* Bir kez silinen artifakt geri yüklenemez.

#### Kullanım Örneği,

**Artifact upload**, GitHub Actions içinde oluşturduğun dosyaları **iş akışı (workflow) bittiğinde** saklayıp erişilebilir hale getirmek için kullanılır. Örneğin, **derleme (build) çıktıları**, **test raporları** veya **log dosyaları** gibi şeyleri sonraki aşamalarda (ya da takım arkadaşların, inceleme yapmak isteyenler) indirip kullanabilsin diye GitHub’ın sunucularına yüklersin.  Misal, Bir CI/CD senaryosunda, projenin derlenmiş hâlini (örneğin `dist/` klasörü ya da bir .zip dosyası) saklayıp, QA ekibi veya başka bir job sonradan indirip test edebilir. Veya, bir step başarısız olduğunda özel log dosyalarını artifakt olarak kaydedersen, sonradan debug amaçlı indirip bakabilirsin.

Belki bir “Release” job’ına geçmeden önce `.tar.gz` şeklinde bir paket oluşturursun. Bu paketi artifakt olarak kaydeder, sonra başka bir job veya kişi indirip manuel inceleme veya ek deploy adımlarında kullanabilir.

Aşağıda, **artifaktı** (dist klasörünü) oluşturup **yeni bir job** içinde **kullanan** basit bir örnek veriyorum. “build” job’ı “dist” klasörünü üretip **Upload artifact** yapıyor, “deploy” job’ı ise **Download artifact** adımıyla o klasörü indirip kullanıyor.

```yaml
name: Build and Deploy with Artifacts
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out
        uses: actions/checkout@v3

      - name: Install and build
        run: |
          npm install
          npm run build   # Bu komut dist klasörünü oluşturuyor (örn. dist/)

      - name: Upload build artifact
        uses: actions/upload-artifact@v3
        with:
          name: built-files     # Artifakt ismi
          path: dist           # Yüklediğimiz klasör

  deploy:
    runs-on: ubuntu-latest
    needs: build           # build job'u bitmeden deploy başlamaz
    steps:
      - name: Check out
        uses: actions/checkout@v3

      - name: Download build artifact
        uses: actions/download-artifact@v3
        with:
          name: built-files   # Yukarıda "name: built-files" diyerek yükledik
          path: dist          # Bu job’da “dist” klasörü olarak indirecek

      - name: Deploy
        run: |
          echo "Deploying dist folder..."
          # Burada dist klasöründeki dosyaları sunucuya yükleyebilir,
          # test edebilir veya başka bir adımda kullanabilirsin.
```

#### Nasıl Çalışır?

1. **build Job**
   * **npm install** ve **npm run build** komutlarıyla `dist/` klasörünü oluşturuyor.
   * `actions/upload-artifact@v3` yardımıyla “dist” klasörünü **“built-files”** adıyla GitHub’a artifakt olarak yüklüyor.
2. **deploy Job** (needs: build)
   * Önceki job bitince devreye girer.
   * `actions/download-artifact@v3` ile aynı “built-files” artifaktını indirip **dist/** klasörüne koyar.
   * Son adımda “Deploying dist folder…” diyerek bu klasör içindeki dosyaları istediğin ortama gönderebilir ya da başka işleme tabi tutabilirsin.

Bu sayede, **artifakt** (dist klasörü) bir job’da üretilip **başka job** (veya kişiler) tarafından **indirilebilir** hale gelir.



