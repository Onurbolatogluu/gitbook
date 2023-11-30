# ◼ BlackBox exporter

Blackbox Exporter, Prometheus izleme sistemi için bir bileşen olarak kullanılan bir yazılımdır. Blackbox Exporter, ağ tabanlı hizmetlerin durumunu ve performansını izlemek için kullanılır. Genellikle ağ sağlığı kontrolü, HTTP(S) hizmetlerinin erişilebilirliği, TCP bağlantı durumu ve DNS çözümlemesi gibi ağ tabanlı metrikleri toplamak için kullanılır.

Blackbox Exporter, Prometheus tarafından kullanılan metrikleri toplamak ve bunları Prometheus sunucusuna sunmak için HTTP tabanlı bir API sağlar. Bu sayede Prometheus, ağ tabanlı hizmetlerin durumu ve performansıyla ilgili metrikleri izleyebilir ve gerekli uyarıları oluşturabilir.

Blackbox Exporter, aşağıdaki özelliklere sahiptir:

1. HTTP, HTTPS, TCP, ICMP ve DNS gibi protokolleri destekler. Bu protokoller üzerinden hedef hizmetlere istekler gönderir ve yanıtlarını analiz ederek metrikleri toplar.
2. Gönderilen isteklerin zaman aşımı, durum kodu, veri doğrulama ve diğer istatistikler gibi ayrıntılı bilgilerini toplayabilir.
3. Blackbox Exporter'ın topladığı metrikler arasında hedef hizmetin erişilebilirlik durumu, yanıt süresi, başarı oranı ve TCP bağlantı durumu gibi bilgiler bulunur.
4. YAML formatında yapılandırılabilir. Bu sayede hangi hedefleri izleyeceğinizi ve izleme için kullanılacak protokollerin nasıl yapılandırılacağını belirleyebilirsiniz.

### Blackbox Exporter Kurulumu (Ubuntu 20.04)

Bu dökümanda, Ubuntu 20.04 üzerinde Blackbox Exporter'ı kurmayı göstereceğiz.

1. İlk olarak, Blackbox Exporter'ın en son sürümünü indirin. Aşağıdaki komutu kullanarak GitHub'dan en son sürümü kontrol edebilir ve indirebilirsiniz:

```shell
wget https://github.com/prometheus/blackbox_exporter/releases/latest/download/blackbox_exporter-<version>.linux-amd64.tar.gz
```

Yukarıdaki komutta `<version>` yerine en son sürüm numarasını yerleştirmeniz gerekmektedir. Örneğin, "v0.19.0" gibi bir sürüm numarasını kullanabilirsiniz.

2. İndirdiğiniz arşivi çıkartın:

```shell
tar xvfz blackbox_exporter-<version>.linux-amd64.tar.gz
```

3. Çıkan dizine girin:

```shell
cd blackbox_exporter-<version>.linux-amd64
```

4. Blackbox Exporter'ı `/usr/local/bin` dizinine kopyalayın:

```shell
sudo cp blackbox_exporter /usr/local/bin/
```

5. Gerekli dizinleri oluşturun ve izinleri ayarlayın:

```shell
sudo mkdir /etc/blackbox_exporter
sudo useradd --system --shell /bin/false blackbox_exporter
sudo groupadd blackbox_exporter
sudo usermod -aG blackbox_exporter blackbox_exporter
sudo chown blackbox_exporter:blackbox_exporter /etc/blackbox_exporter
```

6. Örnek bir yapılandırma dosyasını `/etc/blackbox_exporter/blackbox.yml` olarak oluşturun ve düzenleyin:

```shell
sudo nano /etc/blackbox_exporter/blackbox.yml
```

Dosyada aşağıdaki gibi bir yapılandırma yapısı kullanabilirsiniz:

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: [HTTP/1.1, HTTP/2]
      fail_if_ssl: false
      fail_if_not_ssl: false
  tcp_connect:
    prober: tcp
    timeout: 5s
```

Bu örnek yapılandırmada, HTTP ve TCP prober'ları ile ilgili örnek modüller bulunmaktadır. İhtiyaçlarınıza göre yapılandırmayı özelleştirebilirsiniz.

7. Blackbox Exporter'ı systemd servisi olarak yapılandırın:

```shell
sudo nano /etc/systemd/system/blackbox_exporter.service
```

Aşağıdaki örnekteki gibi systemd servis dosyasını düzenleyin:

```shell
[Unit]
Description=Blackbox Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox_exporter
Group=blackbox_exporter
Type=simple
ExecStart=/usr/local/bin/blackbox_exporter --config.file=/etc/blackbox_exporter/blackbox.yml --web.listen-address=:9115

[Install]
WantedBy=default.target
```

8. Servisi başlatın ve etkinleştirin:

```shell
sudo systemctl daemon-reload
sudo systemctl start blackbox_exporter
sudo systemctl enable blackbox_exporter
```

9. Blackbox Exporter'ın düzgün çalışıp çalışmadığını kontrol edin. Tarayıcınızı açın ve `http://localhost:9115/metrics` adresine gidin. Blackbox Exporter'ın metriklerini görebilmelisiniz.

<figure><img src="../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

Blackbox Exporter'ı Ubuntu 20.04 üzerinde bu adımları takip ederek kurabilirsiniz. Yapılandırmaları ihtiyaçlarınıza göre özelleştirerek ağ tabanlı hizmetlerinizin durumunu izleyebilirsiniz.

**Source:**

{% embed url="https://devconnected.com/how-to-install-and-configure-blackbox-exporter-for-prometheus/" %}

<mark style="color:blue;">-----------------------------------------------------------------------------------------------------</mark>

#### Optional:

<mark style="color:blue;">Blackbox Exporter'ı Prometheus'a eklemek için Prometheus yapılandırma dosyanızı düzenleyin (</mark><mark style="color:blue;">`prometheus.yml`</mark><mark style="color:blue;">):</mark>

```shell
sudo nano /etc/prometheus/prometheus.yml
```

<mark style="color:blue;">Aşağıdaki örnekteki gibi bir yapılandırma ekleyin:</mark>

```yaml
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Blackbox Exporter'da kullanmak istediğiniz modüller
    static_configs:
      - targets:
        - http://example.com  # İzlemek istediğiniz hedef URL'leri
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115  # Blackbox Exporter'ın adresi ve portu
```

<mark style="color:blue;">Yukarıdaki örnekte,</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`http_2xx`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">modülü ve</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`http://example.com`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">hedefi kullanılmaktadır. İhtiyaçlarınıza göre bu kısımları düzenleyebilirsiniz.</mark>

<mark style="color:blue;">Prometheus servisini yeniden başlatın:</mark>

```shell
sudo systemctl restart prometheus
```

<mark style="color:blue;">Artık Blackbox Exporter, Prometheus tarafından scrape edilecek ve izlediğiniz hedeflere ait metrikleri toplayacaktır.</mark>



<mark style="color:blue;">-----------------------------------------------------------------------------------------------------</mark>

#### HTTP Module;

```yaml
  http_find_prom:
    prober: http
    http:
      preferred_ip_protocol: "ip4"
      fail_if_body_not_matches_regexp:
      - "monitoring"
```

Yukarıdaki YAML yapılandırması, Blackbox Exporter'ın HTTP prober modülünü kullanarak belirli bir URL'ye yapılacak bir HTTP isteğini ifade eder. `http_find_prom` olarak adlandırılan bir modül tanımlanmıştır.

Bu özel yapılandırmada, aşağıdaki ayarlar kullanılmıştır:

* `prober: http`: Bu modül, HTTP protokolünü kullanarak istekleri yapmak için kullanılır.
* `preferred_ip_protocol: "ip4"`: Bu ayar, IPv4 protokolünün tercih edildiğini belirtir. Yani, bu istek IPv4 üzerinden gerçekleştirilecektir.
* `fail_if_body_not_matches_regexp`: Bu ayar, yanıtın içeriğinde belirtilen bir düzenli ifadenin eşleşmediği durumda başarısızlık olarak işaretlenmesini sağlar. Bu örnekte, "monitoring" ifadesini içermeyen bir yanıt, başarısızlık olarak kabul edilecektir.

Bu yapılandırma, Blackbox Exporter'ın belirtilen URL'ye bir HTTP isteği göndererek hedefin izlenebilirliğini kontrol etmesini sağlar. Yanıtın içeriğini kontrol ederek, belirli bir kelime veya düzenin olup olmadığını kontrol edebilir ve gerekirse uyarılar veya hata durumları oluşturabilirsiniz.

<figure><img src="../.gitbook/assets/image (129).png" alt=""><figcaption><p><code>fail_if_body_not_matches_regexp</code></p></figcaption></figure>

fail\_if\_body\_not\_matches\_regexp[^1] değeri 0 dönmüştür. Belirtilen parametre şunu ifade eder, prometheus.io sitesinde "monitoring" text'inin olduğuna işarettir.



```yaml
  http_find_prom:
    prober: http
    http:
      preferred_ip_protocol: "ip4"
      fail_if_body_not_matches_regexp:
      - "monitoringeeeeeeee"
```

<figure><img src="../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

prometheus.io sitesinde olmayan bir değeri search ettiğimizde, fail\_if\_body\_not\_matches\_regexp[^2] değeri 1 olarak dönecektir.



#### TCP Module;

<figure><img src="../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

tcp modülünü kullanarak, 10.90.0.144:8000 portunun çalışıp, çalışmadığını kontrol ediyoruz. probe\_success değeri 1 olarak döndüğünü görüyoruz, böylelikle 10.90.0.144:8000 portuna erişimimiz mevcut ve uygulamamız çalışıyor olarak düşünebiliriz.



#### ICMP Module;

```yaml
  icmp:
    prober: icmp
    icmp:
      preferred_ip_protocol: ip4
  icmp_ttl5:
    prober: icmp
    timeout: 5s
    icmp:
      ttl: 5
```

<figure><img src="../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

[probe\_success 1](#user-content-fn-3)[^3]  parametresinin değeri  1 olduğu için, 10.90.0.144 IP adresine icmp erişiminin olduğunu anlıyoruz.



#### DNS Modüle;

```yaml
  www.google.com:
    prober: dns
    timeout: 5s
    dns:
      transport_protocol: "udp"
      preferred_ip_protocol: "ip4"
      query_name: "www.google.com"
      query_type: "A"
      valid_rcodes:
        - NOERROR
```

<figure><img src="../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

Bu yapılandırma, Blackbox Exporter'ın belirtilen alan adı için bir DNS sorgusu gerçekleştirmesini ve elde edilen yanıtın kontrolünü sağlamasını ifade eder. Yanıtın yanıt kodunu ve diğer parametreleri kontrol ederek, DNS hizmetinin sağlıklı bir şekilde çalışıp çalışmadığını ve beklenen sonuçları verip vermediğini kontrol edebilirsiniz.



#### Scraping targets via Blackbox

```yaml
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Blackbox Exporter'da kullanmak istediğiniz modüller
    static_configs:
      - targets:
        - http://google.com  # İzlemek istediğiniz hedef URL'leri
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115
```

Bu yapılandırmada, aşağıdaki ayarlar kullanılmıştır:

* `job_name: 'blackbox'`: Bu scrape konfigürasyonu için bir iş adı belirtilir. Bu örnekte, "blackbox" olarak belirlenmiştir.
* `metrics_path: /probe`: Blackbox Exporter'dan metrikleri almak için kullanılacak yolu belirtir. Bu örnekte, "/probe" olarak belirlenmiştir.
* `params: module: [http_2xx]`: Blackbox Exporter'ın hangi modülleri kullanacağını belirtir. Bu örnekte, "http\_2xx" modülü belirtilmiştir. Yani, sadece HTTP yanıt kodu 2xx olan hedefleri izleyecektir.
* `static_configs`: İzlenecek hedefleri belirtir.
  * `targets`: İzlemek istediğiniz hedef URL'lerini belirtir. Bu örnekte, "http://google.com" belirtilmiştir.
* `relabel_configs`: Etiket yeniden düzenlemelerini belirtir.

Aşamaları sırayla göstermek için örneğinizi kullanalım:

Aşama 1:

* İlk relabel\_configs adımı: `source_labels: [__address__]`, `target_label: __param_target`
* Değişiklik: `__address__` etiketi `__param_target` etiketiyle değiştirilir.
* Değiştirilmiş Sorgu Linki: `http://localhost:9115/probe?__param_target=www.google.com&module=http_2xx`

Aşama 2:

* İkinci relabel\_configs adımı: `source_labels: [__param_target]`, `target_label: instance`
* Değişiklik: `__param_target` etiketi `instance` etiketiyle değiştirilir.
* Değiştirilmiş Sorgu Linki: `http://localhost:9115/probe?instance=www.google.com&module=http_2xx`

Aşama 3:

* Üçüncü relabel\_configs adımı: `target_label: __address__`, `replacement: localhost:9115`
* Değişiklik: `__address__` etiketi `localhost:9115` ile değiştirilir.
* Değiştirilmiş Sorgu Linki: `http://localhost:9115/probe?instance=www.google.com&module=http_2xx`

Sonuç:

* Sorgu linkinde aşamaların etkisiyle yapılan değişiklikler:
  * `__address__` etiketi `__param_target` ile değiştirilir.
  * `__param_target` etiketi `instance` ile değiştirilir.
  * `__address__` etiketi `localhost:9115` ile değiştirilir.

Bu aşamaların sonunda, sorgu linki `http://localhost:9115/probe?instance=www.google.com&module=http_2xx` şeklinde değişir. Bu değiştirilmiş sorgu linki, scrape işlemi sırasında Blackbox Exporter tarafından kullanılacak ve belirtilen parametrelere göre kontrol yapılacaktır.

Bu yapılandırma, Prometheus'un Blackbox Exporter aracılığıyla belirtilen URL'leri izleyeceğini ve bu hedefler için HTTP modülünü kullanacağını belirtir. İzlenen hedeflerin metriklerini `localhost:9115` adresinden alır ve bu metrikleri scrape ederek Prometheus'a sunar. Bu sayede, belirtilen URL'lerin erişilebilirlik durumu, yanıt süreleri ve diğer ilgili metrikleri Prometheus üzerinden izlenebilir hale gelir.

<figure><img src="../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>



[^1]: 

[^2]: 

[^3]: 
