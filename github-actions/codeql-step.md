---
icon: square-code
---

# CodeQL Step

**CodeQL**, GitHub’ın sağladığı bir **kod analizi** ve **güvenlik taraması** aracıdır. Bu araç, depo içindeki potansiyel güvenlik açıklarını (SQL injection, XSS, vb.) **kod düzeyinde** tespit etmeye çalışır.&#x20;

```yaml
name: "CodeQL"

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 14 * * 1'
    # Bu ayarlar, "main" branch'e push veya PR açıldığında
    # ve her pazartesi 14:00'de (UTC) CodeQL analizini tetikler.

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['java', 'javascript', 'python']
        # Bu üç dil için ayrı alt-job'lar oluşturulur
        # ve paralel şekilde analiz yapılır.

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        # Kodları indirerek CodeQL'in erişmesine olanak sağlar.

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v1
        with:
          languages: ${{ matrix.language }}
        # CodeQL'i başlatırken hangi dili analiz edeceğini belirtiyor
        # (matrix'te tanımlanan "java", "javascript" veya "python").

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v1
        # Asıl kod analizi burada yapılır ve sonuçlar GitHub Security sekmesinde görünür.
```

### 1) Workflow Tetikleyicileri

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 14 * * 1'
```

* **push**: `main` branch’ine her push olduğunda CodeQL analizini çalıştırır.
* **pull\_request**: `main` branch’ine PR açıldığında da çalıştırır.
* **schedule (cron)**: Her pazartesi 14:00’de (`0 14 * * 1`) analiz çalışır. Bu şekilde düzenli taramalar yapılır.

### 2) Job Tanımı ve Strateji

```yaml
jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['java', 'javascript', 'python']
```

* **Job adı**: “analyze”.
* **Runner**: “ubuntu-latest” → GitHub’ın Ubuntu ortamı.
* **Matrix**: CodeQL’in **birden fazla dil** için tarama yapacağı belirtiliyor. Yani job, `java`, `javascript`, ve `python`dilleri için **paralelde** (ya da ardışık) çalışacak.
* **fail-fast: false**: Bir dilde analiz hata verse bile diğer dillerin analizini kesme; tamamlamaya çalış.

### 3) Adımlar (Steps)

1.  **Checkout repository**

    ```yaml
    - name: Checkout repository
      uses: actions/checkout@v2
    ```

    * Kodları runner’a indirir, böylece CodeQL taraması dosyaları görebilir.


2.  **Initialize CodeQL**

    ```yaml
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      with:
        languages: ${{ matrix.language }}
    ```

    * CodeQL analizini başlatmak için resmi GitHub Action (`github/codeql-action/init@v1`).
    * **`languages: ${{ matrix.language }}`**: Hangi dili (java, javascript, python) tarayacağını belirtiyor.


3.  **Perform CodeQL Analysis**

    ```yaml
    yamlCopy- name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v1
    ```

    * Asıl taramayı çalıştırır. Kodda potansiyel güvenlik veya kalite sorunlarını saptar.
    * Sonuçları GitHub Security tabında “Code scanning alerts” bölümünde görebilirsin.

### 4) Neden CodeQL?

* Potansiyel açıkları push/PR aşamasında yakalayarak yaygın zafiyetleri önlemiş olursun.
* **Çoklu Dil Desteği**: Java, JavaScript, Python, C/C++, Go, C# vb. popüler dilleri destekler.
* **Entegrasyon**: GitHub Security sekmesinde “code scanning alerts” olarak rapor çıkarır, pull request’lerde uyarı gösterir, hatta “Security tab” üzerinden düzeltmeler önerilir.

### 5) Cron Taramaları vs. Pull Request Taramaları

* `pull_request` tetiklemesi: Daha kod PR aşamasındayken sorunları yakalamayı sağlar.
* `push` tetiklemesi: Branch’e direkt push yapan geliştiriciler için de tarama çalışır.
* `schedule (cron)`: Periyodik olarak (ör. haftada bir) tam kapsamlı tarama yapar. Böylece sürekli güncellenen kurallarla eski kodu yeniden tarayabilirsin.

#### Özet

Bu **CodeQL** workflow, depo içindeki belirli dillerdeki (Java, JS, Python) kodu düzenli tarar ve güvenlik açıklarını raporlar. “init” adımında analiz ayarları yapılır, “analyze” adımında tarama gerçekleşir. Tetikleyiciler (push, PR, cron) sayesinde hem sürekli entegrasyon sırasında, hem de düzenli aralıklarla güvenlik kontrolleri yapılır.
