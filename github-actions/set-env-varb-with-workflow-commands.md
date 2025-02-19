---
icon: terminal
---

# Set Env Varb with Workflow Commands

GitHub Actions’da **$GITHUB\_ENV** adlı özel bir dosyaya yazarak, **runtime** adımlar arasında **dinamik environment değişkeni** paylaşabilirsin. Bu yöntem, sabit YAML tanımlarından farklı olarak “script” içinde hesapladığın veya dış kaynaktan okuduğun değerleri sonraki adımlara aktarır.&#x20;

### 1) $GITHUB\_ENV Nedir?

*   “`$GITHUB_ENV`”, bir **özel dosya** referansıdır.



Bir Step içerisinde,

```yaml
echo "MY_VAR=some_value" >> $GITHUB_ENV
```

diye yazarak, **sonraki adımlar** için “MY\_VAR” adlı bir environment değişkeni atamış olursun.

Yani “workflow command” dediğimiz bu “echo … >> $GITHUB\_ENV” yapısı, **CI/CD hattı boyunca** env değişkenleri eklemeni/üzerinde oynamanı sağlar.



### 2) Neden Kullanılır?

* **Step’ler Arası Veri Aktarmak**: Bir step içindeki script, dinamik bir değer üretir; bunu `$GITHUB_ENV` yoluyla kaydedip sonraki step’te kullanabilirsin.
* **Koşullu ve Dinamik Konfigürasyon**: Örneğin “test sonu başarısına göre bir bayrak set et” gibi mantıklar kurgulayabilirsin.
* **Sabit YAML env**’lerine göre daha esnek. Çünkü workflow dosyasında kodlaman gerekmeden, anlık hesaplanan veya dış API’den alınan değerleri paylaşabilirsin.

### 3) Basit Örnek

```yaml
name: Set Environment Variables Example
on: [push]

jobs:
  setup-and-use-env:
    runs-on: ubuntu-latest
    steps:
      - name: Set dynamic environment variable
        run: |
          echo "DYNAMIC_VAR=Hello from GitHub Actions" >> $GITHUB_ENV

      - name: Use the environment variable
        run: |
          echo "The value of DYNAMIC_VAR is: $DYNAMIC_VAR"
```

* **Step 1** (“Set dynamic environment variable”):
  * `echo "DYNAMIC_VAR=Hello from GitHub Actions" >> $GITHUB_ENV` → DYNAMIC\_VAR’ı sonraki adımlar için tanımladı.
* **Step 2** (“Use the environment variable”):
  * `echo "The value of DYNAMIC_VAR is: $DYNAMIC_VAR"` → Step 1’de set edilen değişkeni okur ve terminale basar.

### 4) Dikkat Edilecekler

* `$GITHUB_ENV` üzerinden set ettiğin değişken, **o job’un geri kalan step’lerinde** kullanılabilir. Farklı job’larda otomatik paylaşılmaz. (Job’lar arası veri paylaşımı için “outputs” veya “needs” mekanizmasına bakabilirsin.)
* Sonraki step’te kullanılabilmesi için, önceki step’in tamamen bitmiş (success/fail) olması gerekir.
* Daha önce tanımlanmış bir değişkeni `$GITHUB_ENV` ile yeniden yazarsan, yeni değeri override eder.

{% hint style="info" %}
**Kapsam**: Aynı job içindeki **ilerleyen adımlar**.
{% endhint %}



