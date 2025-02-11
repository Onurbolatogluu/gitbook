---
icon: square-code
---

# Conditional Keywords For Steps

**GitHub Actions’da Koşullu (Conditional) Çalıştırma** özelliği, bir işin (job) ya da bir adımın (step) **yalnızca belirli bir koşul sağlandığında** çalıştırılmasını sağlar.&#x20;

#### Örnek 1: Commit Mesajında “release” Geçiyorsa deploy Job’u Çalıştır

Bu senaryoda, `deploy` isimli job, sadece commit mesajında “release” kelimesi geçiyorsa devreye girecek. Örneğin sürüm yükseltmesi (release) yaptığında otomatik deploy olsun, aksi hâlde atlanacak.

```yaml
name: conditional-jobs-example
on:
  push:
    branches: [ "main" ]  # main branch'e push olduğunda

jobs:
  deploy:
    if: ${{ contains(github.event.head_commit.message, 'release') }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Run deployment script
        run: echo "Deploy işlemi yapılıyor..."
```

**Ne oluyor?**

* `github.event.head_commit.message` → Son commit mesajı.
* `contains(...)` ifadesiyle “release” kelimesi geçiyor mu diye bakıyoruz.
* Eğer `true` dönerse, job çalışır; `false` ise job “skipped” olur.

#### Örnek 2: Yalnızca Prod Ortamına Ait Repoda Deploy Job’u Çalıştır

```yaml
name: deploy-to-prod-repo
on: [push]

jobs:
  prod-deploy:
    if: ${{ github.repository == 'my-org/my-repo-prod' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Bu job yalnızca my-org/my-repo-prod isimli repoda çalışacak."
```

**Ne oluyor?**

* `github.repository` → “OrganizasyonAdı/RepoAdı” şeklinde bir string.
* Koşul sağlanmazsa job hiç başlamıyor.

#### Örnek 3: Matris (Matrix) Yapısında Belirli Bir OS İçin Ek Adım Çalıştırma

Diyelim ki matrix ile Ubuntu, Windows ve Mac üzerinde build yapıyorsun. Bazı adım sadece Ubuntu’da çalışsın istiyorsan şöyle yapabilirsin:

```yaml
name: matrix-conditional-example
on: [push]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Ubuntu-specific step
        if: ${{ matrix.os == 'ubuntu-latest' }}
        run: echo "Bu adım sadece ubuntu-latest üzerinde çalışacak."
```

**Ne oluyor?**

* Job aynı anda 3 farklı işletim sisteminde çalışır.
* `Ubuntu-specific step`, `if` koşulu yüzünden yalnızca Ubuntu build’ında çalışır, Windows ve Mac’te atlanır.

