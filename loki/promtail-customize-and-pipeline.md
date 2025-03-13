---
icon: input-pipe
---

# Promtail Customize & Pipeline

<figure><img src="../.gitbook/assets/Screenshot 2025-03-14 at 01.25.23.png" alt=""><figcaption></figcaption></figure>

Kubernetes üzerinde çalışan Promtail’in varsayılan konfigürasyon dosyası (`promtail.yaml`), base64 formatında saklandığı için önce decode edilerek ham içeriğine ulaşılabilir. Ardından, logları daha zengin etiketleyebilmek (örneğin `method` ve `code` gibi değerleri yakalamak) amacıyla ek `pipeline_stages` tanımlanarak, JSON içindeki belirli alanlar parse edilip Loki’ye label olarak gönderilebilir. Sonraki adımda, güncellenmiş `promtail.yaml` içeren bir Secret yeniden oluşturulup Promtail Pod’u yeniden başlatılarak, yeni konfigürasyonun devreye girmesi sağlanır.

<figure><img src="../.gitbook/assets/Screenshot 2025-03-14 at 01.27.58.png" alt=""><figcaption></figcaption></figure>

1.  **Mevcut `promtail.yaml` Dosyasını Alma (Decode)**\
    Promtail yapılandırması varsayılan olarak, bir Secret içinde (örneğin `loki-promtail`) saklanır.

    ```bash
    kubectl get secret loki-promtail -o jsonpath="{.data.promtail\.yaml}" | base64 --decode > promtail.yaml
    ```

    Bu komut, base64 kodlu içeriği “promtail.yaml” dosyasına yazar. Böylece varsayılan veya mevcut Promtail yapılandırması düz metin (YAML) olarak düzenlenebilir hale gelir.
2. **Düzenleme: Loglardan Ek Verileri Label Olarak Çıkarmak**\
   `promtail.yaml` içindeki `scrape_configs` altında, **`pipeline_stages`** bölümüne `match` ve `json` gibi ek aşamalar eklenir.
   * **`match`** ile belirli bir label’e (örneğin `app="api"`) sahip logların ekstra işlem görmesi sağlanır.
   * Ardından iç içe **`json`** parse aşamaları kullanılarak log mesajından `code`, `method` gibi alanlar ayrıştırılır.
   * Son olarak **`labels:`** altında `code` ve `method` tanımlanır. Bu sayede Loki, bu alanları **label** olarak saklar ve Grafana’da etiket bazlı filtreleme yapmak mümkün hale gelir.
3.  **Secret’ı Güncelleme**\
    Yeni “promtail.yaml” kaydedildikten sonra mevcut Secret silinir:

    ```bash
    kubectl delete secret loki-promtail
    ```

    Ardından aşağıdaki komutla yeniden oluşturulur:

    ```bash
    kubectl create secret generic loki-promtail --from-file=./promtail.yaml
    ```

    Böylece Promtail Pod’una yeni YAML konfigürasyonu enjekte edilir.\


    <figure><img src="../.gitbook/assets/Screenshot 2025-03-14 at 01.45.36.png" alt=""><figcaption></figcaption></figure>
4.  **Tüm Pod’ları Sırasıyla Yeniden Başlatma (Tek bir pod varsa direkt olarak delete yapabilirsiniz)**

    ```bash
    kubectl rollout restart ds/loki-promtail
    ```

    `loki-promtail` yerine kendi DaemonSet adınızı yazın (örn. `loki-promtail`, `promtail-daemonset` vs.).

    Bu komut, DaemonSet’in bağlı olduğu tüm Pod’ları sırasıyla silip tekrar oluşturur. Yeni Pod’lar ayağa kalkarken **güncellenmiş Secret**’tan `promtail.yaml` dosyasını okur.

    <figure><img src="../.gitbook/assets/Screenshot 2025-03-14 at 01.43.32.png" alt=""><figcaption></figcaption></figure>
5. **Sonuç: Yeni Etiketler (Labels)**\
   Grafana’daki **Explore** ekranında sorgu yaptığınızda, log satırlarının label kısmında `code` ve `method` gibi ek alanlar görünür. Bu sayede `code=200` gibi etiket bazlı filtrelerle loglara **daha hızlı ve verimli** ulaşılabilir.

**Kısacası**, bu yöntemle Promtail, log içindeki alanları **JSON** olarak parse edip **Loki’ye label** biçiminde aktarır. Böylece logları yalnızca metin bazlı aramak yerine, eklenen etiketlere göre de sorgulama yapılabilir (örneğin `method="GET"`, `code="200"`).

