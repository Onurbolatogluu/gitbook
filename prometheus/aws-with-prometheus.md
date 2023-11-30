# 👉 aws with prometheus

Discovery

```yaml
  - job_name: 'ec2'
    ec2_sd_configs:
      - access_key: AIADFGHK6QLLVDF1B6U
        secret_key: H5cIdfGzbxG3ql64h65u5nd6rfy5ZbiXZ/JjgA82
        region: ap-south-1
```

Yukarıdaki prometheus yapılandırması, Prometheus'un "ec2" olarak adlandırılan bir işi (job) tanımladığını göstermektedir. Bu iş, Prometheus tarafından izlenen EC2 sunucuları (Amazon Elastic Compute Cloud) hedef olarak kullanır.

`ec2_sd_configs` bölümü, EC2 hedeflerinin nasıl keşfedileceğini belirtir. Bu örnekte, erişim anahtarı (`access_key`) ve gizli anahtar (`secret_key`) kullanılarak belirli bir bölgeye (`region`) erişim sağlanır. Bu bilgiler, Prometheus'un AWS API'lerine istek göndererek EC2 sunucuları keşfetmesine yardımcı olur.

Bu yapılandırma, Prometheus'un EC2 sunucuları hedef alarak performans metriklerini toplamasını ve izlemesini sağlar. Prometheus, bu ölçümleri kullanarak sunucuların durumunu analiz edebilir, uyarılar oluşturabilir ve metrik verilerini kullanıcıya sunabilir.

Yukarıdaki yapılandırmayı prometheus.yml dosyamıza ekleyip, prometheus servisini restart ettikten sonra, prometheus admin arayüzünde, Service discovery kısmına "ec2" sunucularımıza ait bilgilerin ulaşması gerekmektedir.&#x20;

AWS tarafında ec2 sunucularımız mevcut ise, aşağıdaki görüntü de görebileceğiniz gibi, sunucuya ait verilerin ulaşması gerekmektedir.

<figure><img src="../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

#### Node Exporter

Eğer EC2 sunucusunda bir Debian veya Ubuntu tabanlı işletim sistemi kullanıyorsanız, Prometheus Node Exporter'ı `apt` paket yöneticisi aracılığıyla kurabilirsiniz.&#x20;

1. EC2 sunucusunda ssh ile bağlanın.
2.  Aşağıdaki komutu kullanarak paket deposunu güncelleyin:

    ```
    sudo apt update
    ```
3.  Node Exporter'ı `apt` ile kurmak için aşağıdaki komutu çalıştırın:

    ```
    sudo apt install prometheus-node-exporter
    ```
4. Kurulum tamamlandıktan sonra, Node Exporter otomatik olarak başlatılacaktır.

#### relabel\_configs

```yaml
  - job_name: 'ec2'
    ec2_sd_configs:
      - access_key: AIADFGHK6QLLVDF1B6U
        secret_key: H5cIdfGzbxG3ql64h65u5nd6rfy5ZbiXZ/JjgA82
        region: ap-south-1
    relabel_configs:
      - source_labels: [__meta_ec2_public_ip]
        regex: '(.*)'
        replacement: '${1}:9100'
        target_label: __address__
```

{% hint style="info" %}
Yukarıdaki yml, Prometheus’un AWS’de bulunan EC2 sunucularını otomatik olarak keşfetmesini ve onları metrik toplamak için hedef olarak eklemesini sağlıyor. relabel\_configs bölümü ise hedeflerin adreslerini EC2 sunucularının Public IP adresleri ve 9100 portu ile değiştiriyor. Böylece Prometheus, hedeflerin /metrics yolundan veri alabiliyor.
{% endhint %}

Özetle, ec2\_sd\_configs: parametresi hem EC2 sunucularını keşfedip hem de target olarak ekliyor. relabel\_configs: parametresi ise targetların etiketlerini değiştirmek için kullanılıyor. Bu sayede, Prometheus'un targetlara nasıl ulaşacağını ve onları nasıl tanımlayacağını belirleyebiliyoruz. Örneğin, bu tanım özelinde relabel\_configs: parametresi, targetların adreslerini EC2 sunucularının private IP adreslerinin bulunduğu etikete, ec2 sunucunun public ip adresini ve 9100 portunu ekliyor. Böylece Prometheus, targetların /metrics yolundan veri alabiliyor.



<figure><img src="../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

#### Relabeling KEEP and DROP



Prometheus'un scrape ettiği hedeflerle ilgili olarak "relabel" işlemleri yapabiliriz. Bu relabel işlemleri, hedeflerin etiketlerini değiştirmek, düzenlemek veya filtrelemek için kullanılır. Bu işlemler, Prometheus'un metrikleri gruplamasını ve işlemesini kolaylaştırır.

"relabel" işlemlerinin içinde "drop" ve "keep" adı verilen iki önemli işlem bulunur:

1. `drop`: Belirli bir kurala uyan hedefleri bırakır (silmez). Örneğin, bir hedefin belirli bir etiketi varsa veya belirli bir etiket değerine sahipse, bu hedefleri "drop" işlemiyle çıkarabiliriz. Bu sayede, bu hedeflerin metrikleri scrape edilmez ve Prometheus tarafından işlenmez.
2. `keep`: Belirli bir kurala uyan hedefleri korur (silmez). Örneğin, bir hedefin belirli bir etiketi varsa veya belirli bir etiket değerine sahipse, bu hedefleri "keep" işlemiyle koruyabiliriz. Bu sayede, sadece bu kurala uyan hedeflerin metrikleri scrape edilir ve Prometheus tarafından işlenir.



#### Keep;



`prometheus.yml`

```yaml
  - job_name: 'keep_and_drop'
    file_sd_configs:
      - files:
          - /etc/prometheus/file_sd_config2.yml
        refresh_interval: 30s

    relabel_configs:
      - source_labels: [environment]
        regex: production
        action: keep
```

`file_sd_config2.yml`

```yaml
- targets:
    - 10.90.0.145:9100
  labels:
    environment: production
    job: node-exporter

- targets:
    - 10.90.0.145:9100
  labels:
    environment: staging
    job: node-exporter
```

Bu örnekte, `prometheus.yml` dosyasında `keep_and_drop` adında bir scrape işi (`job`) ve bu iş için `file_sd` hedefleri tanımlanmıştır. `file_sd_configs` bölümünde `file_sd_config2.yml` dosyasının yolu belirtilmiştir ve `refresh_interval` değeri 30 saniye olarak ayarlanmıştır.

`relabel_configs` bölümünde, `environment` etiketi `production` olan hedefler korunacak (keep) şekilde belirtilmiştir. Yani sadece `production` ortamında çalışan hedefler scrape edilecektir. Diğer tüm hedefler, bu işlem sonucunda çıkarılacaktır.

`file_sd_config2.yml` dosyasında, hedefler ve ilgili etiketler YAML formatında tanımlanmıştır. İki hedef grubu bulunmaktadır. İlk hedef grubu `10.90.0.145:9100` adresinde çalışan `node-exporter` işini temsil eder ve `environment` etiketi `production` olarak ayarlanmıştır. İkinci hedef grubu da aynı hedefi ve etiketi (`environment: staging`) kullanmaktadır.

Sonuç olarak, Prometheus, `prometheus.yml` dosyasında tanımlanan scrape işini (job) yürütecek ve `file_sd_config2.yml` dosyasında tanımlanan hedefleri kontrol ederek işleyecektir. `relabel_configs` bölümünde belirtilen kurallara göre sadece `production` ortamında çalışan hedefler scrape edilecek ve diğerleri çıkarılacaktır.

<figure><img src="../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

#### Drop;



`prometheus.yml`

```yaml
  - job_name: 'keep_and_drop'
    file_sd_configs:
      - files:
          - /etc/prometheus/file_sd_config2.yml
        refresh_interval: 30s

    relabel_configs:
      - source_labels: [environment]
        regex: production
        action: drop
```

`file_sd_config2.yml`

```yaml
- targets:
    - 10.90.0.145:9100
  labels:
    environment: production
    job: node-exporter

- targets:
    - 10.90.0.145:9100
  labels:
    environment: staging
    job: node-exporter
```

Bu örnekte, `prometheus.yml` dosyasında `keep_and_drop` adında bir scrape işi (`job`) ve bu iş için `file_sd` hedefleri tanımlanmıştır. `file_sd_configs` bölümünde `file_sd_config2.yml` dosyasının yolu belirtilmiş ve `refresh_interval` değeri 30 saniye olarak ayarlanmıştır.

`relabel_configs` bölümünde, `source_labels` özelliği kullanılarak `environment` etiketi `production` olan hedefler çıkarılır (drop). Yani sadece `environment: production` olan hedefler scrape edilmeyecek ve Prometheus tarafından işlenmeyecektir.

`file_sd_config2.yml` dosyasında, hedefler ve ilgili etiketler YAML formatında tanımlanmıştır. İki hedef grubu vardır. Her iki hedef grubu da aynı `node-exporter` işini temsil eder, ancak `environment` etiketi farklıdır. İlk hedef grubu `environment: production` etiketine sahipken, ikinci hedef grubu `environment: staging` etiketine sahiptir.

Sonuç olarak, Prometheus, `prometheus.yml` dosyasında tanımlanan scrape işini (job) yürütecek ve `file_sd_config2.yml` dosyasında tanımlanan hedefleri kontrol ederek işleyecektir. `relabel_configs` bölümünde belirtilen kurala göre sadece `environment: production` olan hedefler çıkarılacak ve scrape edilmeyecektir.

<figure><img src="../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>



