# 🛃 Custom exporter with Python

Aşağıdaki kod, Prometheus Python Client kütüphanesini kullanarak /tmp dizinindeki dosya sayısını izleyen özel bir exporter oluşturur.

{% code title="custom-exporter.py" %}
```python
import os
from prometheus_client import start_http_server, Gauge

# Dosya sayısını izlemek için bir ölçer (Gauge) oluşturun
file_count = Gauge('tmp_files_count', 'Number of files in /tmp directory')

# /tmp dizinindeki dosya sayısını almak için bir işlev tanımlayın
def get_file_count():
    file_list = os.listdir('/tmp')
    return len(file_list)

# Metrikleri güncellemek için bir işlev tanımlayın
def update_metrics():
    file_count.set(get_file_count())

# HTTP sunucusunu başlatın ve metrikleri güncelleyin
if __name__ == '__main__':
    # Prometheus metriklerini yayınlamak için bir HTTP sunucusu başlatın
    start_http_server(9009)

    # Metrikleri güncellemek için bir döngü başlatın
    while True:
        # Her döngüde dosya sayısını güncelleyin
        update_metrics()

```
{% endcode %}

1. İlk olarak, `os` ve `prometheus_client` modüllerini içe aktarıyoruz. `os` modülü, işletim sistemi işlevlerini kullanmamıza izin verirken, `prometheus_client` modülü Prometheus metriklerini oluşturmak ve yayınlamak için kullanılır.
2. `file_count` adında bir ölçer (Gauge) oluşturuyoruz. Bu ölçer, `/tmp` dizinindeki dosyaların sayısını temsil edecektir. İsim ve açıklama parametreleriyle birlikte `Gauge` sınıfından bir örnek oluşturulur.
3. `get_file_count()` adında bir işlev tanımlanır. Bu işlev, `/tmp` dizinindeki dosyaların sayısını almak için `os.listdir('/tmp')` işlevini kullanır. `len(file_list)` ifadesi, `file_list` listesinin uzunluğunu döndürür, yani `/tmp` dizinindeki dosyaların sayısını verir.
4. `update_metrics()` adında bir işlev daha tanımlanır. Bu işlev, `get_file_count()` işlevini çağırarak `/tmp` dizinindeki dosya sayısını alır ve `file_count` ölçerini günceller. `file_count.set()` işlevi, ölçeri güncel değerle ayarlar.
5. `if __name__ == '__main__':` bloğu, Python betiğinin doğrudan çalıştırıldığında (yani başka bir betik tarafından içe aktarılmadığında) çalışacak kodu içerir. Bu blokta, `start_http_server(9009)` ifadesiyle bir HTTP sunucusu başlatılır ve Prometheus metriklerini `9009` numaralı portta yayınlar.
6. Sonsuz bir döngü içinde `update_metrics()` işlevi çağrılır. Bu sayede `/tmp` dizinindeki dosya sayısı periyodik olarak güncellenir ve Prometheus tarafından toplanabilir.

Bu şekilde çalışan bir örnek, `/tmp_files_count` adıyla bir Prometheus metriğini yayınlayacak ve bu metrik `/tmp` dizinindeki dosya sayısını gösterecektir. `/metrics` yoluna yapılan HTTP istekleriyle bu metriği görüntüleyebilirsiniz.



Uygulamayı çalıştıralım,

```bash
/usr/bin/python3 custom-exporter.py
```

<figure><img src="../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

Prometheus'a özel exporter eklemek için aşağıdaki adımları izleyebilirsiniz:

1. Prometheus yapılandırma dosyasını düzenleyin: Prometheus'un yapılandırma dosyasını (varsayılan olarak `prometheus.yml`) açın ve düzenleyin.
2. Yapılandırma dosyasına hedef ekleyin: `scrape_configs` bölümünde, `job_name` ve `static_configs` ayarlarını kullanarak bir hedef ekleyin. Örneğin:

```yaml
scrape_configs:
  - job_name: 'custom_exporter'
    static_configs:
      - targets: ['10.90.0.144:9009']
```

Yukarıdaki örnekte, `custom_exporter` adında bir iş tanımı (job) ekledik ve hedef olarak `10.90.0.144:9009` adresini belirttik. `9009`, örnek kodunuzda HTTP sunucusunu başlattığınız port numarasına karşılık gelmelidir.

3. Prometheus'u yeniden başlatın: Yapılandırma dosyasını kaydedin ve Prometheus'u yeniden başlatın, böylece yapılandırma değişiklikleri yürürlüğe girecektir.
4. Prometheus arayüzünde metrikleri kontrol edin: Prometheus arayüzünü tarayıcınızda açın.&#x20;
5. Metrikleri sorgulayın: Prometheus arayüzünde, metrikleri sorgulayabilir ve görselleştirebilirsiniz. Örneğin, `custom_exporter` adını kullanarak `/metrics` yoluna yapılan isteklerin sonucunu görebilirsiniz.

Prometheus, yapılandırma dosyasında belirtilen hedefleri düzenli aralıklarla sorgular ve metrikleri toplar. Bu şekilde, özel ihracatçınızın sağladığı metrikleri alabilir ve izleyebilirsiniz. İlgili yapılandırmayı yaptıktan sonra, Prometheus, belirttiğiniz IP adresi olan `10.90.0.144` üzerinde çalışan özel exporter metrikleri alacaktır.

<figure><img src="../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

Düzgün çalışıp, çalışmadığını anlamak için /tmp dizininde random dosyalar oluşturalım.

```bash
for i in {1..15}; do
    file_name=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 8 | head -n 1)
    touch "/tmp/$file_name"
    echo "Oluşturulan dosya: $file_name"
done
```

Yukarıdaki komut, döngü kullanarak 15 defa işlem yapar. Her adımda, `file_name` değişkeni rasgele bir dosya adı oluşturmak için `/dev/urandom` kullanır. `tr` komutuyla yalnızca harf ve rakamlardan oluşan karakterler seçilir ve `fold` komutuyla 8 karakter uzunluğunda bir satır oluşturulur. `head` komutuyla bu satırdan ilk satır seçilir ve `file_name` değişkenine atanır.

Sonra `touch` komutuyla, `file_name` değeri kullanılarak dosya oluşturulur. Oluşturulan dosya adı da ekrana yazdırılır.

Bu shell komutunu çalıştırdığınızda, `/tmp` dizininde 15 tane rasgele dosya oluşturulacak ve her dosya adı ekrana yazdırılacaktır.

<figure><img src="../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

Aşağıda göreceğiniz üzere, dosya sayısı 205 olarak güncellenmiş gözüküyor.

<figure><img src="../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

