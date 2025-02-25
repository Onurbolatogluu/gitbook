---
icon: handshake-simple-slash
---

# Github Token Secret

**GITHUB\_TOKEN**, GitHub Actions tarafından **otomatik** ve **geçici** olarak oluşturulan bir erişim anahtarıdır (token). Bir depo içinde GitHub Actions etkinleştirdiğinde, her job başlarken GitHub bu “token”ı job’a ekler. Bu token sayesinde iş akışın (Workflow), repo üzerinde (issue açma, pull request’e yorum yapma, vb.) ya da GitHub API çağrısı yaparken kimlik doğrulaması yapabilir. Böylece **manuel olarak** herhangi bir “Personal Access Token” (PAT) oluşturup saklamana gerek kalmadan, güvenli ve geçerli bir izinle GitHub eylemleri gerçekleştirebilirsin.



### 1) Nasıl Oluşur ve Ne İşe Yarar?

* **Her job başında GitHub** depoya bir GitHub App yükler ve **o job’a özel** bir “installation access token” tanımlar. Bu token “**GITHUB\_TOKEN**” adıyla environment’a eklenir (aynı zamanda `secrets.GITHUB_TOKEN` gibi erişebilirsin).
* Bu token, GitHub API’sine veya gh CLI’sine karşı **depoya ait** belirli yetkilerle (permissions) işlemler yapmanı sağlar. Örneğin:
  * Issue oluşturmak / güncellemek
  * Kodu klonlamak veya bir pull request eylemi yapmak (örn. label eklemek)
  * Pull request’e yorum yazmak

**Önemli**: Varsayılan yetkiler **okuma** (read) + **yakın eylemler** olabilir, ancak “permissions” ayarıyla `write` veya diğer gerekli alanları açıp kapayabilirsin.

### 2) Workflow’da Kullanımı

En yaygın kullanım:

1. **Bir step’te `env` olarak** GITHUB\_TOKEN’ı atamak veya doğrudan `secrets.GITHUB_TOKEN` ifadesini kullanmak.
2. `gh` CLI veya `curl` ile GitHub API çağrısı yaparak issue, PR vb. eylemler yapmak.

Örnek 1: GitHub CLI ile Issue Açmak

```yaml
name: Open new issue
on: workflow_dispatch

jobs:
  open-issue:
    runs-on: ubuntu-latest
    permissions:
      issues: write  # Bu job, issue yazma hakkına sahip olmalı
    steps:
      - name: Create an issue with gh CLI
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # gh CLI 'GH_TOKEN' veya 'GITHUB_TOKEN' env arar
        run: |
          gh issue create \
            --repo ${{ github.repository }} \
            --title "Issue title" \
            --body "Issue body"

```

* `permissions.issues: write` → Bu job, depo üzerinde “issues” konusunda yazma izni alır.
* `env: GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}` → `gh` CLI “GH\_TOKEN” veya “GITHUB\_TOKEN” adlı env’i okuyarak yetkilendirme yapar.

Örnek 2: `curl` ile GitHub REST API

```yaml
jobs:
  create_issue:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - name: Create issue using REST API
        run: |
          curl -X POST \
            -H "authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" \
            -H "content-type: application/json" \
            -d '{"title":"New Issue from workflow"}' \
            https://api.github.com/repos/${{ github.repository }}/issues
```

* `authorization: Bearer ${{ secrets.GITHUB_TOKEN }}` → GitHub REST API çağrılarında HTTP header olarak token’ı ekliyoruz.
* Bu sayede “issue” oluşturma izni varsa işleme koyar.

### 3) Permissions (İzinler)&#x20;

* **permissions** alanıyla, `contents: read | write`, `issues: write`, `pull-requests: write` vb. kontrolleri yaparsın.
* Varsayılan olarak GitHub Actions, GITHUB\_TOKEN’a “read” izni verir. Daha fazla (write) lazım olduğunda `permissions` eklemek gerekir.
* Bu ayarlar,  istenmeyen bir Action’ın, deposuna zarar vermesini engellemeye yönelik güvenlik amaçlı bir önlem.

### 4) Süresi ve Geçerliliği&#x20;

* **Token** job boyunca geçerlidir. Job bittiğinde token da devre dışı kalır.
* Her job yeni bir token alır, bu yüzden bir job’tan diğerine GITHUB\_TOKEN taşınmaz veya aynı kalmaz.

### 5) Farkı Nedir? (Personal Access Token vs. GITHUB\_TOKEN)

* **GITHUB\_TOKEN** → Otomatik, geçici, repo-lokal. Workflow context’inde kolay kullanılır.
* **Personal Access Token (PAT)** → Kullanıcının el ile oluşturduğu, GitHub kullanıcı hesabına bağlı, genelde daha geniş/küresel izinlere sahip olabilir. PAT ile org veya repo’lar arası iş yaparken dikkatli olmak ve token’i saklamak gerekir.
* GITHUB\_TOKEN, genellikle PR/Issue otomasyonu gibi **repo içi** eylemler için en güvenli yaklaşım kabul edilir.

#### Özet

1. **Otomatik oluşturulur**: Workflow job’u başlayınca GitHub bir token tanımlar.
2. **secrets.GITHUB\_TOKEN** şeklinde erişilir.
3. **İzinleri** `permissions:` bloğuyla ayarlanır (varsayılan read-only, write yapmak için belirtmen gerekir).
4. **API/CLI işlemlerinde** repo düzeyinde kimlik doğrulaması sağlar: Issue açmak, PR yorumlamak, label düzenlemek vb.
5. **Geçici**: Job bitince token geçersiz olur.

Bu şekilde **GITHUB\_TOKEN** kullanarak GitHub’da otomatik işlemler yapabilir, manuel PAT yönetimiyle uğraşmadan güvenli ve pratik bir şekilde CI/CD senaryolarını oluşturabilirsin.



{% embed url="https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication" %}
