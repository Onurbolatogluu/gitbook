---
icon: chart-network
---

# Workflow Context

**GitHub Actions Context’leri**, bir Workflow içinde pek çok farklı bilgilere (ör. hangi branch üzerinde çalışıldığı, hangi ortam değişkenleri set edildiği, hangi sıradaki job/step’in yürüdüğü vb.) erişebilmeni sağlar. Bu bilgilere “**expression syntax**” kullanarak, yani `${{ context.key }}` şeklinde ulaşabilirsin.&#x20;

### 1) `github`

* **Ne içerir?**
  * Repo bilgisi (ör. `github.repository` → “owner/repo”)
  * Event bilgisi (ör. `github.event_name`, `github.event.pull_request.title`, vb.)
  * Ref bilgisi (ör. `github.ref` → hangi branch/tag)
  * Actor bilgisi (`github.actor` → iş akışını tetikleyen kullanıcı)

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

Bu örnekte, branch ana dal (`main`) ise koşul sağlanır.



### 2) `env`

* **Ne içerir?**
  * Workflow, job veya step düzeyinde tanımladığın environment değişkenleri.

```yaml
env:
  FOLDER: "scripts"
steps:
  - name: Print folder
    run: echo "Folder is ${{ env.FOLDER }}"
```

`env.FOLDER` → `"scripts"` bilgisini döndürür.



### 3) `vars`

* **Ne içerir?**
  * GitHub ayarlarında (repo, org veya environment düzeyinde) tanımlanmış **değişkenler** (Variables) bulunur.
* **Farkı nedir?**
  * `env` Workflow içinde tanımladığın değişkenler,
  * `vars` ise GitHub’ın “Settings → Variables” menüsünde eklediğin değişkenlerdir.

```yaml
run: echo "Global variable: ${{ vars.GLOBAL_VAR }}"
```

`vars.GLOBAL_VAR` → GitHub’ın arayüzünde tanımladığın bir değer döner.



### 4) `job`

* **Ne içerir?**
  * O an çalışan job ile ilgili bilgiler: job adı, durumu, container ayarları vb.
* **Kullanım Örneği**
  * Çok sık kullanılmaz, ancak job düzeyinde metadata gerekirken faydalı olabilir. Örneğin `job.status` vb.



### 5) `jobs` (Reusable Workflows İçin)

* **Ne içerir?**
  * Reusable Workflow’larda, birden fazla job’un çıktısına (outputs) erişmene yarar.
  * Normal (tekil) Workflows’da genelde “`needs`” üzerinden job’ların çıktılarına bakarız; “`jobs`” genellikle Reusable Workflow senaryolarında kullanılır.



### 6) `steps`

* **Ne içerir?**
  * Şu anda içinde bulunduğun job’daki **step’lerin** durum ve çıktı (outputs) bilgileri.

```yaml
steps:
  - name: Step A
    id: my_step
    run: echo "result=hello" >> $GITHUB_OUTPUT

  - name: Use step output
    run: echo "Output is ${{ steps.my_step.outputs.result }}"
```



### 7) `runner`

* **Ne içerir?**
  * İş akışını çalıştıran runner hakkında bilgi: işletim sistemi, mimari, vb.
  * Örnek: `runner.os`, `runner.arch`.

```yaml
if: ${{ runner.os == 'Linux' }}
```

Bu koşul sadece Linux runner’da çalışsın istiyorsan kullanabilirsin.



### 8) `secrets`

* **Ne içerir?**
  * GitHub “Settings → Secrets” bölümünde tanımladığın gizli değerler (API anahtarları, parolalar vb.).

```yaml
env:
  MY_SECRET: ${{ secrets.MY_SECRET_KEY }}
run: echo "$MY_SECRET"
```

`secrets.MY_SECRET_KEY` ile sakladığın gizli değere erişirsin. Loglarda bu değer maskelenir.



### 9) `strategy`

* **Ne içerir?**
  * Eğer bir **matrix strategy** (matris yapısı) kullanıyorsan, `strategy` context’i ile matris detaylarına ulaşabilirsin.

```yaml
strategy:
  matrix:
    node: [14, 16, 18]
# ...
run: echo "Node version is ${{ matrix.node }}"
```

`matrix.node` → Her job varyantında ilgili Node sürümü.



### 10) `matrix`

* **Ne içerir?**
  * `strategy.matrix` ile tanımladığın parametre değerlerini taşıyan context.
  * Örneğin “`matrix.os`” veya “`matrix.node`” gibi parametreler.



### 11) `needs`

* **Ne içerir?**
  * Bu job’un **bağımlı olduğu** (önce çalışan) job’ların **outputs** değerleri.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      build_number: ${{ steps.build_info.outputs.number }}

  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build num: ${{ needs.build.outputs.build_number }}"
```

`needs.build.outputs.build_number` → “build” job’undan dönen output.



### 12) `inputs`

* **Ne içerir?**
  * Manuel tetiklenebilir (`workflow_dispatch`) Workflow’larda ya da Reusable Workflow’larda **kullanıcıdan alınan** input değerleri.

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        description: "Select environment"
        required: true
        options:
          - dev
          - staging
          - prod
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Env: ${{ github.event.inputs.environment }}"

```



#### Kısaca Nasıl Kullanırız?

* Bu context’lere, Workflow içindeki “expression syntax” ile, yani `${{ <context>.<property> }}` şeklinde erişirsin.
* Örneğin, `echo "Repo: ${{ github.repository }}"` diyerek console’a yazdırabilirsin.
* Koşul ifadelerinde `if: ${{ condition }}` şeklinde de kullanılır. Örneğin:

```yaml
if: ${{ secrets.MY_TOKEN != '' }}
```

Token boş değilse adım çalışsın.



### Özet

* **Context** = Workflow içinde pek çok meta bilgiyi saklayan objeler.
* `github` → Tetikleme, branch, event, aktör bilgisi
* `env` ve `vars` → Ortam değişkenleri (yerel veya global tanımlı)
* `runner` → Makine/OS bilgisi
* `secrets` → Gizli anahtarlar/parolalar
* `steps`, `jobs`, `needs` → Adımlar arası veya job’lar arası çıktı/bağımlılık bilgisi
* `matrix` ve `strategy` → Matris stratejisinde sürüm ve platform varyasyonları
* `inputs` → Kullanıcının manuel tetiklemede veya Reusable Workflow’da girdiği değerler

Bu context’lerle, GitHub Actions içinde hem condition’lar (if), hem değişken atamalar, hem de step’ler arası veri alışverişi gibi **dinamik** özellikler elde edersin.



{% embed url="https://docs.github.com/en/enterprise-server@3.10/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs" %}
