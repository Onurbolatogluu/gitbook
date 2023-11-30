# 🗒 prometheus.yml dosyasına ilk bakış:

Aşağıdaki yaml dosyası Prometheus'un standart konfigürasyon dosyasının bir örneğidir. Bu dosya, Prometheus'un scrape işlemleri, alerting (alarm) ayarları ve kuralların nasıl çalışacağı hakkında ayarları içerir.

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```

* `global`: Bu bölüm global konfigürasyon ayarlarını içerir. `scrape_interval` ayarı, scrape edilecek hedeflere yapılacak sorguların sıklığını, `evaluation_interval` ise alert kurallarının kaç saniyede bir değerlendirileceğini belirler. `scrape_timeout` ise scrape işleminin sonlandırılacağı süreyi belirler.
* `alerting`: Bu bölümde alertmanager konfigürasyonu yer alır. Alertmanager, Prometheus'tan gelen uyarıları alır ve belirtilen kurallara göre kullanıcıları uyarır. `alertmanagers` bölümünde alertmanager'ın hangi adreslerde çalıştığını belirtiriz.
* `rule_files`: Bu bölümde alert ve recording kurallarının yükleneceği dosyaların listesi yer alır. Eğer bu bölüm boşsa kurallar yüklenmez.
* `scrape_configs`: Bu bölüm scrape edilecek hedeflerin listesini içerir. `job_name` ayarı hedeflerin hangi iş tanımıyla scrape edileceğini belirler. `static_configs` bölümünde scrape edilecek hedeflerin adreslerini belirtiriz.&#x20;
  * Yukarıdaki örnek dosya, Prometheus'un kendi metriklerini scrape etmek için ayarlanmıştır. Dosyanın son bölümünde, `scrape_configs` bölümünde `prometheus` adında bir iş tanımı ve `localhost:9090` adresinde çalışan bir hedef tanımlanmıştır. Bu ayarlar sayesinde Prometheus, kendi metriklerini her 15 saniyede bir sorgular ve sonuçları depolar.
