---
icon: binary-lock
---

# Encrypted Secrets

<figure><img src="../.gitbook/assets/Screenshot 2025-02-18 at 14.36.21.png" alt="" width="558"><figcaption></figcaption></figure>

GitHub Actions’da **Encrypted Secrets**, repo (veya organization ya da environment) düzeyinde tanımlanan **gizli bilgiler**dir. API anahtarları, parolalar, tokenlar gibi hassas verileri koduna veya log’lara açıkça eklemeden, güvenli şekilde kullanmanı sağlarlar.

### 1) Encrypted Secret Nedir?

* “Secret” olarak tanımladığın değerler (ör. `MY_TOKEN`, `DB_PASSWORD`), GitHub tarafından şifreli saklanır.
* Workflow içinde `secrets.<SECRET_NAME>` ifadesiyle bu değere erişebilirsin:

```yaml
env:
  MY_SECRET_TOKEN: ${{ secrets.MY_SECRET_TOKEN }}
```

* Log’lar veya UI aracılığıyla bu secret’ın içeriği asla düz metin halinde gösterilmez; maskelenmiş biçimde (`***`) çıkar.

### 2) Secret İsimlendirme Kuralları

* Alfasayısal karakterler ve alt tire (`_`) kullanılabilir; boşluk, özel karakter yok. Örneğin: `HELLO_world123`.
* `GITHUB_` ile **başlamamalı** (rezerv).
* Rakamla **başlamamalı**.
* Büyük-küçük harfe duyarsız (case-insensitive) ama girerken tutarlı olmak önerilir.
* Aynı düzeyde benzersiz olmalı (aynı isimde bir secret tekrar eklenemez).

### 3) Secrets Nasıl Tanımlanır?

1. **Repo Düzeyinde**:
   * “Settings” → “Secrets and variables” → “Actions” → “New repository secret” butonu.
   * “Name” ve “Value” girip kaydedersin (örn. `MY_API_KEY`).
2. **Organization Düzeyinde**:
   * “Organization Settings” → “Security” → “Actions” → “New organization secret”.
   * Yetkili repo(lar)ı seçebilirsin.
3. **Environment Düzeyinde**:
   * Repo “Settings” → “Environments” → Environment’ı seç → “Add secret”.



#### Workflow İçinde Kullanım:

```yaml
name: Use Secrets
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Display secret
        env:
          MY_SECRET: ${{ secrets.MY_SECRET }}  # “MY_SECRET” adındaki secret’ı atıyoruz
        run: |
          echo "Secret is: $MY_SECRET"
```

* `env:` satırında `secrets.MY_SECRET` atanır, step içinde `$MY_SECRET` olarak erişilir.
* Log’da gerçek değer maskeli görünür (`***`), bu yüzden gizliliği korunur.

### 4) Hassas Bilgileri Log’da Gösterirken Dikkat

* Eğer `echo $MY_SECRET` dersen, GitHub Actions loglarda `***` olarak maskeler.
* Fakat secret değerini başka bir string’e dönüştürür veya harf harf yazarsan, maskeleme davranışı bozulabilir. Bu gibi hileli durumlardan kaçınmak gerekir.

### 7) Common Use Cases

* **API Keys / Tokens**: REST API veya SaaS hizmeti için ihtiyacın olan anahtarlar.
* **Database Credentials**: `DB_USER`, `DB_PASS` gibi bilgiler.
* **SSH Keys**: Bazı dağıtım (deployment) senaryolarında SSH key saklama.

### 8) Özet

* **Secrets** → GitHub Actions’da **gizli bilgi** saklama ve kullanma mekanizması.
* **`secrets` context** → `secrets.MY_SECRET` şeklinde değer atayıp step’lerde kullanırsın.
* **Organizasyon, Repo, Environment** katmanlarında tanımlayabilir, erişim kısıtlaması yapabilirsin.
* **Maskelenir** → Log’larda plain text görünmez, “\*\*\*” olarak gizlenir.

Böylece şifrelere, tokenlara, hassas bilgilere hem CI/CD sürecinde hem de log’larda güvenli şekilde erişme olanağı elde edersin.

