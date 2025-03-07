---
icon: code-compare
---

# Action Versions

**Action Versions**, GitHub Actions’da yazdığın veya kullandığın **custom action’ların** farklı sürümlerini yönetme yöntemidir. Böylece kullanıcılar, “`v1`”, “`v2-beta`”, “`v1.0.2`” gibi etiketlerle hangi sürümü aldıklarını netleştirebilir.&#x20;

“**Action Versions**” basitçe şöyle çalışır:

1. Sen bir **custom GitHub Action** (örneğin “my-action”) oluşturuyorsun.
2. Bu action’ı projelerinde kullanacak kişiler (veya sen) “`uses: my-action@...`” satırıyla sürüm belirtiyor: Mesela “`v1`”, “`v1.0.2`”, “`v2-beta`”, vb.

#### Neden Sürüm (Version) Lazım?

* **Büyük Değişiklik** (Breaking Change) Yaptığında → “v2” diye yeni major sürüm çıkarırsın. Eski projeler hala “v1” kullanabilir.
* **Küçük Güncelleme** (Mesela bir düzeltme) → “v1.0.2” benzeri patch sürüm çıkarırsın.
* **Beta / Deneme Sürümü** → “v2-beta” gibi bir etiket ekleyip, “deneme aşamasında” olduğunu belirtirsin.

#### Nasıl Etiketlersin?

* Repo’nda `git tag v1.0.2` diyerek bir sürüm oluşturursun.
* Veya “v1” adlı bir “major” tag, her zaman en güncel “1.x.x” commit’ine taşınır.
  * Böylece “v1” kullananlar, kırıcı değişiklikler gelmeden “güncel” kalır.
* Yeni major sürüm (mesela “v2”) çıkardığında, eski projeler “v1” ile çalışmaya devam edebilir.

#### Örnek Kullanım

```yaml
jobs:
  example:
    steps:
      - uses: my-org/my-action@v1
        # "v1" => major sürüm 1. İleride "v1.1.0" veya "v1.2.2" çıkarabilirsin, 
        # ve “v1” her zaman o en son 1.x.x sürümüne işaret edebilir.

      - uses: my-org/my-action@v1.0.2
        # Bu tam sürüm (1.0.2) => her seferinde aynı koda işaret, 
        # ufak yamalar veya minor değişiklikler gelmeyecek.
```

#### Basitçe

* **Action Versions**, “action dosyanın sürümü nedir?” sorusunu yanıtlar.
* Kullanıcılar `@v1` veya `@v1.2.3` şeklinde belirleyerek, güncellemeleri nasıl alacaklarını denetler.
* Sen de proje zamanla büyüdükçe “v2” (kırıcı değişiklik), “v1.0.1” (ufak düzeltme) gibi sürümler çıkarırsın.

