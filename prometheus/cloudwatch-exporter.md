# ☁ CloudWatch Exporter

**Adım 1: CloudWatch JAR'ını indirin**

```shell
wget -O cloudwatch_exporter.jar http://search.maven.org/remotecontent?filepath=io/prometheus/cloudwatch/cloudwatch_exporter/0.8.0/cloudwatch_exporter-0.8.0-jar-with-dependencies.jar
```

Bu komut, CloudWatch Exporter'ın JAR dosyasını indirir ve `cloudwatch_exporter.jar` olarak kaydeder.

**Adım 2: Java'yı yükleyin**

```shell
sudo apt-get install -y openjdk-11-jre-headless
```

Bu komut, OpenJDK 11'in JRE (Java Runtime Environment) sürümünü yükler.

**Adım 3: AWS kimlik bilgilerini yapılandırın**

```shell
mkdir -p ~/.aws/
vim ~/.aws/credentials
```

Bu komutlar, `.aws` adlı bir dizin oluşturur ve `credentials` dosyasını düzenler. Vim düzenleyicisi açılacak ve dosyanın içeriğini düzenlemeniz gerekecek.

Dosyanın içeriğini aşağıdaki gibi düzenleyin:

```
[default]
aws_access_key_id=YOUR_ACCESS_KEY
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
```

`YOUR_ACCESS_KEY` ve `YOUR_SECRET_ACCESS_KEY` değerlerini kendi AWS kimlik bilgilerinizle değiştirin. Bu kimlik bilgileri, CloudWatch API'lerine erişmek için kullanılacak.

**Adım 4: Prometheus yapılandırma dosyasını düzenleyin**

```shell
vim /path/to/prometheus.yml
```

Bu komutla Prometheus yapılandırma dosyasını açarsınız. `/path/to/prometheus.yml` yerine Prometheus'un yüklü olduğu dizini ve `prometheus.yml` dosyasının yolunu belirtmelisiniz.

Dosyanın içeriğine aşağıdaki yapılandırmaları ekleyin:

```yaml
- job_name: cloudwatch
  static_configs:
    - targets: ['localhost:9106']
```

Bu yapılandırma, Prometheus'un CloudWatch Exporter'ı hedef olarak kullanmasını sağlar. Exporter'ın varsayılan olarak `localhost:9106` adresinde çalışacağını belirtir. Bu yapılandırmayı ekleyip prometheus servisini yeniden başlatıyoruz.

&#x20;`systemctl restart prometheus`

**Adım 5: CloudWatch Exporter'ı çalıştırın**

CloudWatch Exporter için örnek yapılandırma dosyası (`cloudwatchexporter.yml`):

```yaml
region: us-west-1  # İlgili AWS bölgesini burada belirtin.
metrics:
  - aws_namespace: AWS/ELB
    aws_metric_name: RequestCount
    aws_dimensions: [AvailabilityZone, LoadBalancerName]
    aws_statistics: [SampleCount, Average]
    aws_period: 300
    aws_unit: Count
  - aws_namespace: AWS/EC2
    aws_metric_name: CPUUtilization
    aws_dimensions: [InstanceId]
    aws_statistics: [SampleCount, Average]
    aws_period: 300
    aws_unit: Percent
```

Bu `cloudwatchexporter.yml` dosyası örneğinde, `region` değerini kendi kullanmak istediğiniz AWS bölgesine ayarlayabilirsiniz. Ardından, örnek metrikler eklenmiştir. Bu örnekte, AWS/ELB ve AWS/EC2 adında iki adet metrik bulunmaktadır. Siz kendi ihtiyaçlarınıza göre bu metrikleri ve özelliklerini düzenleyebilirsiniz. Metrikleri ekleyebilir, kaldırabilir veya değiştirebilirsiniz.

{% hint style="info" %}
* `region`: AWS bölgesini belirtir. Bu örnekte "us-west-1" olarak belirtilmiştir.
* `metrics`: CloudWatch Exporter'ın topladığı metriklerin yapılandırmasını içerir. Her bir metrik için aşağıdaki özellikler belirtilir:
  * `aws_namespace`: Metriğin bulunduğu AWS hizmetinin adı.
  * `aws_metric_name`: Toplanacak metriğin adı.
  * `aws_dimensions`: Metriğin boyutlarını belirtir.
  * `aws_statistics`: Toplanan metriğin istatistik türlerini belirtir.
  * `aws_period`: Metriğin toplandığı dönem aralığı.
  * `aws_unit`: Metriğin birimi.
{% endhint %}

Bu `cloudwatchexporter.yml` dosyasını CloudWatch Exporter ile çalıştırdığınız komuta eklemeniz gerekmektedir. Yani, CloudWatch Exporter'ı çalıştırmak için kullanacağınız komutu şu şekilde güncellemeniz gerekmektedir:

```shell
/usr/bin/java -jar cloudwatch_exporter.jar 9106 /path/to/cloudwatchexporter.yml
```

Bu komutla CloudWatch Exporter'ı çalıştırırsınız. Exporter, `localhost:9106` adresindeki HTTP sunucusunda dinlemeye başlar. Bu şekilde CloudWatch Exporter, `cloudwatchexporter.yml` dosyasındaki yapılandırmaları kullanarak metrikleri toplayacak ve Prometheus'a sunacak.

Bu adımları izleyerek CloudWatch Exporter'ı Prometheus ile kullanabilirsiniz.

<figure><img src="../.gitbook/assets/Screen Shot 2023-05-31 at 4.59.33 PM.png" alt=""><figcaption></figcaption></figure>
