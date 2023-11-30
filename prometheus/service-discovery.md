# 🪒 Service Discovery

Prometheus servis keşfi, Prometheus'un bir zaman serisi veri toplama ve izleme aracı olarak kullanıldığı ortamlarda, hedef servislerin otomatik olarak keşfedilmesini sağlayan bir özelliktir. Prometheus, hedef olarak izlenmesi gereken servislerin konumlarını ve özelliklerini dinamik olarak bulabilir ve veri toplama hedeflerine ekleyebilir.

Prometheus, servislerin keşfedilmesi için farklı keşif yöntemlerini destekler. Bunlar şunları içerir:

1. Hedef Servis Tanımları: Prometheus, hedef servisleri belirli bir dosyada veya dizinde tanımlayarak keşfedebilir. Bu tanımlar, hedef servislerin etiketlerini, URL'lerini, port numaralarını ve diğer özelliklerini içerebilir.
2. Servis Keşif Protokolleri: Prometheus, DNS, Kubernetes, Consul, EC2 gibi servis keşif protokollerini destekler. Bu protokoller, servislerin adını çözümleyerek veya belirli bir yapılandırma servisi üzerinden sorgulayarak hedefleri keşfeder.
3. Servis Keşif Sağlayıcıları: Prometheus, servis keşif sağlayıcıları aracılığıyla da hedef servisleri keşfedebilir. Örneğin, Kubernetes'e özgü sağlayıcılar, Kubernetes API'sini kullanarak çalışan hedef servisleri otomatik olarak bulabilir.

Prometheus servis keşfi, dinamik ortamlarda çalışan birçok servisin izlenmesini kolaylaştırır. Servislerin otomatik olarak keşfedilmesi, yapılandırma dosyalarını veya elle yapılan ayarları güncelleme ihtiyacını ortadan kaldırır ve yeni servisler eklenirken izleme sürecini otomatikleştirir. Böylece, Prometheus'un topladığı verilerin güncel ve doğru kalması sağlanır.



#### File Based:

<figure><img src="../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

prometheus.yml

```yaml
  - job_name: 'discover'
    file_sd_configs:
      - files:
          - /etc/prometheus/file_sd_config.yml
```

file\_sd\_config.yml

```yaml
# Örnek 1: Statik Servis Keşfi
- targets:
  - 10.90.0.145:9100
  labels:
    job: devops
    location: Turkey
```

Ardından, servisi restart ediyoruz. `systemctl restart prometheus`

<figure><img src="../.gitbook/assets/image (163).png" alt=""><figcaption><p>Görüldüğü üzere, keşif yaptığmız host 'a ait bilgiler prometheus'a ulaşmış.</p></figcaption></figure>

