---
icon: circle-chevron-right
---

# Expressions - 1

```yaml
name: example-workflow
on: [push]

jobs:
  demo-job:
    runs-on: ubuntu-latest
    steps:
      - name: Step 1 - Her Zaman Çalışır
        run: echo "Step 1 started"

      - name: Step 2 - Branch 'feature-' ile Başlarsa
        if: ${{ startsWith(github.ref, 'refs/heads/feature-') }}
        run: echo "Bu step, feature- ile başlayan branch'lerde çalışır"

      - name: Step 3 - Commit Mesajında 'CI-TEST' varsa
        if: ${{ contains(github.event.head_commit.message, 'CI-TEST') }}
        run: echo "Commit mesajında 'CI-TEST' var!"

      - name: Step 4 - Önceki Adımlar Başarılıysa
        if: ${{ success() }}
        run: echo "Önceki adımlar HATA almadan tamamlandıysa burası çalışır"

      - name: Step 5 - Önceki Adımlar Hata Aldıysa
        if: ${{ failure() }}
        run: echo "Önceki adımların en az biri hata aldıysa burası çalışır"

      - name: Step 6 - Negatif (Ters) Koşul Kullanımı (!)
        if: ${{ !startsWith(github.ref, 'refs/tags/') }}
        run: echo "Eğer push tag ile tetiklenmediyse (tags ile başlamıyorsa) bu step çalışır"

```

* **Step 1 (Her Zaman Çalışır)**
  * Koşul olmadığı için push tetiklendiğinde mutlaka çalışır.
  * “echo” komutuyla basit bir çıktı verir.
* **Step 2 (Branch ‘feature-’ ile Başlarsa)**
  * `startsWith(github.ref, 'refs/heads/feature-')` ifadesi, branch adının `refs/heads/feature-...` biçiminde mi başladığını kontrol eder.
  * Eğer branch adı “feature-” ile başlamıyorsa adım atlanır.
* **Step 3 (Commit Mesajında ‘CI-TEST’ Varsa)**
  * `contains(github.event.head_commit.message, 'CI-TEST')` ifadesi, son commit mesajının içinde “CI-TEST” kelimesi geçip geçmediğine bakar.
  * Commit mesajı “CI-TEST” içeriyorsa adım çalışır, içermiyorsa adım atlanır.
* **Step 4 (Önceki Adımlar Başarılıysa)**
  * `success()` fonksiyonu, o ana kadarki adımların tamamının hatasız bittiğini gösterir.
  * Önceki adımlardan biri bile hata (fail) alsa bu adım çalışmaz.
* **Step 5 (Önceki Adımlar Hata Aldıysa)**
  * `failure()` fonksiyonu, bir önceki adım veya adımlardan herhangi birinin hata aldığını ifade eder.
  * Eğer hata durumu yoksa bu adım atlanır.
* **Step 6 (Negatif Koşul, Tag Olmadığında)**
  * `!startsWith(github.ref, 'refs/tags/')` ifadesi, push işleminin “refs/tags/” ile başlayıp başlamadığını tersine (negation) çevirerek sorgular.
  * Tag push’u değilse adım çalışır, tag push’u ise adım atlanır.

