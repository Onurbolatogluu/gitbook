# 📺 Grafana

#### Ubuntu 20.04 Grafana kurulumu =>&#x20;

1. Öncelikle, yeni bir paket listesi almak için aşağıdaki komutu çalıştırın:

```bash
sudo apt update
```

2. Grafana'yı indirebilmek için apt deposunu ekleyin. Bunun için aşağıdaki komutları kullanın:

```bash
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

3. Şimdi, Grafana'yı yükleyebilirsiniz. Yüklemek için aşağıdaki komutları çalıştırın:

```bash
sudo apt update
sudo apt-get install grafana
```

4. Grafana'yı başlatın ve sistem başladığında otomatik olarak başlamasını sağlayın. Bunun için aşağıdaki komutları kullanın:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

5. Grafana'yı web tarayıcınızdan erişilebilir hale getirmek için, tarayıcınızda `http://your_server_ip:3000` adresini ziyaret edin. Varsayılan kullanıcı adı `admin` ve şifre `admin`'dır.

<figure><img src="../.gitbook/assets/image (128).png" alt=""><figcaption><p>Grafana web arayüzü</p></figcaption></figure>



#### Adding Data source to Grafana

1. Grafana'ya giriş yapın: Grafana'ya giriş yapmak için tarayıcınızın adres çubuğuna `http://your_server_ip:3000` girin. Varsayılan kullanıcı adı ve şifre `admin`'dır.

<figure><img src="../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

1. Sidebar'dan "Data Sources" seçeneğini bulun: Grafana'nın ana ekranında, sol taraftaki yan menüde "Configuration" (ayarlar) simgesini (dişli çark) bulun ve ona tıklayın. Ardından "Data Sources" seçeneğini seçin.

<figure><img src="../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

1. "Add data source" tıklayın: Bu, yeni bir data source eklemenize olanak sağlayacak bir ekranı açacaktır.
2. Data source tipini seçin: Mevcut data source türlerinin listesini göreceksiniz. Bu örnekte, Prometheus'u seçeceğiz.

<figure><img src="../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

1. Detayları girin: Prometheus sunucunuzun URL'sini ve diğer gereken bilgileri girin. URL genellikle `http://<prometheus-server-ip>:9090` şeklindedir.
2. "Save & Test" tıklayın: Grafana, ayarların doğruluğunu kontrol etmek ve data source ile bağlantıyı test etmek için bu butonu kullanır. Eğer her şey doğruysa, "Data source is working" (data source çalışıyor) mesajını göreceksiniz.
3. İşlem tamam! Artık Grafana'ya data source eklediniz ve panolar oluştururken bu data source'dan veri çekebilirsiniz.

<figure><img src="../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

#### Grafana No data problemi çözümü:

<figure><img src="../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

Prometheus yapılandırma dosyasına, grafana servisini target olarak ekliyoruz.

{% code title="/etc/prometheus/prometheus.yml" %}
```yaml
  - job_name: "grafana"
    static_configs:
      - targets: ["10.90.0.138:3000"] # Grafana IP:Port
```
{% endcode %}

Ardından prometheus servisini restart ediyoruz.

```bash
systemctl restart prometheus
```

Servisi yeniden başlattıktan sonra, Grafana ara yüzü kontrol ettiğimizde verilerin geldiğini görüyoruz.&#x20;

<figure><img src="../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>





