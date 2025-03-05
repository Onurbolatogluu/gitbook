---
icon: signal-bars
---

# Workflow Status Badge

**Workflow Status Badge**, bir GitHub Actions iş akışının (“workflow”) **geçerli durumunu** (passing, failing vb.) görsel bir sembolle (rozet) gösteren küçük bir resim dosyasıdır. Genellikle **README.md** veya başka belgelerde yer alır, böylece projeye bakan herkes **son CI/CD sonuçlarını** hızlıca görebilir.

### 1) Ne İşe Yarar?

* **“Passing” veya “Failing” gibi durumları** otomatik günceller.
* Projeye giren kişiler, iş akışının (testler, build’ler) sorunsuz mu çalıştığını görebilir.

### 2) Nasıl Eklenir?

1. **Badge URL’sini Bul**
   * GitHub Actions sekmesinde workflow seçtikten sonra, “Create status badge” veya “Status badge” gibi bir link verilir.
   *   URL genelde şu formatta olur:

       ```
       bashCopyhttps://github.com/<owner>/<repo>/actions/workflows/<workflow_file_name>/badge.svg
       ```
   * İsteğe bağlı olarak `?branch=<branch_name>` veya `?event=<event_type>` gibi parametreler ekleyebilirsin.

**Markdown’a Eklemek**

*   README.md dosyana şu şekilde bir satır ekle:

    ```md
    ![My Workflow](https://github.com/<owner>/<repo>/actions/workflows/<workflow_file_name>/badge.svg?branch=main)
    ```
* Dosyayı commit ettiğinde, GitHub README ekranında “Passing” veya “Failing” rozeti görünür.

### 3) Özel Durumlar

* **Özel (Private) Repo**: Bu rozet dışarıdan (anonim kullanıcılar) görülemez. Sadece repo’ya erişimi olan kişiler görür.
*   **Branch/Event Seçimi**: Parametrelerle hangi branch veya hangi tetikleme (push, PR vb.) baz alınacağını belirleyebilirsin. Örneğin:

    ```bash
    .../badge.svg?branch=feature-1
    .../badge.svg?event=push
    ```

#### Özetle,

GitHub, bir **status badge** (durum rozeti) için **özel bir URL** üretir (örneğin “`.../badge.svg`”). Bu URL, o workflow’un “geçti” veya “hata” gibi en son durumunu döndüren bir **küçük resim** dosyası gibidir.

**Nasıl gösteririz?**

* Projene girenler genelde **README.md** dosyasının içeriğini (Github ana sayfasında) görür.
* README.md içine bir satır ekleyip (Markdown formatında) bu **rozetin URL’sini** işaret edersen, GitHub bu resmi sayfada görüntüler.
* Böylece **proje sayfası**na bakan herkes “Bu workflow şu an PASSING mi, FAILING mi?” diye anında görebilir.
