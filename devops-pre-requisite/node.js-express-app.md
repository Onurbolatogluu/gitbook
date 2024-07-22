# 🟥 Node.js Express App

Node.js Express App, Node.js üzerinde çalışan bir web uygulama framework'ü olan Express kullanılarak oluşturulmuş bir uygulamadır. Express, hızlı, minimalist ve esnek bir yapı sunar, bu sayede geliştiriciler HTTP sunucular ve API'lar oluşturabilirler.

Node.js uygulamalarının processlerini yönetmek için, aşağıda yer alan popüler araçları kullanabilirsiniz.

{% code title="app.js" %}
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```
{% endcode %}

* **supervisord**: Bir process kontrol sistemidir. Express uygulamasını başlatmak ve izlemek için kullanılabilir. Uygulama çökse bile yeniden başlatabilir.

```bash
# 1. Supervisord Kullanım Örneği

# Kurulum
sudo apt-get install supervisor

# Konfigürasyon Dosyası (example.conf)
echo "
[program:example]
command=node /path/to/your/app.js
directory=/path/to/your/
autostart=true
autorestart=true
stderr_logfile=/var/log/example.err.log
stdout_logfile=/var/log/example.out.log
" | sudo tee /etc/supervisor/conf.d/example.conf

# Supervisord'a Konfigürasyon Eklemek
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start example
```

***

* **forever**: Node.js uygulamalarını sürekli çalışır durumda tutmak için kullanılır. Uygulamanın sürekli izlenmesini ve yeniden başlatılmasını sağlar.

```bash
# 2. Forever Kullanım Örneği

# Kurulum
npm install -g forever

# Uygulamayı Çalıştırma
forever start /path/to/your/app.js

# Uygulamayı Durdurma
forever stop /path/to/your/app.js

# Çalışan Uygulamaları Listeleme
forever list
```

***

* **pm2**: Node.js process yöneticisi olan pm2, uygulamaları başlatmak, durdurmak ve yeniden başlatmak için kullanılır. Ayrıca uygulamaların yük dengelemesini ve otomatik yeniden başlatılmasını sağlar.

```bash
# 3. PM2 Kullanım Örneği

# Kurulum
npm install -g pm2

# Uygulamayı Çalıştırma
pm2 start /path/to/your/app.js --name "example-app"

# Uygulamayı İzleme
pm2 monit

# Uygulamayı Durdurma
pm2 stop example-app

# Uygulamayı Yeniden Başlatma
pm2 restart example-app

# PM2 ile Uygulama Konfigürasyonunu Kaydetme
pm2 save

# PM2'yi Sistem Başlangıcına Eklemek
pm2 startup
```



