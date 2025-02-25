---
icon: docker
---

# Push Package #2 Docker

```yaml
name: Publish Docker image

on:
  release:
    types: [published]
    # Bu workflow, bir "release" yayınlandığında ("published") tetiklenecek.

jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub
    runs-on: ubuntu-latest
    # Job, Ubuntu ortamında çalışacak.

    steps:
      - name: Check out the repo
        uses: actions/checkout@v4
        # Kodları runner'a indir, Dockerfile gibi dosyalara erişimi sağlar.

      - name: Log in to Docker Hub
        uses: docker/login-action@4ef78c080cd8ba55a85445d5b36e214a81df20a
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
          # Docker Hub'a giriş yapabilmek için kullanıcı adı ve parola,
          # GitHub Secrets içinden alınıyor.

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9c57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: my-docker-hub-namespace/my-docker-hub-repository
          # Bu action, imaj için otomatik tag ve label oluşturuyor;
          # outputs.tags, outputs.labels olarak sonraki adıma iletir.

      - name: Build and push Docker image
        uses: docker/build-push-action@3b5e8027fcad23fda98b2e3ac259d8d67585f671
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          # Bu adımda Docker imajı build edilir ve Docker Hub'a push yapılır.
          # Tag/label bilgileri "meta" adımının çıktısından alınır.
```



