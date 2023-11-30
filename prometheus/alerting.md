# ⏰ Alerting

Prometheus Alerting, Prometheus'ta tanımlanmış bir dizi kurala göre alarm oluşturmayı sağlayan bir bileşendir.

Prometheus Alerting, metriklerin değerlerini sürekli olarak izler ve tanımlanan kurala uyan metrikler için alarm üretir. Örneğin, bir kural, bir hizmetin yanıt verme süresinin normalde 500 ms'den az olduğunu belirleyebilir ve bu sürenin 500 ms'den yüksek olduğu durumlarda bir alarm üretir.

Alarm yöneticisi, tanımlanan alarm kurallarını gözlemler ve alarm durumlarına göre eylemler gerçekleştirir. Örneğin, bir hizmetin yanıt süresi belirtilen eşiği aşarsa, alarm yöneticisi bir bildirim gönderebilir veya bir dizi eylem gerçekleştirebilir.

Özetle,

Prometheus, belirli aralıklarla birçok farklı hizmetin ölçümlerini toplar ve bu ölçümleri zaman serileri olarak depolar. Bu ölçümler, sunucu kaynaklarının kullanımı, ağ trafiği, uygulama performansı vb. gibi birçok farklı alanda olabilir. Prometheus Alerting, bu ölçümleri kullanarak tanımlanmış kural veya koşullara göre alarm oluşturur. Örneğin, bir kural, bir sunucunun kullanılabilir belleğinin %10'dan az olmaması gerektiğini belirleyebilir ve bu koşul karşılanmadığında bir alarm oluşturabilir.

Bu kural ve koşullar YAML formatında yazılmaktadır.

#### Rule Oluşturmak;

Önceki bölümde oluşturduğumuz rule dosyamızın ismini, " recording\_and\_alerting\_rules\_1.yml " şeklinde güncelliyoruz. Ve içerisine aşağıdaki satırları ekliyoruz.



```yaml
      - alert: NodeExporterDown
        expr: up{job="node_exporter"} == 0
```

```yaml
# rules/recording_and_alerting_rules_1.yml
groups:
  - name: my_rules_1
    rules:
      - record: job:node_cpu_seconds_total:avg_idle
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))

      - alert: NodeExporterDown
        expr: up{job="node_exporter"} == 0

```

YAML dosyasındaki "my\_rules\_1" adlı grup içerisinde, "NodeExporterDown" adlı bir alarm tanımlandık.

Bu alarm, "up{job="node\_exporter"} == 0" PromQL ifadesine göre tetiklenecektir. Bu ifade, "node\_exporter" adlı işin çalışıp çalışmadığını kontrol eder. "up" metriği, bir işin çalışıp çalışmadığını gösteren 1 veya 0 değerini içerir. "job" etiketi, ölçümleri hangi işin sağladığını belirtir.

Eğer "node\_exporter" adlı iş, 0 (çalışmıyor) değerini döndürürse, bu alarm tetiklenecektir. Alarm, bir bildirim veya başka bir işlemle yönetilebilir.

<figure><img src="../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

Örnek olması açısından, bir sunucumuza bağlanıp node\_exporter servisimizi durdurduk. Ardından arayüzü kontrol ettiğimizde "firing" kısmında down olan sunucumuzu görebiliriz.

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

Ek olarak, "ALERTS" sorgusunu çalıştırarak, down durumda olan exporter hakkında bilgi alabiliriz.



#### For

```yaml
groups:
  - name: my_rules_1
    rules:
      - record: job:node_cpu_seconds_total:avg_idle
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))

      - alert: NodeExporterDown
        expr: up{job="node_exporter"} == 0
        for:  1m
```

"For" parametresi, bir alarmın ne kadar süre boyunca devam etmesi gerektiğini belirler. Bu parametre, bir alarmın yanlış bir şekilde tetiklenmesini engellemeye yardımcı olur.

Örneğin, "NodeExporterDown" adlı bir alarm, yukarıdaki örnekte olduğu gibi, "up{job="node\_exporter"} == 0" PromQL ifadesine göre tetiklenecektir. Ancak, "for: 1m" parametresi, alarmın en az 1 dakika boyunca devam etmesi gerektiğini belirtir.

Bu, "node\_exporter" adlı işin geçici bir kesintisi nedeniyle bir alarm tetiklenmesini önlemeye yardımcı olur. Örneğin, bir sistem güncellemesi sırasında "node\_exporter" geçici olarak durabilir ve bu durumda alarm tetiklenebilir. Ancak, "for" parametresi, bu kesintinin geçici olduğunu ve alarmın devam etmesi gerektiğini belirtir.

Bu şekilde, alarm yöneticisi, alarmın yanlış bir şekilde tetiklenmesini önleyebilir ve yalnızca gerçek bir hizmet kesintisi durumunda bir bildirim gönderir.

<figure><img src="../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

"up{job="node\_exporter"} == 0" koşulu sağlandığında hemen bir alarm tetiklenmez. Bunun yerine, alarm durumu "pending" durumuna geçer ve alarm koşulu 1 dakika boyunca sağlandıktan sonra yalnızca bir alarm tetiklenir.

"node\_exporter" adlı işin geçici bir kesintisi olduğunda, alarm durumu "pending" durumuna geçer ve "for" parametresinde belirtilen süre boyunca bekler. Eğer işteki kesinti 1 dakikadan daha kısa sürerse, alarm tetiklenmez ve yanlış bir alarm gönderilmez.



#### Labels

```yaml
groups:
  - name: my_rules_1
    rules:
      - record: job:node_cpu_seconds_total:avg_idle
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))

      - alert: NodeExporterDown
        expr: up{job="node_exporter"} == 0
        for:  1m

      - record: job:app_response_latency_seconds:rate1m
        expr: rate(app_response_latency_seconds_sum[1m]) / rate(app_response_latency_seconds_count[1m])

      - alert: AppLatencyAbove5sec
        expr: job:app_response_latency_seconds:rate1m >= 5
        for: 2m
        labels:
          severity: critical

      - alert: AppLatencyAbove2sec
        expr: 2 < job:app_response_latency_seconds:rate1m < 5
        for: 2m
        labels:
          severity: warning
```

"labels", bir alarm için ek bilgi veya metaveri sağlamak için kullanılan anahtar-değer çiftleridir. Bu etiketler, alarm yöneticisi tarafından kullanılabilen ve alarmı daha iyi yönetmeye yardımcı olan ek bilgiler sağlarlar.

Örneğin, yukarıdaki Prometheus kurallarında, "labels" parametresi kullanılarak iki farklı alarm için etiketler tanımlanmıştır. "severity" adlı bir etiket, her iki alarmda da kullanılmıştır. Bu etiket, bir alarmın önceliğini veya ciddiyetini belirtir.

"AppLatencyAbove5sec" adlı alarm için, "severity: critical" etiketi tanımlanmıştır. Bu etiket, alarmın kritik bir durumu işaret ettiğini belirtir. Bu, alarm yöneticisi tarafından daha hızlı bir şekilde işlenmesi gereken bir durum olduğunu gösterir.

Benzer şekilde, "AppLatencyAbove2sec" adlı alarm için, "severity: warning" etiketi tanımlanmıştır. Bu etiket, alarmın daha az ciddi bir durumu işaret ettiğini belirtir. Bu, alarm yöneticisinin bu durumu daha düşük bir öncelik seviyesinde ele alabileceğini gösterir.

Bu şekilde, "labels" parametresi, alarm yöneticisi tarafından alınacak aksiyonların önceliği veya ciddiyeti hakkında ek bilgi sağlar.

<figure><img src="../.gitbook/assets/image (74).png" alt=""><figcaption><p>Gelen istekler 5 saniyenin üzerinde işlem görüyor.</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption><p>Gelen istekler 2 ila 5 saniyenin arasında işlem görüyor.</p></figcaption></figure>





### Alert Manager

Prometheus Alertmanager, Prometheus ile birlikte kullanılan açık kaynaklı bir uyarı yönetim sistemidir. Alertmanager, Prometheus'tan gelen alarm sinyallerini alır, gruplandırır, filtreler ve bunlara yanıt olarak belirli eylemler gerçekleştirir.

Alertmanager, gelen uyarıları e-posta, SMS, Slack, PagerDuty ve diğer bildirim kanalları aracılığıyla iletebilir. Ayrıca, uyarıların otomatik olarak tekrarlanmasını veya silinmesini sağlayabilir. Alertmanager, karmaşık alarm durumlarını işlemek için özelleştirilebilir ve genişletilebilir bir yapıya sahiptir.

Genel olarak, Prometheus Alertmanager, Prometheus'tan gelen alarm sinyallerini yönetmek, filtrelemek ve yanıtlamak için tasarlanmıştır.&#x20;

#### Kurulum,

{% hint style="info" %}
Kurulacak sunucu üzerinde "prometheus" kullanıcısı ve grubu mevcut değilse, oluşturmalısınız.

```bash
sudo groupadd --system prometheus && sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
{% endhint %}

```bash
cd /tmp && wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz && tar xzf alertmanager-0.25.0.linux-amd64.tar.gz && cd alertmanager-0.25.0.linux-amd64 && mv -v alertmanager amtool /usr/local/bin/ && mkdir -v /etc/alertmanager && mv -v alertmanager.yml /etc/alertmanager && mkdir -v /etc/alertmanager/data && chown -Rfv prometheus:prometheus /etc/alertmanager/
```

Yukarıdaki komut ile "Alert Manager" servisini kuruyoruz. Özetle yukarıdaki komut şunları yapar;

1. /tmp dizinine geçer: "cd /tmp"
2. Alertmanager'ın Linux AMD64 sürümünü indirir: "wget [https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz](https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz)"
3. İndirilen dosyayı açar: "tar xzf alertmanager-0.25.0.linux-amd64.tar.gz"
4. İndirilen dizine geçer: "cd alertmanager-0.25.0.linux-amd64"
5. Alertmanager ve amtool dosyalarını /usr/local/bin dizinine taşır: "mv -v alertmanager amtool /usr/local/bin/"
6. /etc/alertmanager dizinini oluşturur: "mkdir -v /etc/alertmanager"
7. alertmanager.yml dosyasını /etc/alertmanager dizinine taşır: "mv -v alertmanager.yml /etc/alertmanager"
8. /etc/alertmanager/data dizinini oluşturur: "mkdir -v /etc/alertmanager/data"
9. /etc/alertmanager/ dizinini prometheus:prometheus kullanıcısına ve gruba sahip olacak şekilde değiştirir: "chown -Rfv prometheus:prometheus /etc/alertmanager/"

<figure><img src="../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

Alert Manager systemd dosyasını oluşturalım.

```bash
vi /etc/systemd/system/alertmanager.service
```

```bash
[Unit]
Description=Prometheus Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml \
    --storage.path=/etc/alertmanager/data
Restart=always

[Install]
WantedBy=multi-user.target

```

Bu systemd dosyasını kaydettikten sonra, "`systemctl daemon-reload`" komutunu kullanarak systemd'ye dosyayı yeniden yüklemelisiniz. Daha sonra, "`systemctl start alertmanager`" komutunu kullanarak Alertmanager hizmetini başlatabilirsiniz. "`systemctl stop alertmanager`" komutu ile de hizmeti durdurabilirsiniz.

<figure><img src="../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption><p>Alertmanager'in kullandığı 9093 portu cevap veriyor.</p></figcaption></figure>

Yukarıdaki adımları uygulayarak, Alert Manager servisini kurup, çalıştırdık.



Prometheus.yaml dosyamızda, yeni kurduğumuz alert manager servisini tanımlayıp servisi restart ediyoruz.

```bash
vi /etc/prometheus/prometheus.yml
```

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093
```

`systemctl restart prometheus` komutu ile prometheus servisi restart ediyoruz.

<figure><img src="../.gitbook/assets/image (103).png" alt=""><figcaption><p>Alert Manager tanımlı durumda.</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption><p>Ui</p></figcaption></figure>

Alert Manager kurulumu tamamlandı ve çalışır vaziyette olduğunu teyit ettik.



#### Sending email

```yaml
global:
   smtp_require_tls: false

route:
  receiver: admin

receivers:
- name: admin
  email_configs:
  - to: 'example@gmail.com'
    from: 'example@gmail.com'
    smarthost: smtp.example.com:587
    auth_username: 'example@example.com'
    auth_identity: 'example@example.com'
    auth_password: 'fqkvkumorgaqgkat'

```

1. `global` bölümü: Bu bölüm, Alertmanager için küresel yapılandırma ayarlarını içerir. `smtp_require_tls` ayarı, SMTP sunucusuyla iletişim sırasında TLS gerektirilip gerektirilmeyeceğini belirler. Bu örnekte, `false` olarak ayarlandığı için TLS gerekli değildir.
2. `route` bölümü: Prometheus tarafından gönderilen uyarıların nasıl yönlendirileceğini belirler. Bu örnekte, `admin` adlı bir alıcının belirtildiği `receiver` özelliği tanımlanmıştır.
3. `receivers` bölümü:  Bu bölüm, `admin` alıcısının ayrıntılarını içerir. Bu örnekte, e-posta göndermek için kullanılacak SMTP sunucusunun ayrıntıları tanımlanmıştır. `email_configs` özelliği, SMTP sunucusu için gerekli yapılandırma ayarlarını içerir. `to`, e-postanın gönderileceği alıcının e-posta adresini belirtir. `from`, gönderenin e-posta adresini belirtir. `smarthost`, SMTP sunucusunun adını ve bağlantı noktasını belirtir. `auth_username`, SMTP sunucusuna kimlik doğrulama yapmak için kullanılan kullanıcı adını belirtir. `auth_identity`, SMTP sunucusuna bağlanmak için kullanılan kimlik bilgisini belirtir. `auth_password`, SMTP sunucusuna bağlanmak için kullanılan şifreyi belirtir.



<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>





#### annotations

`annotations`, Alertmanager'daki bir uyarıya ek ayrıntılar eklemek için kullanılan bir özelliktir. Bu özellik, bir uyarı hakkında daha ayrıntılı bilgi sağlamak ve alıcının uyarıyı daha iyi anlamasına yardımcı olmak için kullanılabilir.

Alertmanager, `annotations` özelliğinde belirtilen tüm ayrıntıları, bir uyarının gösterildiği arayüzlerde veya uyarı e-postalarında görüntüleyebilir.

```yaml
groups:
  - name: my_rules_1
    rules:
      - record: job:node_cpu_seconds_total:avg_idle
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))

      - alert: NodeExporterDown
        expr: up{job="node_exporter"} == 0
        for:  1m

      - record: job:app_response_latency_seconds:rate1m
        expr: rate(app_response_latency_seconds_sum[1m]) / rate(app_response_latency_seconds_count[1m])

      - alert: AppLatencyAbove5sec
        expr: job:app_response_latency_seconds:rate1m >= 5
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: 'Python app latency is over 5 seconds.'
          description: 'App latency of instance {{ $labels.instance }} of job {{ $labels.job }} is {{ $value }} seconds for more than 5 minutes.'
          app_link: 'http://10.90.0.144:8000/'

      - alert: AppLatencyAbove2sec
        expr: 2 < job:app_response_latency_seconds:rate1m < 5
        for: 2m
        labels:
          severity: warning
```

* `summary`: Bu özellik, uyarı hakkında kısa bir özet sağlar. Bu örnekte, özet, "Python uygulaması gecikmesi 5 saniyenin üzerinde" olarak belirtilmiştir.
* `description`: Bu özellik, uyarıyı daha ayrıntılı bir şekilde açıklar. Bu örnekte, açıklama, belirli bir uygulama örneğinin adını (`{{ $labels.instance }}`), iş adının adını (`{{ $labels.job }}`) ve ölçümleme verilerinin değerini (`{{ $value }}`) içerir. Açıklama, "5 dakikadan daha uzun süre için \{{ $value \}} saniye olan \{{ $labels.job \}} işinin \{{ $labels.instance \}} örneğinin uygulama gecikmesi" olarak belirtilmiştir.
* `app_link`: Bu özellik, uyarının açıklamasında bahsedilen uygulamanın bağlantısını sağlar. Bu örnekte, bağlantı `http://10.90.0.144:8000/` olarak belirtilmiştir.

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

Alarm 'a `annotations`  kullanarak, detaylı bilgiler ekledik ve bu şekilde daha mantıklı uyarılar göndermeye başladı.

