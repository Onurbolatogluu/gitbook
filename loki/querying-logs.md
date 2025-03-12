---
icon: comments-question-check
---

# Querying Logs

### 1. Grafana Kurulumu

1.  **Grafana GPG Anahtarını İndirme ve Kaydetme**

    ```bash
    sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
    ```
2.  **Grafana Deposu Ekleme**

    ```bash
    echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    ```
3.  **Paket Listesi Güncelleme**

    ```bash
    sudo apt-get update
    ```
4.  **Grafana’nın Kurulumu**

    ```bash
    sudo apt-get install grafana
    ```
5.  **Grafana Servisini Başlatma**

    ```bash
    systemctl start grafana-server
    ```
6.  (İsteğe bağlı) **Servisin Otomatik Başlaması**

    ```bash
    systemctl enable grafana-server
    ```

Bu işlemler tamamlandığında, Grafana varsayılan olarak **3000** numaralı portta (ör. `http://<sunucu_ip_adresi>:3000`) hizmet vermeye başlayacaktır.

### 2. Grafana’ya Erişim

1. Tarayıcınızda `http://<sunucu_ip_adresi>:3000` adresine gidin.
2. Varsayılan kullanıcı adı/parola genellikle `admin/admin` şeklindedir. İlk girişte parola değiştirmeniz istenebilir.

### 3. Loki Datasource’u Ekleme

<figure><img src="../.gitbook/assets/Screenshot 2025-03-13 at 00.57.50.png" alt=""><figcaption></figcaption></figure>

> **Önkoşul**: Loki’nin ve Promtail’in çalışır durumda olduğundan emin olun. Bizim örneğimizde Loki, `node01:3100` adresinden erişilebilecek şekilde ayarlanmış durumda.

1. **Grafana Ana Ekranı**: Sol menüdeki “Connections” (veya bazı sürümlerde “Configuration”) → “Data sources” bölümüne gidin.
2. **Yeni Bir Data Source Ekle**: “Add new data source” butonuna tıklayın.
3. **Loki’yi Seçin** (Ekran görüntünüzde olduğu gibi “Loki” seçeneği belirecektir).
4. **Bağlantı Ayarları (Connection)**:
   * `Name` alanına örnek olarak `loki` yazabilirsiniz.
   * `URL` kısmına, Loki’nin çalıştığı adresi girin, örn. `http://node01:3100`.
   * Authentication metodunu “No Authentication” veya ihtiyacınıza göre farklı bir yöntem olarak ayarlayın.
5. **Kaydet ve Test Et**: En altta “Save & test” butonu aracılığıyla bağlantıyı doğrulayın. “Data source is working” benzeri bir mesaj almanız gerekir.

### 4. Logları Görüntüleme (Explore Ekranı)

<figure><img src="../.gitbook/assets/Screenshot 2025-03-13 at 00.58.36.png" alt=""><figcaption></figcaption></figure>

Artık Grafana’da bir Loki veri kaynağı (data source) tanımlandı. Logları görmek için:

1. **Sol Menüde** “Explore” sekmesine gidin.
2. Üstte yer alan data source seçiciden `loki`’yi seçin (veya ismini ne koyduysanız).
3. **Label Filtreleri**: Hangi label'a ait logları görüntüleyeceksek bunu belirtiyoruz, `job="varlogs"` gibi etiketlere göre filtre uygulayabilirsiniz.
   *   Örnek sorgu:

       ```logql
       {job="varlogs"}
       ```
   * Arama kutusu altında ek filtreler (ör. `|= "error"`) de ekleyebilirsiniz.
4. **Zaman Aralığı**: Sağ üst taraftaki zaman seçiciyle (`Last 1 hour`, `Last 5 minutes` gibi) dilediğiniz zaman dilimini seçerek log verilerini daraltabilirsiniz.
5. **Sonuçların İncelenmesi**: Ekranın altında gelen satırlar, Loki’den sorgu sonucu dönen log kayıtlarıdır.
   * Birçok log satırının listelendiğini ve zaman/grafik penceresinde “Log volume” grafiğini görebilirsiniz.

### 5. Sık Karşılaşılan Sorunlar ve İpuçları

* **Bağlantı Başarısız Hatası**:
  * Loki URL’nizin doğru olduğundan, firewall veya güvenlik grubunda (örneğin AWS’de) 3100 portunun açık olduğundan emin olun.
* **Log Yok Görünüyor**:
  * Promtail’in çalıştığını ve `scrape_configs` altında tanımlı yolların doğru olduğundan emin olun (`__path__`: `/var/log/*log` vb.).
  * Zaman aralığını genişleterek veya farklı label değerleri girerek arama yapın.
* **Etiket (Label) Yok**:
  * Promtail konfigürasyonunda `labels` altında `job`, `env`, `stream` gibi değerleri eklememiş olabilirsiniz. Daha zengin etiket tanımlamak için `promtail-local-config.yaml` dosyasını düzenleyin.

### 6. Özet

1. **Grafana’yı Yükle ve Başlat**: GPG anahtarı ekle, depo ekle, `apt-get install grafana` ile kur, `systemctl start grafana-server` komutuyla başlat.
2. **Tarayıcıdan Giriş**: `http://<sunucu_ip_adresi>:3000` adresine giderek “admin/admin” varsayılan bilgileriyle giriş yap.
3. **Loki Datasource Kurulumu**: “Connections” (veya “Configuration”) → “Data sources” → “Add new data source” → `Loki` → URL olarak `http://node01:3100` (örnek).
4. **Test**: “Save & test” butonuyla bağlantıyı doğrula.
5. **Log Analizi (Explore)**: Data source olarak `loki`’yi seç, etiket bazlı sorgular (ör. `{job="varlogs"}`) yaparak log kayıtlarını görüntüle.

