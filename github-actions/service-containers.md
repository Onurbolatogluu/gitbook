---
icon: box-open
---

# Service Containers

**Service containers**, GitHub Actions içinde bir job’un yanına tanımlayabileceğin **ek Docker konteynerleri**dir. Örneğin, **test** aşamasında bir **veritabanı** (MySQL, PostgreSQL), **cache** (Redis), **message broker** (RabbitMQ) vb. hizmete ihtiyaç duyarsan, bu hizmeti “service” olarak tanımlarsın ve aynı job içinde, test kodun bu konteynerle iletişime geçer. Aşağıda konuyu özetliyorum:

### 1) Service Container Nedir?

* GitHub Actions, job başlarken senin tanımladığın “service container” imajlarını **otomatik** olarak çalıştırır.
* Job bitince (veya başarısız olup da tamamlannca) bu container’lar **otomatik** kapatılıp kaldırılır.
* **Aynı job içindeki step’ler**, bu service container’lara “hostname” veya “localhost” + port üzerinden erişebilir.

#### Uses Cases

```yaml
name: Redis Cache Test

on:
  push:
    branches: [ "main" ]

jobs:
  cache-tests:
    runs-on: ubuntu-latest

    services:
      redis:
        image: redis:6-alpine          # 1) "redis:6-alpine" imajını kullan
        ports:
          - 6379:6379                 # 2) "6379" portunu job’a aç
        options: >-
          --health-cmd "redis-cli ping || exit 1"
          --health-interval 5s
          --health-retries 5
          --health-timeout 3s
        # 3) Bu "options" satırı, container hazır olana kadar (sağlıklı) bekler.
        # "redis" adında bir hostname ile job içindeki adımlar erişebilir.

    steps:
      - name: Check out repo
        uses: actions/checkout@v4
        # 4) Kodlarımızı çekiyoruz (örneğin test dosyalarımız vs.)

      - name: Install dependencies
        run: |
          npm install
        # 5) Node.js projesi ise bağımlılıkları yüklüyoruz

      - name: Run tests (with Redis)
        run: |
          # 6) Testlerimiz "redis" konteynerine bağlanacak. 
          # Varsayılan network modunda "redis" hostname:port (6379) üzerinden erişilebilir.
          npm run test
        env:
          REDIS_HOST: redis
          REDIS_PORT: 6379
        # 7) Testlerimize redis bağlantı bilgilerini veriyoruz (hostname: "redis")

      - name: Show Redis Info
        run: |
          # 8) Örnek: "redis-cli -h redis info" komutuyla Redis’e bağlanıp bilgi alıyoruz.
          redis-cli -h redis info

```

#### Use Case Açıklaması

* **Proje Senaryosu**: Örneğin Node.js tabanlı bir uygulama/cache kütüphanesi veya test suite’inin Redis’e ihtiyaç duyduğunu varsayalım. Bu workflow, “main” branch’e push yaptığında otomatik devreye girip testlerini koşturur.
* **Service Container**: “services.redis” alanı, job başlarken bir Redis konteyneri (versiyon 6, alpine imaj) açar. 6379 portunu job ile bağlar.
* **Erişim**: Testler “redis” adıyla hostname’e bağlanabilir. `env.REDIS_HOST` olarak “redis” ayarlanmıştır.
* **Sağlık Kontrolü**: “options” altındaki `--health-cmd` vs. Redis başlatılınca “READY” duruma gelene dek bekler.
* **Temizlik**: Job bittiğinde container otomatik kapatılır.

Böylece **her push**’ta “sıfırdan” temiz bir Redis ortamı oluşturulup test edilir, test sonra biter bitmez container kaldırılır. Bu yaklaşım, CI/CD’de paylaşılan veritabanı/servis kalıntılarından kaçınarak **izole, tekrarlanabilir test ortamı** sağlar.

#### Uses Case 2

```yaml
jobs:
  test-with-db:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:13
        ports:
          - 5432:5432
        env:
          POSTGRES_USER: myuser
          POSTGRES_PASSWORD: mypass
          POSTGRES_DB: mydb
        options: >-
          --health-cmd "pg_isready -U myuser" 
          --health-interval 10s 
          --health-timeout 5s 
          --health-retries 5
    steps:
      - uses: actions/checkout@v2
      - name: Install Dependencies
        run: npm install
      - name: Run Tests
        run: npm test
```

### 2) Nasıl Çalışır?

1. **Job Başlarken**
   * GitHub, senin `services:` altına yazdığın her imajı birer container olarak ayağa kaldırır.
   * “Bridge mode” ya da “Host mode” gibi network seçenekleriyle, job’un ana container’ı (veya runner) ile bu service container iletişim kurar.
2. **İletişim**
   * Default “bridge mode”da, service container’a **hostname** olarak “service id” kullanırsın. Yukarıdaki örnekte “postgres”.
   * Port mapping ile, “`5432:5432`” diyorsan job, “`<serviceName>:5432`” veya “`localhost:5432`” şeklinde erişebilir. Genelde hostname = “postgres” + port = 5432.
3. **Job Bitince**
   * Tüm service container’lar otomatik sonlandırılır ve temizlenir.
   * Manual cleanup gerekmez.

### 3) Neden Kullanmalı?

* **Kolay Test Ortamı**: Örneğin CI aşamasında veritabanına veya cache sistemine ihtiyaç varsa, sunucu kurmak yerine Docker imajını “services” olarak ekleyebilirsin.
* **Geçici ve İzole**: Her job ayrı container çalıştırdığı için test verileri, config vb. bir sonraki job’a sızmaz.
* **Otomatik Yönetim**: Container’ları elle başlatma/durdurma uğraşın olmadan, “services” tanımıyla her job’da istediğin ek hizmeti açarsın.

### 4) Dikkat Noktaları

1. **Sadece Linux Runner**
   * Docker tabanlı bu sistem, GitHub Hosted runner’larda **Ubuntu** kullanır veya Self-Hosted runner’da **Linux**makine gerekir. Windows/macOS runner’larda service containers desteklenmez.
2. **Network Ayarları**
   * Varsayılan “bridge” modda, hostname “service adın”dır. `ports:` ile port yönlendirmesi yapılır.
   * Bazı durumlarda “host” mod istersen, self-hosted runner ve Docker kurulumuna özel ayarlar gerekebilir.
3. **Büyük İmajlar**
   * Service container imajı çok büyükse, job başlarken indirme süresi uzun olabilir. Gerekirse caching yöntemleri veya optimize edilmiş imajlar kullanılmalı.
4. **Kısıtlamalar**
   * Service containers “composite actions” içinde tanımlanamaz. Sadece bir job context’inde çalıştırabilirsin.

#### Özet

* **Service Containers**: Job’un yanına ek Docker container’lar ekleyip, test veya uygulama için gerekli altyapı hizmetlerini hızlıca ayağa kaldırma yolu.
* **Otomatik yönetim**: Job başında kurulur, job bitince silinir.
* **Kolay Entegrasyon**: Step’lerin bu hizmete `hostname:port` üzerinden erişip test yapar.
* **Örnek**: DB, cache, MQ vb. test ortamları.

{% hint style="info" %}
Eğer “services” altındaki imaj özel bir registry’de barınıyorsa (Docker Hub Private, GHCR Private, vs.), “credentials” blokuyla login bilgilerini verebilirsin.
{% endhint %}

```yaml
services:
  redis:
    image: my-private-registry.com/redis
    credentials:
      username: ${{ secrets.dockerhub_username }}
      password: ${{ secrets.dockerhub_password }}
```

