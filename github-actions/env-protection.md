---
icon: file-lock
---

# Env Protection

**Environment Protection**, GitHub Actions’da “production” veya “staging” gibi **önemli ortamlara** (environments) yapılan iş akışlarını **kontrol** etmek için kullanılır. Böylece “Canlı sisteme (production) deploy” gibi kritik işlemler, belirli kurallara (örneğin insan onayı) bağlanır.

#### Basit Bir Örnek:

1. **Environment Tanımlama**
   * GitHub repo ayarlarında “Environments” sekmesinde “production” adlı bir ortam oluşturursun.
   * Bu ortama “koruma” (protection) ekler, örneğin “Bu ortama job başlamadan önce bir yönetici onay vermeli” gibi bir kural koyarsın.

**Workflow’ta Belirtme**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy step
        run: ./deploy.sh
```

* Bu job, “environment: production” diyerek “production” ortamına deploy yapacağını söyler.

1. **Koruma Kuralları (Protection Rules)**
   * “production” ortamında **“Required reviewers”** ayarladıysan, job **başlamadan** önce GitHub senin ekrandan onay ister (veya belirlenmiş yöneticilerden).
   * Onay verildikten sonra job runner’a gider ve çalışır.
2. **Secrets Erişimi**
   * Bu ortamın secrets’ları (ör. PROD\_API\_KEY) **ancak** onay verilince job’a açılır.
   * Böylece, izinsiz kimse o secrets’ı göremez ve gerçek canlı sisteme deploy yapamaz.

**Kısaca:**

* Environment = “production” gibi önemli ortam.
* Protection = “Bu ortama job’un gitmesi için önce onay/koşul gerek.”
* Job “environment: production” derse, GitHub **koruma kurallarını** uygular. Onaylanırsa job çalışır, onaylanmazsa durur. Bu şekilde istemeden canlı ortama dağıtım (deploy) yapılması engellenir.

Yani aslında **“environment: production”** demek, o job’un **production ortamına ait** tüm ayarlara (özellikle environment-level secrets veya değişkenlere) **erişebileceği** anlamına gelir. Genelde bu ortamı “deploy” amaçlı kullanırız (ör. “canlı sisteme kod gönderme” gibi), ama aynı zamanda “production” adlı environment’ın altındaki API\_HOST gibi değişken/secrets değerlerini de okuyabiliriz.

#### Nasıl İşliyor?

1. **Environment’de (ör. “production”) Tanımlı Secrets/Variables**
   * GitHub repo ayarlarında “Environments” sekmesine gidersin, “production” adlı environment oluşturursun.
   * Bu environment’a özel **variables** (ör. `API_HOST=prod.api.com`) veya **secrets** (`PROD_API_KEY=...`) eklersin.
2. **Job → “environment: production”**
   * Bu job, “production” ortamının secrets/variables’ına erişebilir.
   * Ayrıca, “production” ortamına eklenmiş **koruma kuralları (protection rules)** varsa, job başlamadan onay vs. gerekebilir.

Aşağıdaki Örneği inceleyelim,

* “production” adlı bir ortamımız (environment) var.
* Bu ortamda “vars.API\_HOST” ve “secrets.DB\_PASSWORD” tanımlı. (GitHub repo “Settings” → “Environments” → “production” → “Variables” & “Secrets”)
* Job, bu ortamı kullanmak istediği için **“environment: production”** satırıyla başlıyor.
* Koruma ayarı olarak (“requires review” vb.) “production” environment’ta tanımladığını varsayalım. Bu yüzden job, onay gelene kadar bekliyor.

```yaml
name: Env Protection Example
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production    # 1) "production" ortamına deploy ediyoruz.
    # Bu nedenle "production" environment'ın koruma kuralları (onay/review vb.) devreye girer.
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Print environment variable
        run: echo "Deploying to $API_HOST"
        env:
          # 2) production environment'ta tanımlı vars.API_HOST'e erişiyoruz.
          API_HOST: ${{ vars.API_HOST }}

      - name: Show DB Password
        # 3) Örnek (GERÇEK senaryoda normalde bu şifreyi ekrana basmak istemezsin, burada sadece gösterim amaçlı)
        run: echo "DB Password is $DB_PASS"
        env:
          # 4) production environment'ta tanımlı secret DB_PASSWORD'e erişiyoruz.
          DB_PASS: ${{ secrets.DB_PASSWORD }}

      - name: Deploy to production
        run: |
          echo "Starting deployment process..."
          # 5) Burada $API_HOST ve $DB_PASS gibi değişkenleri kullanarak gerçek deploy komutları çalıştırabilirsin.
```

#### Nasıl Çalışır?

1. **environment: production**
   * Bu job, “production” adlı ortamı kullanacağını belirtiyor.
   * GitHub, bu ortam için tanımlanmış **koruma** (approval required, vs.) ayarlarını uygular. Gerekirse job “Waiting for approval” durumunda kalır.
2. **Vars (ör. `vars.API_HOST`)**
   * “production” ortamı altında, “Variables” kısmında `API_HOST=myproduction.example.com` gibi bir değer girdiğini varsayalım.
   * YAML’da `env: API_HOST: ${{ vars.API_HOST }}` diyerek, “$API\_HOST” olarak erişiyoruz.
   * Sonra `echo "Deploying to $API_HOST"` çıktısı logda görülür (burada maskelenmez, çünkü “vars” non-sensitive’dır).
3. **Secrets (ör. `secrets.DB_PASSWORD`)**
   * Aynı “production” ortamının “Secrets” bölümünde `DB_PASSWORD=someS3cr3tValu3` olduğunu düşünelim.
   * YAML’da `env: DB_PASS: ${{ secrets.DB_PASSWORD }}` diyerek bu değeri bir shell değişkenine atıyoruz.
   * Loglarda normalde secret maskelenir (yıldızlanır).
4. **Deployment**
   * En sonda “Deploy to production” adımında bu değerleri kullanarak gerçek komutlarla canlıya geçiş yapabiliriz (örn. `./deploy.sh --host $API_HOST --dbpass $DB_PASS`).

#### Özet

* Job, “**environment: production**” diyerek hem vars (`vars.API_HOST`) hem de secrets (`secrets.DB_PASSWORD`) kullanıyor.
* Eğer ortamda “approval” kuralı varsa, job başlamadan repo yöneticisi onay vermeli. Bu şekilde **env protection** devreye girmiş olur.
* Normal “env” bloklarında “$API\_HOST” vb. değişkenleri, “$DB\_PASS” gibi secrets’ları güvenli biçimde kullanabiliriz.
