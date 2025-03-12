---
icon: ubuntu
---

# Install Loki For Ubuntu

### 1. Gereksinimler

* **Ubuntu** (veya benzer bir Linux dağıtımı) üzerinde internet erişimi.

### 2. Loki’nin İndirilmesi

1. **İlgili Sürüme Göz Atın**
   * İlk olarak [Loki’nin GitHub’daki sürüm (release) sayfasına](https://github.com/grafana/loki/releases) gidin.
   * Kullanmak istediğiniz sürümü bulun ve `Assets` başlığı altında sistemiize uygun arşiv dosyalarını (örn. `loki-linux-amd64.zip`) indirin.

Örnek komut (son sürüm için; lütfen sürümü ihtiyacınıza göre değiştirin):

```bash
wget https://github.com/grafana/loki/releases/download/<sürüm>/loki-linux-amd64.zip
```

### 3. Arşiv Dosyasının Çıkarılması

İndirdiğiniz `loki-linux-amd64.zip` dosyasını bir klasöre çıkarın ve o klasöre geçiş yapın:

```bash
unzip loki-linux-amd64.zip
cd <klasör_adı>
```

> **Not**: Eğer Promtail de indirdiyseniz, onun arşivini de aynı klasöre çıkarmak faydalı olacaktır.

### 4. Konfigürasyon Dosyasının İndirilmesi

Loki için hazır bir yerel konfigürasyon (local config) dosyası sunulmaktadır. Bu dosyayı, kullandığınız Loki sürümüyle **aynı Git referansını** kullanarak indirmeniz gerekir.

Örneğin, `v2.8.0` sürümünü kullanıyorsanız:

```bash
wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/cmd/loki/loki-local-config.yaml
```

> **Önemli**: Sürüm numarasını (ör. `v2.8.0`) kendi indirdiğiniz Loki sürümüne uygun şekilde değiştirin.

### 5. Loki’yi Başlatma

Arşivden çıkan `loki-linux-amd64` dosyasını kullanarak Loki’yi konfigürasyon dosyası eşliğinde başlatabilirsiniz:

```bash
sudo ./loki-linux-amd64 --config.file=loki-local-config.yaml
```

* Burada `loki-local-config.yaml` indirdiğiniz konfigürasyon dosyasının adıdır.
* **sudo** kullanmak isteğe bağlıdır, ancak sisteme tam erişim gerektiğinde yararlı olabilir.

Loki bu komutla birlikte arka planda çalışmaya başlayacak ve `loki-local-config.yaml` dosyasında tanımlanmış ayarları (ör. port, depolama dizini, vs.) kullanacaktır.

### 6. Doğrulama ve Sonraki Adımlar

1. **Çalışıp Çalışmadığını Kontrol Edin**
   * Terminalde, Loki’nin log’larını (uygulama mesajlarını) gözlemleyerek hatasız bir şekilde başladığından emin olun.
   *   Varsayılan olarak 3100 portunu kullanır. Aşağıdaki komutla Loki’nin dinleme yapıp yapmadığını kontrol edebilirsiniz:

       ```bash
       netstat -tunlp | grep 3100
       ```
2. **Loki API Testi**

<figure><img src="../.gitbook/assets/Screenshot 2025-03-12 at 12.04.14.png" alt=""><figcaption></figcaption></figure>

* Tarayıcınızda `http://<sunucu_IP_adresi>:3100/metrics` veya `http://localhost:3100/metrics`adresine giderek Loki’nin temel metriklerini görebilirsiniz.

{% embed url="https://grafana.com/docs/loki/latest/setup/install/local/#install-manually" %}
