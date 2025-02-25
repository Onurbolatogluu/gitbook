---
icon: scroll-old
---

# Add Script to workflow

**GitHub Actions** içerisinde **kendi bash script’lerini** (veya benzer komut dosyalarını) **nasıl çalıştırabileceğin** üzerine odaklanıyor. Normalde “run:” satırlarıyla tek satırlık veya birkaç satırlık komutları çalıştırabiliyorsun; fakat uzun veya karmaşık işlemler için bunları `.sh` gibi script dosyalarında tutmak daha düzenli olur. GitHub Actions, bu script’leri **repo içinde** saklamana ve iş akışı sırasında çağırmana imkan tanır.

```yaml
jobs:
  example-job:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./scripts
    steps:
      - name: Check out the repository to the runner
        uses: actions/checkout@v4
      
      - name: Run a script (in ./scripts)
        run: ./my-script.sh

      - name: Run another script (in ./scripts)
        run: ./my-other-script.sh

      - name: Different Working Dir
        run: ls -la
        working-directory: ../

      - name: Run script with direct path
        run: ../other-scripts/some-other.sh
        # Burada "scripts" klasöründen bir üst dizine (../) çıkıp
        # "other-scripts" klasörüne girerek "some-other.sh" dosyasını çalıştırıyoruz.
```



### 1) `runs-on: ubuntu-latest`

* Bu, job’un Ubuntu işletim sisteminde çalışan bir runner üzerinde çalışacağını belirtir.
* Dolayısıyla `.sh` (bash) komut dosyaları varsayılan olarak desteklenir.

### 2) `defaults.run.working-directory: ./scripts`

* `defaults.run` bloğu ile, job içerisindeki tüm `run:` satırlarının **varsayılan çalışma dizinini** (`working-directory`) ayarlıyorsun.
* Burada `./scripts` dendiği için, **repo’nun checkout** edildikten sonraki kök dizininde bulunan “scripts” klasörüne gidiliyor.
* Yani “run: ./my-script.sh” gibi komutlar, `scripts` klasöründen çağrılmış oluyor.

### Sık Karşılaşılan Sorunlar

* **İzinler**: Script’e çalıştırma izni (executable permission) verilmemişse (`chmod +x my-script.sh`), script çalışmaz. Genellikle repo’da commit’lerken yürütme iznini korumak gerekir.
* **Yanlış Path**: “`defaults.run.working-directory`” yoksa, script için tam yolu yazman (`./scripts/my-script.sh`) veya `cd scripts` komutu kullanman gerekir.
* **Shebang**: Script dosyası başında `#!/usr/bin/env bash` gibi bir shebang satırı kullanman önerilir (özellikle farklı shell’lerde tutarlılık için).

### Özet

1. **Checkout** step’iyle repo kodunu indir.
2. `defaults.run.working-directory: ./scripts` diyerek job içindeki tüm komutların “scripts” klasöründe çalışmasını sağla (isteğe bağlı).
3. `run: ./my-script.sh` diyerek script’ini çağır.
4. Script dosyası içinde istediğin karmaşık komutları saklayabilir, bakımını kolaylaştırırsın.

Böylece “**Add Script to Workflow**”, yani GitHub Actions’da `.sh` (veya diğer dillerde) script dosyalarını iş akışında kullanma konusu netleşmiş oluyor.



{% embed url="https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/adding-scripts-to-your-workflow" %}

