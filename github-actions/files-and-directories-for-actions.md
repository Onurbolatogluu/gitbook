---
icon: files-medical
---

# Files and Directories for Actions

**“Files and Directories for Actions”** konusu, GitHub Actions’da **kendi aksiyonunu** (custom action) oluştururken **nasıl bir klasör yapısı (directory structure)** kullanman gerektiğini ve **hangi dosyaların** zorunlu olduğunu açıklar.&#x20;

1. **JavaScript Action** (Node.js tabanlı)
2. **Docker Container Action** (Dockerfile + entrypoint ile)

### 1) JavaScript Action: Dosya ve Klasör Yapısı

#### Örnek Dizilim

```
/my-js-action
├── action.yml        # Aksiyonun tanımlama (metadata) dosyası
├── index.js          # Aksiyonun asıl kodu (JS)
├── package.json      # NPM bağımlılıklarını tanımladığın dosya
├── node_modules/     # (install sonrası oluşabilir)
└── README.md         # Nasıl kullanılır, açıklama
```

1. **action.yml**
   * Aksiyonun **inputs** (girdi parametreleri), **outputs** (çıktı değerleri) ve `runs` (hangi ortamda/ana dosyada çalışacağı) gibi metadata’yı içerir.
   *   Örnek:

       ```yaml
       name: "My JavaScript Action"
       description: "A description of your action"

       inputs:
         my-input:
           description: "Input to use in the action"
           required: true
           default: "default input value"

       outputs:
         my-output:
           description: "Output from the action"

       runs:
         using: "node16"
         main: "index.js"
       ```
2. **index.js**
   * Action çalıştığında bu Node.js dosyası çalışır.
   * `@actions/core` gibi paketler kullanarak `getInput()`, `setOutput()` yapabilirsin.
   *   Mesela:

       ```js
       const core = require('@actions/core');
       async function run() {
         try {
           const myInput = core.getInput('my-input');
           // ...some logic...
           core.setOutput('my-output', 'some-value');
         } catch (err) {
           core.setFailed(err.message);
         }
       }
       run();
       ```
3. **package.json** ve **node\_modules**
   * Action’ın JS bağımlılıklarını (ör. `@actions/core`, `@actions/github`) listelediğin NPM dosyası.
   * “npm install” sonrası `node_modules` klasörü oluşur.
   * Bu klasör isteğe bağlı olarak commit edilebilir (actions require bundling sometimes), ya da bundling / ncc kullanarak tek dosyaya da paketleyebilirsin.
4. **README.md**
   * Action’ın nasıl kullanılacağı, hangi `inputs`/`outputs` olduğu, sürüm bilgileri gibi dokümantasyon.

#### Özet (JavaScript Action)

* **action.yml** → En önemli metadata.
* **index.js** → Ana çalışma dosyası. Node.js kodun burada.
* **package.json** → Bağımlılık yönetimi.
* Bu yapı, Node tabanlı (platform bağımsız) action geliştirmeye imkan verir.



### 2) Docker Container Action: Dosya ve Klasör Yapısı

#### Örnek Dizilim

```
/my-docker-action
├── Dockerfile        # Action'ın environment ve komutları
├── entrypoint.sh     # Container çalışınca ilk çalışan script (örnek)
├── action.yml        # Metadata (inputs, outputs, runs: docker)
└── README.md
```

1. **Dockerfile**
   * İçinde “FROM ubuntu:latest”, “RUN apt-get install…” vb. satırlar olur.
   * Action çalışırken GitHub runner, bu Dockerfile’dan bir imaj oluşturur (veya cached). Sonra container’ı çalıştırır.
2. **entrypoint.sh**
   * Container başlarken hangi script’in çalışacağını belirlersin (ENTRYPOINT or CMD).
   *   Örneğin:

       ```sh
       #!/bin/bash
       set -e
       echo "My Docker action is running..."
       # Read inputs from env variables, do stuff, output results
       ```
   * “action.yml” dosyasında “args: \[ ... ]” diyerek `inputs` parametrelerini bu script’e aktarabilirsin.
3.  **action.yml** (Docker Sürümü)

    ```yaml
    name: "My Docker Action"
    description: "A description of what your action does"

    inputs:
      my-input:
        description: "Input to use in the action"
        required: true

    outputs:
      my-output:
        description: "Outputs of the action"

    runs:
      using: "docker"
      image: "Dockerfile"
      args:
        - ${{ inputs.my-input }}
    ```

    * “using: docker” diyerek Docker Action olduğunu belirtirsin.
    * “image: Dockerfile” → Bu Dockerfile bu action’ın imajını inşa eder.
    * “args: \[ $\{{ inputs.my-input \}} ]” → Bu girdi parametresini container’a komut satırı argümanı olarak iletir.
4. **README.md**
   * Açıklama: “Bu Docker action şu paketleri kuruyor, şu parametreleri alıyor.”

#### Özet (Docker Container Action)

* **action.yml** → “using: docker” + “image: Dockerfile”.
* **Dockerfile** → Ortam ve bağımlılıklar.
* **entrypoint.sh** (ya da benzer script) → Container çalıştığında asıl kod.
* Sadece **Linux runner** üstünde çalışır.

### Sonuç

“**Files and Directories for Actions**” şöyle özetlenebilir:

1. **action.yml** → Tüm action türlerinde (JS, Docker, Composite) kullanılan **metadata** dosyası. Inputs, outputs, runs.
2. **Ana Kod Dosyası** →
   * JavaScript Action → `index.js` veya benzer.
   * Docker Action → `Dockerfile` + `entrypoint.sh`.
   * Composite Action → Sadece `.yml` step tanımları.
3. **package.json, node\_modules** (JS Action’da) veya **Dockerfile** (Docker Action’da) ya da step’ler (Composite) eklenir.
4. **README.md** → Kullanım dokümantasyonu.
