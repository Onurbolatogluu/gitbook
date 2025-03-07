---
icon: recycle
---

# Reusable Workflow & Reuse Templates for Actions and Workflows

GitHub Actions’ın “reusable workflow” (yeniden kullanılabilir iş akışı) ve “template” (şablon) özellikleri, özellikle kurumsal ortamlarda sık tekrar eden CI/CD (Continuous Integration/Continuous Delivery) iş akışlarını hızlıca oluşturmanızı ve paylaşmanızı sağlayan çok kullanışlı araçlardır.

### 1. Reusable Workflow Nedir?

**Reusable workflow**, bir GitHub Actions iş akışını (workflow) başka bir repodaki yahut aynı repodaki farklı bir workflow içinden tekrar kullanılabilir şekilde tanımlama imkânı sunar. Örneğin:

* Kurumsal ölçekte her projede kullanılan bir test veya build aşaması varsa bu aşamayı tekrar tekrar kopyalamak yerine tek bir “reusable workflow” oluşturup diğer projelerde onu “çağırmak” yeterli olur.
* Böylece bakımı kolaylaşır: Ortak iş akışında güncelleme gerektiğinde merkezî bir noktada düzenleme yapmanız tüm projelerde güncel iş akışının otomatik kullanılmasına izin verir.

#### Kısa Örnek

```yaml
# .github/workflows/reusable-ci.yml
name: Reusable CI

on: [workflow_call]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Echo from Reusable Workflow
        run: echo "Hello from Reusable CI!"
```

Bu reusable-ci.yml dosyası, `workflow_call` event’ine yanıt verecek şekilde tanımlanır. Böylece başka bir repo ya da workflow içinden şu şekilde çağırabilirsiniz:

```yaml
# .github/workflows/main-ci.yml
name: Main CI

on: [push]

jobs:
  call-reusable:
    uses: <sahip/kullanılabilir repo yolu>/.github/workflows/reusable-ci.yml@main
    with:
      # Gerekirse input parametreleri
    secrets:
      # Gerekirse secret değişkenleri
```

Bu sayede `reusable-ci.yml` dosyası tek bir yerde tanımlı olur, ancak farklı projeler veya farklı iş akışları onu kullanabilir.



### 2. Workflow Templates (Enterprise Özelliği)

“Workflow Templates”, GitHub Enterprise hesabınızda belirli şablonları _kurumsal_ tüm repo sahiplerinin kullanımına açmak için tasarlanmıştır. Yani, reusable workflow tek bir repo içinde kullanılırken, **workflow template** ise bir “.github” _kurumsal_ deposu içinde tanımlanır ve tüm diğer repolarda bu şablon iş akışını baştan oluşturma fırsatı sunar.

#### Genel Mantık

1. Enterprise hesabına ait, genelde “.github” adlı bir repo bulunur. Bu depo içindeki `workflow-templates` klasörüne bir `.yml` (iş akışı) ve `.json` (metadata) dosyası yüklenir.
2. Hem `workflow.yml` hem de `workflow.json` aynı isimde olmalıdır (örneğin: `octo-org-workflow.yml` ve `octo-org-workflow.json`).
3. **Enterprise Üyeleri**: Bu şablonu kendi reposuna eklemek istediğinde, GitHub arayüzü üzerinden “Actions” bölümünde yeni bir workflow eklerken bu şablonu görebilir.

#### Örnek Dosya Yapısı

```
.github
└── workflow-templates
    ├── octo-org-workflow.yml
    └── octo-org-workflow.json
```

**octo-org-workflow.yml** (örnek iş akışı içeriği):

```yaml
name: Octo Organization CI

on:
  push:
    branches: [$default-branch]
  pull_request:
    branches: [$default-branch]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Run a one-line script
        run: echo Hello from Octo Organization
```

**octo-org-workflow.json** (metadata dosyası):

```json
{
  "name": "Octo Org Workflow",
  "description": "Octo Org CI workflow template.",
  "iconName": "example-icon",
  "categories": [
    "Go"
  ],
  "filePatterns": [
    "package.json$",
    "^Dockerfile",
    ".*\\.md$"
  ]
}
```

* **name** ve **description**: UI (kullanıcı arayüzü) üzerinde şablonun ismi ve kısa tanımı.
* **iconName**: UI’da bu şablona ait simge ismi (GitHub’ın kendi belirli ikon setinden).
* **categories**: Projenin türüne ilişkin etiketler.
* **filePatterns**: Hangi dosyalar varsa bu şablonun önerileceğini gösteren bir filtre listesi. Örneğin reposunda `package.json` veya `Dockerfile` gibi dosyalar bulunan bir kullanıcı, yeni workflow oluştururken bu şablon önerilir.

> **Not**: Bu Workflow Templates özelliği, _Enterprise Cloud_ veya benzer üst seviye planlarda aktifleşir.

### 3. Adım Adım Kurulum ve Kullanım

1. **Kurumsal .github deposu oluşturma**: Eğer hali hazırda yoksa, Enterprise hesabınıza ait bir “.github” adlı _publi&#x63;_&#x76;eya _internal_ repo oluşturulur (genelde public öneriliyor).
2. **workflow-templates klasörü ekleme**: Depo içinde `workflow-templates` klasörü oluşturun.
3. **Şablon dosyalarını ekleme**: `my-workflow.yml` ve `my-workflow.json` şeklinde aynı isimde iki dosya ekleyin.
4. **Enterprise kullanıcılarına erişim izni verme**: Bu depo okunabilir olması gerektiği için enterprise kullanıcılarına read veya write izni verilmiş olduğundan emin olun. (Örneğin, “internal” olarak ayarlamak çoğu zaman işinizi görür.)
5. **Template’i kullanma**: Artık enterprise kullanıcıları kendi repo’suna girip Actions bölümünde “New workflow” butonuna tıkladığında, bu şablonu listede görebilir.

