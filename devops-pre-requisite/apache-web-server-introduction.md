# 🟧 Apache Web Server Introduction

<figure><img src="../.gitbook/assets/https___dev-to-uploads.s3.amazonaws.com_uploads_articles_7dvu3xzvepdcy1919s2q.avif" alt=""><figcaption></figcaption></figure>

Apache2, dünya çapında en yaygın kullanılan web sunucularından biridir. Web sitelerini barındırmak ve HTTP protokolü üzerinden web tarayıcılarına içerik sunmak için kullanılır. Açık kaynaklı bir yazılımdır ve Apache Software Foundation tarafından geliştirilir ve desteklenir.&#x20;

* Apache2 tamamen ücretsizdir ve açık kaynak lisansı altında dağıtılır.
* Apache2, Unix, Linux, Windows ve macOS dahil olmak üzere birçok işletim sisteminde çalışabilir.
* Apache2, çeşitli işlevsellikleri eklemek veya çıkarmak için modüller (modules) kullanır. Bu, sunucunun performansını ve yeteneklerini ihtiyaçlara göre uyarlamayı sağlar.
* Apache2,  SSL/TLS desteği, erişim kontrolü ve kullanıcı kimlik doğrulaması gibi güvenlik özelliklerini içerir.
* Apache2, yapılandırma dosyaları aracılığıyla çok esnek bir şekilde yapılandırılabilir. Kullanıcılar, ihtiyaçlarına göre çeşitli modülleri etkinleştirip devre dışı bırakabilirler.

Apache2'nin kurulumu genellikle basittir ve çoğu Linux dağıtımında paket yöneticileri aracılığıyla kolayca yapılabilir. Örneğin, Ubuntu'da Apache2'yi kurmak için şu komutlar kullanılır:

```bash
sudo apt update
sudo apt install apache2
```

#### Temel Yapılandırma Dosyaları

1. **apache2.conf:**
   * Ana yapılandırma dosyasıdır ve genel sunucu ayarlarını içerir.
   * `/etc/apache2/apache2.conf` yolunda bulunur.
   * İçeriğinde genel sunucu ayarları, modül tanımları ve global kurallar bulunur.

```apacheconf
ServerRoot "/etc/apache2"
Mutex file:${APACHE_LOCK_DIR} default
PidFile ${APACHE_PID_FILE}
Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}

# LoadModule directives
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

# Virtual hosts
IncludeOptional sites-enabled/*.conf
```



2. **sites-available/ ve sites-enabled/:**

{% hint style="info" %}
Virtual host, tek bir sunucu üzerinde birden fazla web sitesini barındırma imkanı sağlayan bir yapılandırma yöntemidir. Apache HTTP Server gibi web sunucuları, virtual host kullanarak farklı alan adlarına veya IP adreslerine göre çeşitli web sitelerini sunabilirler. Virtual host'lar, aynı sunucuda birden fazla web sitesi barındırmak için yaygın olarak kullanılır.
{% endhint %}

* **sites-available/**: Virtual host yapılandırma dosyalarının bulunduğu dizindir.
* **sites-enabled/**: Aktif olan virtual host yapılandırma dosyalarına sembolik linkler içerir.
* Genellikle `/etc/apache2/` dizini altında bulunurlar.

**Örnek Virtual Host Dosyası (sites-available/example.com.conf):**

```apacheconf
<VirtualHost *:80>
    ServerAdmin webmaster@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/html/example.com/public_html
    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined
</VirtualHost>
```



3. **mods-available/ ve mods-enabled/:**
   * **mods-available/**: Yüklenebilir Apache modüllerinin yapılandırma dosyalarını içerir.
   * **mods-enabled/**: Aktif olan modüllere sembolik linkler içerir.
   * Genellikle `/etc/apache2/` dizini altında bulunurlar.



4. **ports.conf:**

* Apache2'nin hangi portlardan dinleyeceğini belirler.
* `/etc/apache2/ports.conf` yolunda bulunur.

```apacheconf
Listen 80
<IfModule ssl_module>
    Listen 443
</IfModule>
<IfModule mod_gnutls.c>
    Listen 443
</IfModule>
```



#### Yapılandırma Dosyalarının Yönetimi;

```bash
# Virtual Hostları Etkinleştirme ve Devre Dışı Bırakma

# Virtual Hostları Etkinleştirme:
sudo a2ensite example.com.conf
# Bu komut, /etc/apache2/sites-available/example.com.conf dosyasına,
# /etc/apache2/sites-enabled/ dizininde bir sembolik link oluşturur.

# Virtual Hostları Devre Dışı Bırakma:
sudo a2dissite example.com.conf
# Bu komut, /etc/apache2/sites-enabled/example.com.conf sembolik linkini kaldırır.

# Modülleri Etkinleştirme ve Devre Dışı Bırakma

# Modül Etkinleştirme:
sudo a2enmod rewrite
# Bu komut, /etc/apache2/mods-available/rewrite.load dosyasına,
# /etc/apache2/mods-enabled/ dizininde bir sembolik link oluşturur.

# Modül Devre Dışı Bırakma:
sudo a2dismod rewrite
# Bu komut, /etc/apache2/mods-enabled/rewrite.load sembolik linkini kaldırır.

# Yapılandırmayı Test Etme ve Yeniden Başlatma

# Yapılandırmayı Test Etme:
sudo apache2ctl configtest
# Bu komut, Apache2 yapılandırma dosyalarını kontrol eder ve yapılandırmanın geçerli olup olmadığını bildirir.

# Apache2'yi Yeniden Başlatma:
sudo systemctl restart apache2
# Bu komut, Apache2 hizmetini yeniden başlatır ve yapılandırma değişikliklerini uygular.
```



### httpd

Red Hat tabanlı sistemlerde (örneğin, RHEL, CentOS, Fedora) Apache HTTP Server genellikle `httpd` adıyla bilinir ve yönetilir.&#x20;

#### Temel Yapılandırma Dosyaları

Red Hat tabanlı sistemlerde Apache HTTP Server yapılandırma dosyaları genellikle `/etc/httpd/` dizini altında bulunur.



1. **httpd.conf:**

* Ana yapılandırma dosyasıdır ve genel sunucu ayarlarını içerir.
* `/etc/httpd/conf/httpd.conf` yolunda bulunur.
* Bu dosya, Apache'nin temel yapılandırma ayarlarını içerir.

```apacheconf
ServerRoot "/etc/httpd"
Listen 80
Include conf.modules.d/*.conf
User apache
Group apache
ServerAdmin root@localhost
DocumentRoot "/var/www/html"
<Directory "/var/www/html">
    AllowOverride None
    Require all granted
</Directory>
ErrorLog "logs/error_log"
CustomLog "logs/access_log" combined
```



2. **conf.d/:**

* Ek yapılandırma dosyalarını içerir.
* Bu dizindeki dosyalar otomatik olarak `httpd.conf` tarafından dahil edilir.
* Genellikle virtual host 'lar ve diğer modül yapılandırmaları için kullanılır.



3. **modules.d/:**

* Modül dosyalarını içerir.
* Hangi modüllerin yükleneceğini belirten `.conf` dosyaları bu dizinde bulunur.



4. **sites-available/ ve sites-enabled/:** (Bu dizinler varsayılan olarak yoktur, ancak el ile oluşturulabilir)

