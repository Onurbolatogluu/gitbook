---
icon: globe-wifi
---

# Webhook Events

GitHub Actions’ı “dış dünyadan” bir HTTP isteği ile tetiklemek istiyorsan, `repository_dispatch` event’ini kullanırsın. Yani GitHub harici bir sistemden, senin GitHub repo’na özel bir endpoint’e istek atarak, bir iş akışını başlatabilir.

#### Örnek Workflow:

```yaml
name: Workflow on Repository Dispatch
on:
  repository_dispatch:
    types: [webhook]

jobs:
  respond-to-dispatch:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Run a script
        run: echo "Event of type ${GITHUB_EVENT_NAME}"
```

* `on.repository_dispatch.types: [webhook]` diyerek, `event_type` değeri `webhook` olan dış istekleri yakalıyoruz.
* `GITHUB_EVENT_NAME` gibi değişkenlerle, gelen isteğin türünü veya payload verisini Workflow içinde kullanabiliriz.

#### **HTTP isteği (curl) örneği**:

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token PERSONAL_ACCESS_TOKEN" \
  -d '{"event_type": "webhook", "client_payload": {"key": "value"}}' \
  https://api.github.com/repos/OWNER/REPO/dispatches
```

* `https://api.github.com/repos/{owner}/{repo}/dispatches` GitHub’ın “repository dispatch” endpoint’i.
* `-H "Authorization: token ..."` kısmına, **repo’ya yazma izni** olan bir **Personal Access Token** eklememiz gerekiyor.
* `-d '{"event_type": "webhook", ...}'` ile, `event_type` değerini `webhook` (veya istediğin başka bir isim) verip, **ek payload** gönderebiliyoruz.
* Bu istek başarıyla yapılırsa, GitHub Actions senin tanımladığın `repository_dispatch` Workflow’unu tetikler.



`-d` parametresinde yer alan `"client_payload": {"key": "value"}` kısmı, GitHub Actions’a **dış dünyadan** ek veri (parametre) yollamak içindir. Yani bu sayede `repository_dispatch` event’ine istediğin kadar “anahtar-değer” çifti ekleyebilirsin ve GitHub Actions içinde bu verilere ulaşabilirsin.&#x20;

#### Workflow Dosyası Örneği,

```yaml
name: Handle External Dispatch (Webhook Example)

on:
  repository_dispatch:
    types: [webhook]  # Dışarıdan tetiklemede 'event_type' olarak "webhook" beklendiğini ifade eder

jobs:
  handle_webhook:
    runs-on: ubuntu-latest
    steps:
      - name: Show Payload
        run: |
          echo "Verilen versiyon: ${{ github.event.client_payload.version }}"
          echo "Çalışma ortamı: ${{ github.event.client_payload.environment }}"
          echo "Herhangi başka bir değer: ${{ github.event.client_payload.anyOtherKey }}"

```

* `on.repository_dispatch.types: [webhook]`:\
  `event_type` değeri “webhook” olan bir dış çağrı geldiğinde bu Workflow tetiklenecek.
* `github.event.client_payload.*`:\
  `repository_dispatch` ile gelen **ek parametreleri** okuyabileceğimiz yerdir. Örneğin yukarıda `version`, `environment`, `anyOtherKey` gibi alanlara değer girersek, Workflow bunları terminale yazacak.

Aşağıdaki komutu kendi terminalinde çalıştırarak (öncesinde `PERSONAL_ACCESS_TOKEN`, `OWNER`, `REPO` değerlerini kendine göre düzenleyerek) bu Workflow’u tetikleyebiliriz:

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token PERSONAL_ACCESS_TOKEN" \
  -d '{
    "event_type": "webhook",
    "client_payload": {
      "version": "1.2.3",
      "environment": "staging",
      "anyOtherKey": "örnek bir değer"
    }
  }' \
  https://api.github.com/repos/OWNER/REPO/dispatches
```

* `"event_type": "webhook"`:\
  Buradaki değer, Workflow’da `types: [webhook]` satırıyla eşleştiği için tetikleme gerçekleşir.
* `"client_payload": {...}`:\
  Bu alana eklediğin anahtar-değer çiftleri, GitHub Actions içinde `github.event.client_payload` üzerinden okunabilir. Örnekte `"version": "1.2.3"`, `"environment": "staging"`, `"anyOtherKey": "örnek bir değer"` gönderiyoruz.
* **Token Gereksinimi**:\
  `PERSONAL_ACCESS_TOKEN`’ın repo’ya yazma (write) erişimi olması gerekir. Aksi takdirde `repository_dispatch` event’ini tetikleyemeyiz.

Özetle, `"client_payload"` altındaki `{"key": "value"}` kısımları tamamen workflow’muza göndermek istediğimiz **ek veri**yi temsil eder. Bu verileri Actions içinde okuyup istediğin gibi kullanabiliriz.

