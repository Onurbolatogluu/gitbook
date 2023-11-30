# 🦯 Ubuntu 20.04 Prometheus Kurulum

* **Aşağıdaki komut ile iki adet dizin oluşturun:**&#x20;

{% hint style="info" %}
/etc/prometheus dizini Prometheus konfigürasyon dosyaları için kullanılır. Bu dosyalar, Prometheus'un scrape edeceği hedeflerin tanımlarını içerir.

/var/lib/prometheus dizini, Prometheus'un depolama dizini olarak kullanılır. Prometheus, scrape ettiği metrikleri burada saklar.
{% endhint %}

```bash
sudo mkdir -p /etc/prometheus && sudo mkdir -p /var/lib/prometheus
```

* **Ardından aşağıdaki komutu çalıştırın, aşağıdaki komut iki işlemi gerçekleştirir:**
  1. "wget" komutu, belirtilen URL'den Prometheus'un belirli bir sürümünü (2.43.0) indirir. "[https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz](https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz)" adresindeki sıkıştırılmış Prometheus dosyalarını indirir.
  2. "tar" komutu, indirilen dosyayı açar. "-xvf" parametreleri, sıkıştırılmış dosyayı açmak ve çıktıları konsola göstermek için kullanılır. Bu işlem sonucunda, "prometheus-2.43.0.linux-amd64" dizini oluşturulur ve bu dizinde Prometheus uygulamasının çalıştırılması için gerekli dosyalar bulunur.

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz && tar -xvf prometheus-2.43.0.linux-amd64.tar.gz
```

* **Ardından indirdiğimiz uygulama dosyaları içerisinde, ihtiyacımız olan dosyaları kullanıyoruz. Aşağıdaki komutu çalıştırıyoruz. Aşağıdaki komut;**&#x20;
  1. "cd" komutu, çalışma dizinini "prometheus-2.31.3.linux-amd64" dizinine değiştirir.&#x20;
  2. "sudo mv prometheus promtool /usr/local/bin/" komutu, prometheus ve promtool dosyalarını /usr/local/bin/ dizinine taşır. Bu dizin,  kullanıcıların yürütülebilir dosyalarını sakladığı bir sistem dizinidir.
  3. "sudo mv consoles/ console\_libraries/ /etc/prometheus/" komutu, consoles ve console\_libraries dizinlerini /etc/prometheus/ dizinine taşır. Bu dizinler, Prometheus'un grafik arayüzü için kullanılan dosyaları içerir.
  4. "sudo mv prometheus.yml /etc/prometheus/prometheus.yml" komutu, prometheus.yml dosyasını /etc/prometheus/ dizinine taşır. Bu dosya, Prometheus'un scrape edeceği hedeflerin ve scrape işlemleri için kullanılacak özelliklerin tanımlandığı dosyadır.

{% hint style="info" %}
Bu işlemlerin sonucunda, Prometheus'un dosyaları ve konfigürasyon dosyaları, sistemde doğru konumlara taşınmış olur ve Prometheus uygulaması başlatılabilir hale gelir.
{% endhint %}

```bash
cd prometheus-2.31.3.linux-amd64 && sudo mv prometheus promtool /usr/local/bin/ && sudo mv consoles/ console_libraries/ /etc/prometheus/ && sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

* **Aşağıdaki komut ile, Prometheus ve promtool uygulamalarının sürüm numaralarını görüntülüyoruz.**&#x20;
  1. "prometheus --version" komutu, yüklü Prometheus sürümünün numarasını gösterir. Bu sürüm numarası, Prometheus'un çalıştığı sürüm hakkında bilgi verir.
  2. "promtool --version" komutu, yüklü promtool sürümünün numarasını gösterir. Promtool, Prometheus araç setinin bir parçasıdır ve konfigürasyon dosyalarının doğruluğunu kontrol etmek ve metric değerlerini test etmek için kullanılır.

{% hint style="info" %}
Bu komutlar, Prometheus ve promtool'un doğru şekilde yüklendiğini ve çalıştırıldığını doğrulamak için kullanılabilir.
{% endhint %}

```bash
prometheus --version && promtool --version
```

* **Aşağıdaki komut ile, Prometheus için bir grup ve kullanıcı hesabı oluşturup ve bu hesapların gerekli dizinlere erişim haklarını ayarlıyoruz. Aşağıdaki komut;**
  1. "sudo groupadd --system prometheus" komutu, "prometheus" adında bir sistem grubu oluşturur. Bu grup, yalnızca sistem hesapları tarafından kullanılabilir.
  2. "sudo useradd -s /sbin/nologin --system -g prometheus prometheus" komutu, "prometheus" adında bir sistem kullanıcısı oluşturur. Bu kullanıcı, sistemde bir oturum açamaz ve sadece "prometheus" grubuna aittir.
  3. "sudo chown -R prometheus:prometheus /etc/prometheus/ /var/lib/prometheus/" komutu, /etc/prometheus/ ve /var/lib/prometheus/ dizinlerinin sahibini "prometheus" kullanıcısı ve "prometheus" grubu olarak ayarlar. Bu, bu dizinlerdeki tüm dosyaların ve alt dizinlerin "prometheus" kullanıcısı tarafından okunabilir ve yazılabilir olmasını sağlar.
  4. "sudo chmod -R 775 /etc/prometheus/ /var/lib/prometheus/" komutu, /etc/prometheus/ ve /var/lib/prometheus/ dizinlerinin izinlerini ayarlar. Bu dizinlerin sahibi ve grubu, içindeki dosyaları okuma, yazma ve çalıştırma izni verirken, diğer kullanıcılar için sadece okuma ve çalıştırma izni verir. Bu, sadece "prometheus" kullanıcısının bu dizinlerdeki dosyalara yazabileceği ve değişiklik yapabileceği anlamına gelir.

{% hint style="info" %}
Bu işlemlerin sonucunda, Prometheus, "/etc/prometheus" ve "/var/lib/prometheus" dizinlerine erişebilecek ve bu dizinlerde değişiklik yapabilecek "prometheus" kullanıcısı ve grubu oluşturulur.
{% endhint %}

```bash
sudo groupadd --system prometheus && sudo useradd -s /sbin/nologin --system -g prometheus prometheus && sudo chown -R prometheus:prometheus /etc/prometheus/ /var/lib/prometheus/ && sudo chmod -R 775 /etc/prometheus/ /var/lib/prometheus/
```

* **Ardından prometheus için systemd dosyası oluşturuyoruz.**

```
sudo vim /etc/systemd/system/prometheus.service
```

&#x20;Yukarıdaki oluşturduğumuz dosya içerisine aşağıdaki parametreleri ekliyoruz;

```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=always
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
```

{% hint style="info" %}
Yukarıdaki dosyadaki yapılandırma ayarları, Prometheus'u sistem başlatıldığında otomatik olarak başlatır ve herhangi bir arıza durumunda yeniden başlatır. Ayrıca, Prometheus'un yapılandırma dosyasının yeri, depolama dizini, web konsolu şablonları ve kütüphaneleri ve dinlenecek adres ve port gibi diğer ayarları da belirler.
{% endhint %}

* **Ardından aşağıdaki komut ile servisi başlatıyoruz ve durumunu kontrol ediyoruz.**

```bash
sudo systemctl start prometheus && sudo systemctl enable prometheus && sudo systemctl status prometheus
```

<figure><img src="../.gitbook/assets/Ekran görüntüsü 2023-03-28 142424.png" alt=""><figcaption></figcaption></figure>

* **Ardından tarayıcı üzerinden prometheus arayüzüne bağlanabiliriz;**

```web-idl
http://server-ip:9090
```

<figure><img src="../.gitbook/assets/Ekran görüntüsü 2023-03-28 142136.png" alt=""><figcaption></figcaption></figure>

Official Docs: [https://prometheus.io/download/](https://prometheus.io/download/)

