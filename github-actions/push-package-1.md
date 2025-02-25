---
icon: box-taped
---

# Push Package #1

**GitHub Actions** kullanarak **Docker imajı** oluşturma ve ardından **GitHub Container Registry (GHCR)**’ye (veya benzeri bir container registry'e) **pushlama** (yayınlama) sürecini gösteriyor.

### 1) Amaç: Docker İmajını Build Edip GitHub Packages’e Push Etmek

* **Neden?**
  * Projendeki Docker imajını **GitHub’ın container registry’sinde** saklayabilir, versiyonlayabilir ve dilediğin yerde kullanabilirsin.
  * Böylece Docker Hub gibi harici servislere bağımlı kalmadan, GitHub platformu içinde CI/CD ve paket yönetimini birleştirirsin.

### 2) Örnek Workflow Yapısı

#### `on.push.branches: [ 'release' ]`

* Bu Workflow, “release” branch’ine yapılan push’larda tetikleniyor.
* Genelde “main” veya “release” branch’lerine commit atınca otomatik Docker imajı build + push yapmak istenebilir.

Ortak “env” Değişkenleri;

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```

* `REGISTRY=ghcr.io`: İmajı GHCR’ye atmak istediğini belirtiyor.
* `IMAGE_NAME=${{ github.repository }}`: Repo adını Docker imajı adı olarak kullanıyor (örnek: `owner/repo`).

#### permissions

```yaml
permissions:
  contents: read
  packages: write
```

* Docker imajı push edebilmek için “packages: write” izni gerekli.
* GITHUB\_TOKEN kullanırken bu permission’ı eklemezsen push işlemi başarısız olur.

### 3) Adımlar (Steps) İncelemesi

1. **C**heckout repository

```yaml
uses: actions/checkout@v4
```

2. Log in to the Container registry

```yaml
uses: docker/login-action@<commit-SHA-or-version>
with:
  registry: ${{ env.REGISTRY }}
  username: ${{ github.actor }}
  password: ${{ secrets.GITHUB_TOKEN }}
```

* `docker/login-action` ile GHCR (ya da belirttiğin registry) için login yapıyorsun.
* Kullanıcı adı “GitHub aktörü” (workflow’u tetikleyen kullanıcı), şifre olarak da “secrets.GITHUB\_TOKEN” kullanılıyor.
* Bu şekilde Docker CLI “login” komutu yapmadan, action kendisi authentication’ı hallediyor.

3. Extract metadata (tags, labels) for Docker

```yaml
uses: docker/metadata-action@<commit-SHA>
with:
  images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
```

* Bu step, Docker imajına eklenecek **tag**, **label** gibi bilgileri otomatik oluşturuyor.
* Örneğin commit SHA, sürüm etiketi, vs. Docker imajına eklenebilir.
* Çıktısını (`steps.meta.outputs.tags` vb.) sonraki adımda kullanıyor.

4. Build and push Docker image

```yaml
uses: docker/build-push-action@<commit-SHA>
with:
  context: .
  push: true
  tags: ${{ steps.meta.outputs.tags }}
  labels: ${{ steps.meta.outputs.labels }}

```

* Asıl işi yapan step: Docker imajını build edip GHCR’ye push ediyor.
* `context: .` → “.” klasöründen Dockerfile alır.
* `push: true` → Build bittikten sonra registry’ye push etsin.
* `tags`, `labels` → Bir önceki “metadata-action” step’inden gelen bilgileri kullanıyor.

#### Özet

* Bu Workflow, “release” branch’e her push geldiğinde Docker imajı oluşturup GHCR’ye yayınlıyor.
* `docker/login-action` ile oturum açılıyor, `docker/metadata-action` ile etiketler ayarlanıyor, `docker/build-push-action` ile imaj build/push ediliyor.
* Sonuçta projenin Docker imajı GHCR’de saklanmış oluyor, böylece diğer ortamlarda veya sunucularda doğrudan çekilip kullanılabiliyor.
