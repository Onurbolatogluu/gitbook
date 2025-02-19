---
icon: face-dotted
---

# Default & Custom Env Varb

GitHub Actions’da **Environment Variables (env vars)** hem GitHub tarafından varsayılan olarak sağlanan bilgilerden (default env vars) hem de kendi tanımladığın değişkenlerden (custom env vars) oluşur. Bu değişkenlere, Workflow içindeki adımlarda (`steps`) direkt `echo "$DEGISKEN_ADI"` şeklinde ya da Actions expression syntax’ıyla (`${{ env.DEGISKEN_ADI }}`) erişebilirsin.

{% embed url="https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables" %}

### 1) Default Environment Variables

GitHub, **her job** ve **her step** için **otomatik** tanımladığı bazı ortam değişkenlerini sağlar. Örnekler:

* **`GITHUB_REF`**: Hangi branch veya tag üzerinde çalıştığını (`refs/heads/main`, `refs/tags/v1.0.0`) gösterir.
* **`GITHUB_REPOSITORY`**: “Sahip/RepoAdı” formatında, örneğin “octocat/Hello-World”.
* **`GITHUB_WORKFLOW`**: İş akışının adı (Workflow dosyasında `name:` ile belirttiğin değer).
* **`GITHUB_ACTOR`**: İş akışını tetikleyen kullanıcı veya uygulamanın adı.
* **`GITHUB_SHA`**: Tetiklenen commit’in SHA’sı.
* **`RUNNER_OS`**, `RUNNER_NAME`, `RUNNER_TEMP`, `RUNNER_ARCH` gibi, çalıştığı runner’ın işletim sistemi, adı, mimarisi vb. bilgileri tutar.

**Kullanım Örneği,**

```yaml
jobs:
  example_job:
    runs-on: ubuntu-latest
    steps:
      - name: Print GitHub Environment Variables
        run: |
          echo "Repository: $GITHUB_REPOSITORY"
          echo "Workflow: $GITHUB_WORKFLOW"
          echo "Action: $GITHUB_ACTION"
          echo "Actor: $GITHUB_ACTOR"
```

* Bu step, GitHub’ın otomatik oluşturduğu değişkenleri terminal çıktısına basar.
* “$GITHUB\_REPOSITORY” gibi ifadeler, default environment variables’dan gelir.

### 2) Custom Environment Variables

Workflow içinde kendi istediğin değişkenleri de tanımlayabilirsin. Bu tanımlamaları **üç farklı düzeyde** yapabilirsin:

1. Workflow Düzeyinde (`env:` en üstte)

```yaml
name: My Workflow
on: [push]
env:
  DAY_OF_WEEK: Monday
  API_URL: https://api.example.com

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Today is $DAY_OF_WEEK and the API is $API_URL"
```

* Tüm job’lar ve step’ler, `DAY_OF_WEEK` ve `API_URL` değişkenlerini görebilir.



2. Job Düzeyinde (`env:` job altında)

```yaml
jobs:
  greeting_job:
    runs-on: ubuntu-latest
    env:
      GREETING: Hello
    steps:
      - run: echo "$GREETING from job-level environment"
```

Sadece bu job içinde geçerli olur. Diğer job’lar bu değişkeni göremez.



3. Step Düzeyinde (`env:` step altında)

```yaml
steps:
  - name: Step with custom env
    env:
      FIRST_NAME: Mona
    run: echo "Hello $FIRST_NAME"
```

* Sadece bu step için geçerli olur, diğer step’lere aktarılmaz.



#### Koşul (if) İfadelerinde Kullanma

* `env` değişkenlerini koşullu ifadelerde şöyle kullanabilirsin:

```yaml
steps:
  - name: Conditional step
    if: ${{ env.DAY_OF_WEEK == 'Monday' }}
    run: echo "It's Monday!"
```

* Burada “`env.DAY_OF_WEEK`” expression syntax içinde kullanılıyor.

#### İç İçe Yazma Sırası

* GitHub Actions sırasıyla en üstteki “env” → job düzeyindeki “env” → step düzeyindeki “env” şeklinde okur.
* Aynı isme sahip bir değişken en altta (step düzeyinde) yeniden tanımlanırsa, o step için bu en güncel değeri kullanır (override).

### Özet

1. **Default Env Vars**
   * GitHub’ın sağladığı “`GITHUB_*`”, “`RUNNER_*`” gibi değişkenlerdir (örneğin `GITHUB_REF`, `GITHUB_WORKFLOW`, `RUNNER_OS`).
   * Her step’te otomatik bulunur, “`$GITHUB_REF`” vb. şekilde kullanılır.
2. **Custom Env Vars**
   * Kendi istediğin değerleri tanımlayıp `$DEGISKEN_ADI` şeklinde kullanırsın.
   * Düzeyler: **Workflow**, **Job**, **Step**.
   * Daha spesifik düzeyde tanımlanan env var, önceki düzeydekileri geçersiz kılar.\
