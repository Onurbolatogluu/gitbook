# 💚 Services | Linux Basics #5

Linux Services, sistemin kritik işlevlerini yerine getiren arka plan süreçleridir. `systemd` kullanılarak bu servisler kolayca yönetilebilir. Servislerin başlatılması, durdurulması, yeniden başlatılması ve sistem başlatıldığında otomatik olarak çalışacak şekilde ayarlanması gibi işlemler yapılabilir.

```bash
# HTTPD hizmetini başlatmak için kullanılır. Bu iki komut aynı işi yapar, 
# ancak systemctl daha modern ve tercih edilen yöntemdir.
service httpd start
systemctl start httpd

# HTTPD hizmetini durdurmak için kullanılır.
systemctl stop httpd

# HTTPD hizmetinin durumunu kontrol etmek için kullanılır. 
# Hizmetin çalışıp çalışmadığını ve varsa hata mesajlarını gösterir.
systemctl status httpd

# HTTPD hizmetini sistem başlatıldığında otomatik olarak çalışacak şekilde ayarlamak için kullanılır.
systemctl enable httpd

# HTTPD hizmetini sistem başlatıldığında otomatik olarak çalışmayacak şekilde ayarlamak için kullanılır.
systemctl disable httpd
```

Linux sistemlerinde servisler, eskiden init scripts adı verilen scriptler ile yönetilirdi. Bu scriptler, her servisi başlatmak, durdurmak ve durumunu kontrol etmek için kullanılırdı. Init scripts basit ve shell script olarak yazılırdı, bu da yönetimi kolaylaştırırdı. Ancak, bu yöntem servisleri sıralı olarak başlattığı için sistemin açılış süresi uzar ve servisler arası bağımlılıkları ve hataları yönetmek zor olurdu.

Öte yandan, systemd modern Linux dağıtımları için tasarlanmış bir servis yöneticisidir. Servisleri bağımlılıklarına göre paralel olarak başlatabilir, bu da sistemin daha hızlı açılmasını sağlar. Ayrıca, systemd bağımlılıkları daha iyi yönetir ve Servislerin daha kararlı çalışmasını sağlar. Yapılandırma dosyaları INI formatındadır. Systemd sadece hizmet yönetimiyle kalmaz, aynı zamanda log yönetimi, cihaz yönetimi gibi birçok ek işlev sunar.&#x20;

Sonuç olarak, init scripts basit ve geleneksel bir yöntemken, systemd modern sistemler için daha uygun, hızlı çözümdür. Bu yüzden çoğu modern Linux dağıtımı systemd'yi kullanmaktadır.



#### Python Flask uygulamamızı systemd servisi haline getirelim;

* &#x20;Örneğin, `app.py` adında basit bir Flask uygulamanız olduğunu varsayalım.

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

```

* Flask'ı sisteme kurulu hale getirelim.

```bash
pip install flask
```

* `/etc/systemd/system/` dizininde yeni bir servis dosyası oluşturun. Örneğin, `flaskapp.service` adında bir dosya oluşturacağız.

```bash
sudo vi /etc/systemd/system/flaskapp.service
```

* Aşağıdaki örnek içeriği servis dosyasına yapıştırın. Bu içerik, Flask uygulamanızı doğrudan Python kullanarak systemd hizmeti olarak çalıştıracaktır.

{% code title="flaskapp.service" %}
```ini
[Unit]
Description=A simple Flask web application
After=network.target

[Service]
User=your_user
WorkingDirectory=/path/to/your/app
ExecStart=/usr/bin/python3 /path/to/your/app/app.py

[Install]
WantedBy=multi-user.target

```
{% endcode %}

* `User=your_user`: Buraya servisi çalıştıracak kullanıcının adını yazın.
* `WorkingDirectory=/path/to/your/app`: Buraya Flask uygulamanızın bulunduğu dizinin yolunu yazın.
* `ExecStart=/usr/bin/python3 /path/to/your/app/app.py`: Bu komut, Flask uygulamanızı Python kullanarak çalıştırır.

{% hint style="info" %}
"your\_user", Flask uygulamanızı çalıştırmak için kullanılacak kullanıcı adıdır. Bu, sisteminizde mevcut olan bir kullanıcı adı olmalıdır. Genellikle, bu kullanıcı uygulamanın çalıştırıldığı dizin ve dosyalara erişim iznine sahip bir kullanıcı olur.
{% endhint %}

* Servisi etkinleştirin ve başlatın.

```bash
# systemd'yi yeniden yükleyin
sudo systemctl daemon-reload

# Servisi etkinleştirin
sudo systemctl enable flaskapp

# Servisi başlatın
sudo systemctl start flaskapp

# Servisin durumunu kontrol edin
sudo systemctl status flaskapp

```

Bu adımları izleyerek, Python Flask uygulamanızı systemd servisi olarak çalıştırabilirsiniz. Bu yöntemle Flask uygulamanız doğrudan sistem genelinde kurulu Python ile başlatılır ve systemd tarafından yönetilir.

***

#### Sample Systemd File,

```ini
# /lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket

# Unit sekmesi, servisin ne olduğunu, hangi servislere bağlı olduğunu ve hangi servislerden sonra başlatılması gerektiğini belirtir.
# Description: Hizmetin kısa açıklaması.
# Documentation: Hizmetle ilgili dökümantasyonun bulunduğu URL.
# BindsTo: Bu servis containerd.service ile bağlantılıdır. containerd.service durduğunda bu servis de durur.
# After: Bu servis, network-online.target, firewalld.service ve containerd.service servislerinden sonra başlatılmalıdır.
# Wants: Bu servis network-online.target servisini de başlatmak ister.
# Requires: Bu servis, docker.socket servisine ihtiyaç duyar. Bu servis olmadan çalışmaz.

[Service]
Type=notify
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Service sekmesi, servisin nasıl başlatılacağını, durdurulacağını ve yeniden yükleneceğini belirtir.
# Type: Servisin başlatılma tipini belirtir. "notify" servisin systemd'ye başlatılma durumunu bildireceği anlamına gelir.
# ExecStart: Servisin başlatılacağı komut. Burada Docker daemon (dockerd) başlatılıyor.
# ExecReload: Servisin yeniden yüklenmesi için kullanılacak komut. Burada ana süreç (MAINPID) yeniden başlatılıyor.
# Restart: Servisin her zaman yeniden başlatılmasını sağlar.
# StartLimitBurst: Belirli bir zaman aralığında kaç kez yeniden başlatılabileceğini belirtir.
# StartLimitInterval: Yeniden başlatma sınırının zaman aralığı.
# LimitNOFILE: Bu servis için açık dosya sayısı sınırı.
# LimitNPROC: Bu servis için işlem sayısı sınırı.
# LimitCORE: Bu servis için core dump dosyası sınırı.

[Install]
WantedBy=multi-user.target

# Install sekmesi, servisin hangi hedeflerle (target) birlikte başlatılacağını belirtir.
# WantedBy: Bu servis, multi-user.target ile birlikte başlatılacak. multi-user.target, sistemin çok kullanıcı modunda çalıştığı anlamına gelir.
```

