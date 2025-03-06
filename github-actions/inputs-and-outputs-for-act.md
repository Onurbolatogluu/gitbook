---
icon: caret-right
---

# Inputs and Outputs for act

GitHub Actions’da **kendi oluşturduğun bir aksiyonu** (yani “Custom Action”’ı) belirli bir **metadata** dosyasıyla tanımlarsın (genelde `action.yml` adında). Bu dosyada:

1. **inputs** (girdiler):
   * Action’ı kullanmak isteyen birisi, hangi parametreleri verebileceğini gösterir.
   * Bu girdiler, action çalışırken içindeki kod tarafından **kullanılır.**
2. **outputs** (çıktılar):
   * Action çalıştığında üretebileceği sonuçları tanımlar.
   * Başka step’ler veya job’lar, bu çıktıları “`steps.<action-id>.outputs.<output-name>`” şeklinde okuyarak **kullanabilir**.
3. **runs**:
   * Aksiyonunun hangi **teknoloji/tür** üzerine çalıştığını belirtir. Örneğin:
     * “`using: node20`” diyerek bunun bir **JavaScript action** olduğunu ve Node.js 20 ile çalıştığını söylersin,
     * “`using: docker`” diyerek Docker container tabanlı bir action kullandığını belirtirsin,
     * “`using: composite`” diyerek YAML step’lerden oluşan bir “composite action” olduğunu söylersin.

Kısacası, **“inputs ve outputs”** action’ın parametrelerini ve üreteceği verileri tanımlarken, **“runs”** ise action’ın hangi ortamda veya hangi giriş noktasıyla (ör. `main.js`) çalıştığını belirler.

#### Kısaca Örnek Action Dosyası (action.yml)

```yaml
name: "Draw Octocats"

description: "Draws a certain number of Octocats with specified eye color"

inputs:
  num-octocats:
    description: "Number of Octocats"
    required: false
    default: "1"
  octocat-eye-color:
    description: "Eye color of the Octocats"
    required: true

outputs:
  sum:
    description: "The sum of the inputs"

runs:
  using: "node20"
  main: "main.js"
```

* Burada tanımlanan **`inputs`** ve **`outputs`** action’ı kullanan workflow’larda parametre geçebilmeni, sonuç döndürebilmeni sağlar.
* “`runs`” kısmı action’ın türünü (JavaScript) ve giriş noktasını (`main.js`) belirtiyor.

***

### Sonuç

1. **Inputs**: Action’ın **neyi beklediğini** (kullanıcıdan girdi parametreleri)
2. **Outputs**: Action’ın **ne sonuç üretebileceğini**
3. **runs**: Action’ın hangi ortamda / hangi dosyayla çalıştığını tanımlarsın.

Böylece GitHub Actions’da **kendi custom action**’larını oluştururken, parametre alma ve geri sonuç döndürmeyi ayarlayarak **esnek** ve **yeniden kullanılabilir** bir aksiyon elde edersin.
