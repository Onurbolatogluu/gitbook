---
icon: arrow-down-to-arc
---

# Runners

**GitHub Runners**, GitHub Actions iş akışlarının (workflow’ların) **hangi makine/ortam** üzerinde çalışacağını belirler. Yani, bir job çalıştırıldığında aslında “runner” adı verilen bir sunucu ya da bilgisayar (sanal ortam) üzerinde adım adım komutların yürütülmesi gerçekleşir.

### 1) Runner Seçimi: `runs-on`

* YAML dosyanda `jobs.<job_id>.runs-on: <runner_type>` şeklinde belirttiğin kısım, job’un hangi işletim sistemi / ortamda çalışacağına karar verir.
  * Örnek: `runs-on: ubuntu-latest`, `runs-on: windows-latest`, `runs-on: macos-latest`

```yaml
name: Multi-label example
on: [push]

jobs:
  build-linux-x64:
    runs-on:
      - self-hosted
      - linux
      - x64
    steps:
      - name: Build
        run: echo "Building on a self-hosted runner that is Linux and x64"
```

* İstersen bir liste (array) olarak da verebilirsin. Ancak bu durumda runner, listedeki tüm etiketlere uyan bir runner bulmalıdır. (Örneğin `[self-hosted, linux, x64]` gibi.)

### 2) GitHub-Hosted Runner

* **GitHub’ın sağladığı hazır ortamlar**dır. Sen `ubuntu-latest`, `windows-latest` gibi bir değer yazdığında, GitHub senin için otomatik olarak bir sanal makine oluşturur, iş akışı bittiğinde de o makineyi kapatır.
* Avantajları:
  * **Kurulum / bakım gerektirmez**. Tüm yönetimi GitHub yapar.
  * **Otomatik ölçeklendirir** (scaling): Talep arttığında yeni makineler ayarlayabilir.
  * **Üç ana işletim sistemi** (Ubuntu, Windows, macOS) varyantlarını sunar.
* Dezavantajları:
  * Konfigürasyon ve işletim sistemi üzerinde kısıtlı kontrolün var. (Örneğin, root erişimi var ama custom kernel vb. kısıtlı.)
  * Kullanım limitleri (dakika/ay) veya ek ücretlendirmeler olabilir (özellikle özel planlarda).
  * Ortam paylaşımlı olduğundan, bazen belirli güvenlik / IP adresi / özel network ihtiyaçlarında sınırlamalar olabilir.

### 3) Self-Hosted Runner

* **Kendi sunucun** (fiziksel makine, VM, hatta Raspberry Pi vb.) üzerine GitHub Actions runner uygulamasını kurarsın. Bu makine GitHub ile haberleşir ve Actions iş akışlarını orada çalıştırırsın.
* Avantajları:
  * **Tam denetim**: Kullanacağın donanım, yazılım, OS sürümü tamamen sana bağlı.
  * **Özel ağ / veri tabanlarına** doğrudan erişim: Şirket içi kaynaklara ya da izole bir ağa erişmek istiyorsan, bu runner’ı o ağa koyabilirsin.
  * Performansını, bellek, disk, CPU gibi kaynakları istediğin gibi ölçeklendirebilirsin.
* Dezavantajları:
  * **Bakım ve güvenlik** sorumluluğu sende. Makinenin güncellenmesi, güvenlik yamaları vb. senin görevin.
  * GitHub, o makineyi yönetmiyor; yani ek kapasite gerektiğinde otomatik oluşturmaz. Kendin ayarlamak zorundasın.

### 4) Standart Boyut vs. Larger Runners (GitHub-Hosted)

* **Standart Boyut**: Ücretsiz planların sunduğu normal sanal makineler. Normal bellek/CPU kaynaklarıyla gelirler.
* **Larger** (daha büyük makineler) veya “premium” runner seçenekleri, Team ya da Enterprise planlarda sunulur. Daha fazla RAM, CPU, disk alanı, sabit IP gibi özellikleri olabilir.
  * Büyük runner’lar genelde **yüksek kaynak** isteyen (çok parallel testler, büyük build’lar vb.) projelerde kullanılır.

### 5) Koşullar, Kotalar ve Özelleştirme

* **Koşullar / Kotalar**:
  * GitHub Hosted Runner’larda dakikalık kullanım limitin olabilir (ücretsiz planlarda sınırlı, ücretli planlarda daha yüksek).
  * Self-hosted runner’da GitHub’ın dakika limiti yoktur; ama kendi altyapı maliyetin devreye girer.
* **Özelleştirme**:
  * GitHub Hosted Runner’lar “hazır” ortamda belirli yazılımlar (Node, Python, Java, vs.) kurulu gelir.
  * Self-hosted ise tamamen istediğin yazılımı / sürümü kurmakta özgürsün.

#### Özetle,

* **GitHub-Hosted**: Kolay kurulum, anında ölçeklendirme, üç temel işletim sistemine destek, paylaşımlı/standart ortamlara uygun.
* **Self-Hosted**: Tüm kontrol ve bakım sende; özel ağ, özel donanım ve ihtiyaçlar varsa ideal.

Workflow’larda “**`runs-on: ubuntu-latest`**” ya da “**`runs-on: self-hosted`**” diyerek job’un hangi runner’da çalışacağını belirliyorsun. İhtiyacına ve projenin gereksinimlerine göre bu seçimi yaparsın.



