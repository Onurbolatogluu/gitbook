---
icon: shield-check
---

# Dependent Jobs

GitHub Actions’da **Dependent Jobs**, bir job’un başka bir job’un (veya job’ların) tamamlanmasını bekledikten sonra çalışması anlamına gelir. Böylece iş akışında varsayılan “**paralel çalışma**” davranışını değiştirip, **sıralı** veya **kademeli** bir yapı oluşturabilirsin. Bunun için temel olarak:

* **`needs`** özelliğini kullanırsın:
  * `jobs.<job_id>.needs: <önceki_job_adı>` veya `[önceki_job1, önceki_job2, ...]` şeklinde yazıldığında, bu job ancak ihtiyaç duyduğu job’lar bitince başlar.
  * Ayrıca, `needs` ile belirttiğin job’ların **başarılı** sonuçlanması gerekir (varsayılan olarak `success()`), aksi halde dependent job tetiklenmez.
* **`if`** ile ek koşullar ekleyebilirsin:
  * `jobs.<job_id>.if: ${{ success() }}` gibi bir ifade, bu job’un ancak tüm dependent job’lar hatasız biterse çalışmasını sağlar.
  * `if: ${{ always() }}` dersen, bağımlı job’lar hata alsa bile (fail olsa bile) yine bu job’u çalıştırabilirsin. Örneğin cleanup aşamalarında kullanılır.

#### Nasıl Çalışır?

* **Varsayılan Paralellik**: GitHub Actions, aynı Workflow içindeki job’ları **birbirinden bağımsız** (parallel) çalıştırır.
* **Sıralı İlerleme**: `needs` ekleyerek, “job2, job1 bitmeden başlamasın” gibi bir sıra tanımı yaparsın. Örnek:

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job 1"

  job2:
    needs: job1    # job1 bitmeden job2 başlamaz
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job 2"
  job3:
    needs: [job1, job2]  # job3, job1 ve job2 tamamlanınca başlar
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job 3"

```

Bu senaryoda:

* `job1` doğrudan başlar (çünkü bir bağımlılığı yok).
* `job2` ancak `job1` tamamlanınca (success) başlayabilir.
* `job3` de hem `job1` hem `job2` tamamlanınca devreye girer.

#### “if” Koşulları

`needs` ile belirttiğin job’ların sonucuna göre daha ince ayarlar da yapabilirsin:

**Sadece Başarılı Sonuçta**:

```yaml
if: ${{ success() }}
needs: job1
```

* `success()` → Dependent job’(lar) başarıyla bitmişse `true`.
* Başarısız (`fail`) olsa step hiç çalışmaz.

**Her Zaman Çalıştır**:

```yaml
if: ${{ always() }}
needs: job1
```

* Bağımlı job hata alsa bile bu job devreye girer. Mesela hataları temizlemek, log göndermek vs. için kullanılır.

**Belirli Bir Koşulla**:

```yaml
if: ${{ needs.job1.outputs.some_value == 'ok' }}
needs: job1
```

* Burada “job1”’in ürettiği bir output değerine bakarak, job2’yi koşturup koşturmamaya karar verebilirsin.

#### Özeti

* **`needs`**: Job’ları birbirine **bağımlı** hale getirir, sıralı çalıştırma sağlar.
* **`if`**: Ek koşullar ekleyip, “bağımlı job fail olsa da/olmasa da” senaryolarını yönetmeye yarar.
* Bu sayede CI/CD sürecinde, “Önce `build` → Sonra `test` → Sonra `deploy`” gibi aşamalı bir akış oluşturabilirsin veya test başarısızsa deploy’u otomatik olarak iptal edebilirsin.

{% hint style="info" %}
GitHub Actions’da iki önemli “katman” vardır:

1. **Job’lar (jobs)**: Aynı workflow içinde tanımlanan bölümler.
2. **Adımlar (steps)**: Bir job’un içindeki, sırasıyla yürütülen komut veya eylemler.\


**Adımlar (steps) her zaman sırayla (top-down) çalışır**, yani job içindeki Step 1 → Step 2 → Step 3 şeklinde. Fakat **job’lar** (örneğin `job1`, `job2`, `job3`), aynı workflow içinde paralel olarak başlarlar. Yani “job1” bitene kadar “job2” otomatik beklemeye girmiyor; GitHub ikisini aynı anda koşturmaya çalışır.
{% endhint %}

