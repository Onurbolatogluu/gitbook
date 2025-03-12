---
icon: mailbox-flag-up
---

# Install Promtail For Ubuntu

Aşağıdaki adımları izleyerek **Promtail**’i Ubuntu (veya benzeri bir Linux) ortamına kurup temel bir konfigürasyonla **Loki**’ye log gönderebilirsiniz.

### 1. Gerekli Ön Bilgiler

* **Promtail**, Loki’ye log göndermek için kullanılan bir ajan (agent) servisidir.
* Çalışma mantığı: Sunucudaki log dosyalarını belirttiğiniz yollar (scrape\_configs) üzerinden okur, topladığı logları **Loki**’ye (belirtilen URL) push eder.

### 2. Promtail Dosyalarının İndirilmesi

1. **Sürüm Seçimi**
   * [Grafana Loki’nin GitHub’daki Releases sayfasına](https://github.com/grafana/loki/releases) gidin ve sisteminize uygun Promtail arşiv dosyasını bulun (ör. `promtail-linux-amd64.zip`).
2. **İndirme Komutu**
   *   Aşağıda, `v3.4.2` sürümünü indiriyoruz. Versiyon numarasını kendi ihtiyacınıza göre değiştirebilirsiniz:

       ```bash
       wget https://github.com/grafana/loki/releases/download/v3.4.2/promtail-linux-amd64.zip
       ```

### 3. Dosyaların Çıkarılması

Arşiv dosyasını (ZIP) çıkarın:

```bash
unzip promtail-linux-amd64.zip
```

> Not: Arşivden tipik olarak `promtail-linux-amd64` isimli bir (binary) dosya çıkar.

### 4. Konfigürasyon Dosyasının İndirilmesi

Grafana Loki reposunda, Promtail için hazır örnek konfigürasyon dosyaları bulunur. Aşağıdaki örnekte “main” branch’ten indiriliyor, ancak production ortamında **indirdiğiniz sürüm** ile **uyumlu** bir Git referansı kullanmanız önerilir.

```bash
wget https://raw.githubusercontent.com/grafana/loki/main/clients/cmd/promtail/promtail-local-config.yaml
```

Bu dosyayı dilediğiniz metin editörüyle açıp (örneğin `vi`, `nano` vb.) düzenleyebilirsiniz:

```bash
vi promtail-local-config.yaml
```

### 5. Örnek Konfigürasyon İçeriği

Aşağıda az önce indirdiğimiz tipik bir `promtail-local-config.yaml` dosyasının içeriği bulunmaktadır:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://node01:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log
      stream: stdout
```

#### Önemli Parametreler

* **server.http\_listen\_port**: Promtail’in kendi arayüzü ve metriklerini dinlediği port.
* **clients.url**: Logların gönderileceği Loki endpoint’i (örn. `http://node01:3100/loki/api/v1/push`).
* **scrape\_configs**: Hangi yolların (log dosyalarının) nasıl etiketleneceğini belirler.
  * `__path__`: Bu örnekte `/var/log/*log` içindeki log dosyaları toplanıyor.
  * `labels`: Loglara ek metadata (örn. `job: varlogs`) eklemek için kullanılır.

### 6. Promtail’i Çalıştırma

Konfigürasyon dosyası hazır olduktan sonra Promtail’i şu şekilde başlatabilirsiniz:

```bash
sudo ./promtail-linux-amd64 --config.file=promtail-local-config.yaml
```

* `sudo` kullanımı isteğe bağlıdır, ancak sistem loglarına erişimi (örn. `/var/log` klasöründeki dosyalar) kolaylaştırmak için yönetici hakları gerekebilir.
* Promtail, komut satırında başladığında konfigürasyonda belirtilen log dosyalarını izlemeye ve Loki’ye göndermeye başlar.

### 7. Doğrulama

1. **Promtail’in Çalışıp Çalışmadığını Kontrol**
   * Terminalde Promtail loglarını takip ederek hata veya uyarı mesajı olup olmadığına bakın.
   * Ayrıca `ps aux | grep promtail` gibi bir komutla Promtail’in işlem listesinde olduğunu doğrulayabilirsiniz.
2. **Loki Tarafından Log Alınıp Alınmadığı**
   * Loki arka planda çalışıyorsa, `[Loki Sunucusu Adresi]:3100/metrics` gibi bir endpoint’ten metrikleri gözlemleyebilir veya Grafana ile “Explore” menüsünden LogQL sorguları yaparak gelen logları görüntüleyebilirsiniz.
3. **Port Kontrolleri**
   * `netstat -tunlp | grep 9080` komutu ile Promtail’in 9080 portunu dinlediğini teyit edebilirsiniz.

**Promtail** ve **Loki** kurulumları tamamlandıktan sonra, Promtail konfigürasyonundaki `scrape_configs` ve `clients.url`kısımlarını ihtiyaçlarınıza göre düzenleyerek ek log dizinleri ekleyebilir veya log etiketlerinizi (labels) zenginleştirebilirsiniz.

Bu adımlar sayesinde, Promtail’inizi başarıyla kurulup Loki’ye log iletecek şekilde ayarlayabilir, sistemdeki log’larınızı daha görünür ve yönetilebilir hale getirebilirsiniz.
