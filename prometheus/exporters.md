# 📇 Exporters

Nedir?

Exporters, ölçümlerin üçüncü taraf sistemden (linux veya Windows işletim sistemi gibi) Prometheus ölçümleriyle aynı biçimde dışa aktarılmasına yardımcı olan bir servistir.&#x20;

Exporter, belirli bir hizmet, uygulama veya sistem için tanımlanmış metrikleri toplamak ve bu metrikleri Prometheus veri toplayıcısına aktarmak için tasarlanmış bir araçtır.

Exporter'lar, Prometheus tarafından kullanılabilen bir HTTP endpoint'e sahiptir ve belirli bir metrikle ilişkili olan birçok farklı istekle çağrılabilirler. Prometehus, bir exporter'ın HTTP endpoint'ini belirli bir zaman aralığı boyunca düzenli aralıklarla çağırarak, verileri toplar ve depolar.

Exporter'lar, belirli bir hizmet veya sistem için birçok farklı metrikleri toplayabilir. Örneğin, bir veritabanı exporter'ı, CPU kullanımı, bellek kullanımı, disk kullanımı, sorgu sayısı, bağlantı sayısı gibi metrikleri toplayabilir.&#x20;

Bir web sunucusu exporter'ı ise, istek sayısı, yanıt süresi, hata sayısı gibi metrikleri toplayabilir.&#x20;

Prometheus, bir exporter'ın topladığı verileri, Grafana gibi diğer araçlarla birlikte kullanarak, metrikleri grafikleştirmeye, monitör etmeye veya alarm kurmaya olanak tanır.&#x20;

Exporter'lar, verileri toplama ve sunma işlemini yürütmek için Go, Python, Java gibi farklı dillerde yazılabilir. Prometheus exporter'ları Zabbix agent gibi düşünebiliriz. Her ikisi de monitör edilen hedeflerin belirli metriklerini toplayıp bir merkezi sunucuya gönderirler.

Özetle, sunucudan ölçümler almak için, sunucu da çalışan ve istatistikleri sunan bir veri toplayıcıya (örneğin bir exporter) ihtiyacımız var. Bu nedenle,  misal bir Linux sunucunun CPU veya bellek grafiğini görmek istiyorsak, öncelikle bu sunucuda çalışan bir exporter kurmamız gerekir.



#### Node Exporter

Node Exporter, bir Linux veya Unix işletim sistemi üzerinde çalışan, Prometheus'a sistem ölçümleri sağlamak için kullanılan bir araçtır. Bu ölçümler, işletim sistemi kaynaklarından (CPU, bellek, disk vb.), ağ bağlantılarından, sistem istatistiklerinden ve diğer performans ölçümlerinden elde edilir.&#x20;

Node Exporter, işletim sistemi istatistiklerini çıkarmak için önceden tanımlanmış bir dizi metrik toplar ve Prometheus'a düzenli aralıklarla sunar. Bu ölçümler daha sonra Prometheus tarafından işlenir ve depolanır, böylece bir sunucunun performansı, tarihsel eğilimleri veya beklenmedik değişiklikleri izlemek için kullanılabilir.&#x20;

Node Exporter, bu metriklerin sağladığı bilgileri bir HTTP endpoint aracılığıyla sunar. Bu endpoint'e erişim, Prometheus'un bu metrikleri toplamasına ve depolamasına olanak tanır.

#### Node Exporter Kurulum ( Ubuntu 20.04 )

* **Gerekli dosyanın indirilmesi ve çıkarılması gerekmektedir.**&#x20;
  1. Aşağıdaki komut, öncelikle "/tmp" dizinine geçişi sağlar (eğer yoksa oluşturulur), ardından node\_exporter adlı Prometheus exporter'ının 0.18.1 sürümünün indirilmesini ve açılmasını gerçekleştirir. İndirilen dosya "node\_exporter-_._-amd64.tar.gz" şeklinde bir formatta olduğundan, gerçek dosya adı tam olarak belirtilmez ve yıldız işaretiyle yer değiştirilir. Bu nedenle, "tar xvfz" komutuyla dosya sıkıştırması çözümlenir ve node\_exporter uygulaması /tmp dizininin altındaki bir dizine çıkarılır.

```bash
cd /tmp && wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz && tar xvfz node_exporter-*.*-amd64.tar.gz 
```

* **Dosyaların taşınması ve kullanıcı oluşturulması gerekmektedir.**
  1. Aşağıdaki komut 2 adımdan oluşmaktadır =>
     1. &#x20;`/usr/local/bin/` dizinine `node_exporter` adındaki ana uygulama dosyamızı taşıyoruz.&#x20;
     2. `node_exporter` kullanıcısı oluşturuluyoruz. Bu komutla birlikte, `/usr/local/bin/` dizinine taşınan `node_exporter` dosyasının çalışması için bir kullanıcı oluşturulmaktadır. `-rs /bin/false` argümanları, kullanıcının oturum açmasını engellemektedir. Yani `node_exporter` kullanıcısı, yalnızca `node_exporter` servisini çalıştırmak için kullanılacak bir sistem kullanıcısıdır.

```bash
sudo mv node_exporter-*.*-amd64/node_exporter /usr/local/bin/ && sudo useradd -rs /bin/false node_exporter
```

* **Systemd Dosyasını oluştuyoruz:**

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

Yukarıdaki dosyayı oluşturup, aşağıdaki parametreleri dosyanın içerisine ekliyoruz;

```bash
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
```

* Ardından daemond servisini reload edip,  node exporter servisini çalıştırıp, startup duruma getiriyoruz.

```bash
sudo systemctl daemon-reload &&  sudo systemctl enable node_exporter && sudo systemctl start node_exporter && sudo systemctl status node_exporter
```

<figure><img src="../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

* **Servisin çalışıp, çalışmadığını kontrol etmek için aşağıdaki http endpointe istek gönderiyoruz.**

<pre class="language-bash"><code class="lang-bash"><strong>curl http://&#x3C;server-IP>:9100/metrics
</strong></code></pre>

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

#### Node exporter kurduğumuz sunucuyu, Prometheus 'a eklemek için;

* Prometheus kurulu sunucumuza geçiyoruz ve prometheus.yaml dosyamızı düzenlemek için vi editörü ile dosyamızı açıyoruz.

```bash
vi /etc/prometheus/prometheus.yml
```

Aşağıdaki parametreleri dosyamızın en altına yazıyoruz.

```yaml
  - job_name: "node_exporter"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["10.90.0.144:9100"]
```

Dosyamızın içeriği aşağıdaki şekilde gözükecek;

<figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

* Ardından servisi prometheus servisini restart ediyoruz:

```bash
sudo systemctl restart prometheus
```

* Ardından prometheus arayüzünden hem targets kısmını, hem de promQL ile up sorgusunu gönderip, node exporter yüklediğimiz ubuntu makinemizin durumunu kontrol ediyoruz.

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

Yukarıda görüldüğü üzere bir sorun gözükmüyor.



<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Yukarıda Node exporter yüklediğimiz sunucunun kullanılabilir memory miktarını sorguluyoruz.



#### WMI Exporter

WMI (Windows Management Instrumentation) Exporter, Windows işletim sistemi üzerinde çalışan uygulamaların, Windows yönetim bileşenlerinin, servislerin ve işletim sistemi hakkındaki performans istatistiklerini toplayan ve Prometheus tarafından kullanılabilen bir exporter'dır. Windows WMI aracılığıyla toplanan verileri Prometheus'a sunar. Bu exporter, Windows sunucularının performansının izlenmesi için oldukça yararlıdır ve işletim sistemi, bellek kullanımı, disk ve ağ performansı gibi çeşitli performans ölçütlerini takip etmek için kullanılabilir.

#### WMI Exporter Kurulumu

* [https://github.com/prometheus-community/windows\_exporter/releases/tag/v0.22.0](https://github.com/prometheus-community/windows\_exporter/releases/tag/v0.22.0) > Adresine gidip, işlemci mimarimize uygun olan sürümü indiriyoruz.

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

* Ardından indirdiğimiz exe dosyasını çalıştırıyoruz.

<figure><img src="../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

* Ardından wmi kurduğumuz sunucunun http://sunucuip:9182/metrics adresine gidip, servisin çalışıp, çalışmadığını kontrol ediyoruz.

<figure><img src="../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

* Sıra geldi wmi exporter kurduğumuz sunucuyu prometheus servisimize eklemeye, bunun için prometheus sunucumuza gidip "prometheus.yaml" dosyamıza wmi servisini yüklediğimiz sunucunun bilgilerini yazıp, servisi restart ediyoruz.

<figure><img src="../.gitbook/assets/image (101).png" alt=""><figcaption><p>En alt satıra wmi yüklediğimiz sunucuyu ekliyoruz.</p></figcaption></figure>

```yaml
  - job_name: "wmi_exporter"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["10.90.0.134:9192"]
```

Eklediğimiz kısım yukarıda görüldüğü gibidir.

Ardından prometheus servisimizi restart ediyoruz;

```bash
systemctl restart prometheus
```

* Yeni eklediğimiz sunucuyu, promQL'de up sorgusu ile ve targets sekmesinden kontrol ediyoruz.

<figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Ekran görüntüsü 2023-03-28 194339.png" alt=""><figcaption></figcaption></figure>

Bir sorun gözükmüyor. Aşağıda teyit etmek için disk üzerinde kalan boş alanın sorgusunu gönderiyorum.

<figure><img src="../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
WMI exporter gibi bir araç kullanılarak toplanan verilerde, "windows\_logical\_disk\_free\_bytes" adlı bir metrik, her bir mantıksal disk için mevcut kullanılabilir boş alanı bayt cinsinden içerir. Ancak, bu bilgi bayt olarak verildiği için okunması zor olabilir. Bu nedenle, "windows\_logical\_disk\_free\_bytes" metriğini 1024^3 ile bölerek GB cinsinden daha okunaklı hale getirmek için /1024/1024/1024 kısmı kullanılır.
{% endhint %}

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

#### Bonus Bilgi : prometheus "admin-api" aktif duruma getirmek için,

* Prometheus systemd dosyasını açıp aşağıda işaretlediğim satırı dosyaya ekliyoruz.



<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* Ardından daemon reload ediyoruz ve prometheus servisi yeniden başlatıyoruz.

```bash
systemctl daemon-reload && systemctl restart prometheus
```

Ardından bir instance'a ait verileri temizlemek için, Prometheus sunucunun shellinde aşağıdaki sorguyu çalıştırabilirsiniz.

```bash
curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={instance="10.90.0.134:9192"}'
```
