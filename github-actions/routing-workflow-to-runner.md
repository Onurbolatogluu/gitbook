---
icon: arrow-progress
---

# Routing workflow to runner

Bu konu, özellikle **self-hosted runner** kullanırken hangi runner’ın job’u alacağına karar vermek için önemlidir.

### 1) Self-Hosted Runner Otomatik Etiketleri (Default Labels)

* **self-hosted**: Tüm self-hosted runner’larda otomatik bulunur.
* **linux, windows, macOS**: Runner’ın işletim sistemi.
* **x64, ARM, ARM64**: Donanım mimarisine göre eklenen etiketler.

```yaml
runs-on: [self-hosted, linux, ARM64]
```

Bu job, **linux + ARM64** etiketlerine sahip bir self-hosted runner’da çalışır. Runner etiketlerinin **tümü** uyuşmalı (mantık “AND”).

### 2) Custom Labels (Ör. gpu)

* Kendin özel etiketler ekleyebilirsin (ör. “gpu”, “ssd”, “high-memory”, vb.).
* GitHub Actions arayüzünden veya CLI ile runner’a eklenen bu etiketler, job’u hangi runner’ın alabileceğini daha spesifik hale getirir.

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

Bu job, “gpu” etiketine sahip x64 Linux runner’ını arar.

### 3) Runners Groups

* **Runner groups**, özellikle **organization** veya **enterprise** düzeyinde, benzer runner’ları bir arada yönetme yöntemidir.
  * Örneğin “ubuntu-runners” adında bir grup oluşturup, Ubuntu tabanlı self-hosted runner’ları içine koyabilirsin.
* **Neden?**
  * Güvenlik sınırları oluşturmak (bazı projeler sadece bir grup runner’a erişebilsin).
  * Farklı departman/proje runner’larını ayrı gruplarda tutmak.

```yaml
jobs:
  example:
    runs-on:
      group: ubuntu-runners
      labels: ubuntu-20.04-16core
    steps:
      ...

```

Bu job, “ubuntu-runners” adlı runner grubunda olup aynı zamanda “ubuntu-20.04-16core” etiketine sahip bir runner’a yönelir.

### 4) Labels ve Groups Birlikte

* Bir job için hem **`group:`** hem de **`labels:`** verebilirsin.
* Runner’ın bulunduğu **grup** + istenen **etiketleri** **birlikte** sağlaması gerekir.
* **Mantık**:
  * Runner önce “ubuntu-runners” grubunda olmalı,
  * Sonra “ubuntu-20.04-16core” etiketlerine de sahip olmalı.

```yaml
name: learn-github-actions
on: [push]
jobs:
  check-bats-version:
    runs-on:
      group: ubuntu-runners
      labels: ubuntu-20.04-16core
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

### 5) Özet ve Önemli Noktalar

1. **Self-Hosted Runner’larda Etiket (Label) Sistemi**
   * Varsayılan: “self-hosted”, OS, mimari.
   * Özel (custom): dilediğin isim.
   * Job, `runs-on: [label1, label2]` yazınca bu etiketlerin tümüne sahip runner aranır.
2. **Runner Groups**
   * Enterprise veya Team/Organization planlarında, runner’ları gruplara ayırabilirsin.
   * “group:” parametresiyle job’u belli bir grup runner’a yönlendirebilirsin.
3. **Mantık “AND”**
   * Bir job, hem belirli “labels” hem de “group” yazılmışsa, runner’ın hem doğru grupta hem de o etiketlere sahip olması gerekir.
4. **Neden Gerekli?**
   * Farklı donanım/OS özellikli runner’ları ayırt etmek (GPU, ARM, Windows vs.).
   * Güvenlik ve erişim kontrolü (bazı runner’lar sadece özel projeler için).
   * Daha iyi dağıtım (büyük bir self-hosted runner havuzunda job’u doğru yere yönlendirmek).

Bu şekilde, **etiket** ve **grup** kullanımını kombine ederek job’ları spesifik runner’lara route (yönlendirme) edebilirsin.
