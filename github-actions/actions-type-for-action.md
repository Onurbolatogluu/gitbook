---
icon: location-pin
---

# Actions type for Action

**Action Types**, GitHub Actions içindeki **özel actions** (ör. “actions/checkout”, “actions/setup-node” gibi) **yaratırken** seçebileceğin **yöntemler**dir.&#x20;

Başlıca 3 tür bulunur:

<figure><img src="../.gitbook/assets/Screenshot 2025-03-06 at 11.46.12.png" alt=""><figcaption></figcaption></figure>

1. **Docker container action**
   * Action kodun, bir **Docker imajı** içinde çalıştırılır.
   * Yalnızca **Linux** runner’larında kullanılır (çünkü Docker container’lar Windows ve macOS runner’larında desteklenmez).
   * Örnek: `Dockerfile` içinde “entrypoint.sh” vb. ile bir komut seti paketleyebilirsin. Runner “docker run …” diyerek o container’ı çalıştırır.
2. **JavaScript action**
   * Action kodun, **Node.js tabanlı** JavaScript olarak doğrudan host makinede (runner) çalışır, Docker’a ihtiyaç yok.
   * Linux, macOS, Windows runner’larda da çalışır (Node 16 veya benzer sürüm kurulu kabul edilir).
   * Genelde **en yaygın** türdür; `index.js` + `action.yml` yazarak custom logic ekleyebilirsin.
3. **Composite action**
   * **Birden fazla** step (komut) ve belki ek script’leri bir araya getirerek **tek bir action** gibi tanımlarsın.
   * Docker veya JavaScript yazmadan, YAML step’lerini “composite” haline getirirsin.
   * Linux, macOS ve Windows’ta çalışabilir.

**Custom Actions**, GitHub Actions ekosisteminde kendi oluşturduğun (veya başkalarının oluşturup paylaştığı) **yeniden kullanılabilir komut setleridir**. Yani, Actions’ın “modüler bileşenleri” gibi düşünülebilir. Varsayılan olarak GitHub “actions/checkout”, “actions/setup-node” gibi resmî actions 'lar sunar. Fakat ihtiyacın doğrultusunda **kendin** de bir aksiyon geliştirebilirsin.



***
