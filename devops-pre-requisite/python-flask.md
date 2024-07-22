# 🟫 Python Flask

Flask, Python programlama dili ile yazılmış sade bir web framework'tür. Basit ve esnek bir yapıya sahip olan Flask, özellikle küçük ve orta ölçekli web uygulamaları geliştirmek için idealdir. Flask, minimal bir çekirdek üzerine inşa edilmiş olup, geliştiricilere istedikleri eklentileri kolayca ekleyebilme esnekliği sağlar.

Flask uygulamalarını prod ortamlarda çalıştırmak için birkaç popüler araç ve teknik kullanılır.

* **Gunicorn**
* **uWSGI**
* **Gevent**
* **Twisted Web**

#### Örnek Flask Uygulaması

Aşağıda basit bir Flask uygulaması örneği bulunmaktadır:

```python
# main.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run()
```

#### 1. Flask Uygulamasını Gunicorn ile Çalıştırmak

{% hint style="info" %}
Gunicorn (Green Unicorn), Python programlama dili için bir WSGI HTTP sunucusudur.
{% endhint %}

Flask uygulamasını Gunicorn ile çalıştırmak için aşağıdaki komutu kullanabilirsiniz:

```bash
gunicorn main:app
```

Bu komut, `main.py` dosyasındaki `app` adlı Flask uygulamasını Gunicorn kullanarak çalıştırır. Aşağıdaki gibi bir çıktı alırsınız:

```bash
[2020-03-10 09:53:29 +0000] [2916] [INFO] Starting gunicorn 20.0.4
[2020-03-10 09:53:29 +0000] [2916] [INFO] Listening at: http://127.0.0.1:8000 (2916)
[2020-03-10 09:53:29 +0000] [2916] [INFO] Using worker: sync
[2020-03-10 09:53:29 +0000] [2919] [INFO] Booting worker with pid: 2919
```



#### 2. uWSGI

uWSGI, Python web uygulamalarını prod ortamında çalıştırmak için kullanılan güçlü bir sunucudur. Flask uygulamasını uWSGI ile çalıştırmak için şu komutu kullanabilirsiniz:

```bash
uwsgi --http :8000 --module main:app
```



#### **3. Gevent**

Gevent, Python'da eşzamanlı uygulamalar geliştirmek için kullanılan bir kütüphanedir. Flask uygulamasını Gevent ile çalıştırmak için aşağıdaki komutu kullanabilirsiniz:

```bash
gunicorn main:app -k gevent
```



#### **4. Twisted Web**

```bash
twistd -n web --class=main.app
```



