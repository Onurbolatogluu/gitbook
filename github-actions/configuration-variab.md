---
icon: square-root-variable
---

# Configuration Variab

GitHub Actions’da **Configuration Variables (Konfigürasyon Değişkenleri)**, “**non-sensitive**” (hassas olmayan) değerleri saklamak ve bunları Workflow’larında kullanmak için geliştirilmiş bir özelliktir. Yani bu değişkenler, “Secrets” gibi gizli/şifreli saklanmıyor; fakat yine de tanımlanan değişkenlere pratik şekilde erişebilmeni sağlıyor.

### 1) Configuration Variables Nedir?

* **Amaç**: API URL, sürüm numarası, dosya yolları gibi **hassas olmayan** değerleri global veya environment bazında saklayıp, Workflow’larda tekrar tekrar kullanmak.
*   **Farkı ne?**

    * **Secrets**: Gizli/saklanması gereken şifre, token gibi veriler. `secrets.*` context’iyle erişilir ve log’larda maskelenir.
    * **Configuration Variables**: Düz metin, hassas olmayan ayarlar. `vars.*` context’iyle erişilir, maskelenme gibi bir ihtiyaç yok.



### 2) “vars” Context

Workflow içinde şöyle kullanabilirsin:

```yaml
steps:
  - name: Print variable
    run: echo "App ID: ${{ vars.APP_ID_EXAMPLE }}"
```

* `vars.APP_ID_EXAMPLE`, GitHub arayüzünden ya da CLI’dan ayarladığın bir değişkenin değerini döndürür.

**Önemli Not**: Bu değişkeni tanımlarken hassas değer koyarsan, log’larda olduğu gibi görünecektir (maskelenmez). Çünkü bu “non-sensitive” kullanım içindir.



### 3) Nasıl Tanımlanır?

GitHub, CLI (komut satırı) ya da web arayüzünden değişken ekleme/düzenleme işini destekliyor:

1. **GitHub CLI (“gh variable set …”)**
   * Örnek komutlar:

```yaml
# Mevcut repo için bir değişken ekle
gh variable set MYVARIABLE

# Değeri doğrudan ver
gh variable set MYVARIABLE --body "example_value"

# .env dosyasından çoklu değişken yükle
gh variable set -f .env

# Organization-level variable
gh variable set MYVARIABLE --org myOrg
```

**Web Arayüzünden**

* Repo “Settings” → “Secrets and variables” → “Actions” menüsünde “Variables” sekmesi bulunur.
* “New Repository Variable” diyerek isim ve değer girersin.
* Organization ya da Environment düzeyinde de benzer menüler vardır.

### 4) Neden Secrets Değil de Vars?

* **Sensitive (gizli) data** → `secrets` kullan
* **Non-sensitive** (herhangi bir URL, sürüm numarası, config) → `vars` kullan

Bu sayede parolalarla sıradan ayarları karıştırmadan, hem daha düzenli hem de güvenlik açısından doğru yaklaşımı benimsersin.

```yaml
name: Use Config Variable
on: [push]

jobs:
  demo:
    runs-on: ubuntu-latest
    steps:
      - name: Print Config
        run: |
          echo "My Config: ${{ vars.MY_CONFIG_VAR }}"
```

* Burada “`MY_CONFIG_VAR`” adlı bir değişkeni (örneğin “https://api.example.com/v1”) sakladığını varsayalım.
* Workflow’da `$MY_CONFIG_VAR`’ın değerini doğrudan görebilirsin (maskelenmez).

#### Özet

* **Configuration Variables** → Hassas olmayan değerler için “vars” context.
* **Tanımlama** → Web arayüzünden veya GitHub CLI ile (`gh variable set …`).
* **Düzeyler** → Repo / Environment / Organization.
* **Fark**: Secrets’lar şifreli ve maskelenir, Vars ise düz metin; dolayısıyla güvenli bilgileri asla Vars’ta tutmamalısın.

Bu şekilde, GitHub Actions içinde bir yandan gizli verileri “Secrets” ile yönetirken, diğer yandan “Vars” ile konfigürasyon ayarlarını kolayca paylaşabilirsin.
