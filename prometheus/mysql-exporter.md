# 👨‍🚒 mysql exporter

MySQL Exporter, MySQL sunucusunda çalışan ve sunucu üzerinde birçok önemli performans ölçütünü yakalayan bir istemci hizmetidir. Bu ölçütler arasında CPU kullanımı, bellek tüketimi, sorgu sayısı, sorgu süresi, ağ trafiği ve daha fazlası yer alabilir. MySQL Exporter, bu metrikleri ölçer ve Prometheus tarafından toplanabilir hale getirilir. Sonuç olarak, Prometheus bu metrikleri toplar, depolar ve analiz eder, böylece sistem yöneticileri ve geliştiriciler performans sorunlarını tespit edebilir, kapasite planlaması yapabilir ve genel veritabanı sağlığını izleyebilir.



### MySQL Exporter Kullanıcısı Oluşturma

1.  MySQL veritabanına root kullanıcısıyla bağlanın:

    ```
    mysql -u root -p
    ```
2.  MySQL komut satırına giriş yaptıktan sonra, aşağıdaki SQL komutlarını sırasıyla çalıştırın:

    ```sql
    CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'hkyDh42Kb';
    GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;
    ```

    Yukarıdaki komutlarla aşağıdaki işlemler gerçekleştirilmektedir:

    * `CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'hkyDh42Kb';` komutuyla `exporter` adında bir kullanıcı oluşturulur. Bu kullanıcının erişim sağlayabileceği IP adresi `localhost` olarak belirlenir.
    * `GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';` komutuyla `exporter` kullanıcısına `PROCESS`, `REPLICATION CLIENT` ve `SELECT` yetkileri verilir. Bu yetkiler, MySQL Exporter'ın gerekli veritabanı metriklerini alabilmesi için gereklidir.
    * `FLUSH PRIVILEGES;` komutuyla MySQL yetki tabloları yenilenir ve yapılan değişikliklerin etkili olması sağlanır.
    * `EXIT;` komutuyla MySQL komut satırından çıkış yapılır.

    Not: Eğer Master-Slave veritabanı mimarisine sahipseniz, kullanıcıyı sadece master sunucuda oluşturmanız gerekmektedir.

<figure><img src="../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

Bu dökümanı izleyerek MySQL Exporter için gerekli kullanıcıyı oluşturabilirsiniz. Kullanıcı, gerekli yetkilere sahip olacak ve MySQL Exporter'ın veritabanı metriklerini alabilmesi için hazır hale gelecektir.



### MySQL Exporter'ı Ubuntu 20.04'e Kurma Kılavuzu

1.  Prometheus kullanıcısı ve grubunu oluşturun:

    ```bash
    sudo groupadd --system prometheus
    sudo useradd -s /sbin/nologin --system -g prometheus prometheus
    ```
2.  En son MySQL Exporter sürümünü indirin:

    ```bash
    curl -s https://api.github.com/repos/prometheus/mysqld_exporter/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -
    ```
3.  İndirilen dosyayı çıkartın:

    ```bash
    tar xvf mysqld_exporter*.tar.gz
    ```
4.  `mysqld_exporter` dosyasını `/usr/local/bin/` dizinine taşıyın:

    ```bash
    sudo mv mysqld_exporter-*.linux-amd64/mysqld_exporter /usr/local/bin/
    ```
5.  Dosyanın çalıştırılabilir olduğundan emin olun:

    ```bash
    sudo chmod +x /usr/local/bin/mysqld_exporter
    ```
6.  MySQL Exporter sürümünü kontrol edin:

    ```bash
    mysqld_exporter --version
    ```
7.  MySQL Exporter için yapılandırma dosyasını oluşturun:

    ```bash
    sudo vim /etc/.mysqld_exporter.cnf
    ```

    Not: Vim yerine tercih ettiğiniz metin düzenleyiciyi kullanabilirsiniz.
8.  Aşağıdaki içeriği `mysqld_exporter.cnf` dosyasına yapıştırın ve kaydedin:

    ```bash
    [client]
    user=exporter
    password=hkyDh42Kb
    ```

    Not: Kullanıcı adı ve şifre, gerektiğinde güncelleyebileceğiniz örnek değerlerdir.
9.  Dosyanın sahipliğini düzenleyin:

    ```bash
    sudo chown root:prometheus /etc/.mysqld_exporter.cnf
    ```
10. MySQL Exporter için systemd servis dosyası oluşturun:

    ```bash
    sudo vi /etc/systemd/system/mysql_exporter.service
    ```

    Not: Vim yerine tercih ettiğiniz metin düzenleyiciyi kullanabilirsiniz.
11. Aşağıdaki içeriği `mysql_exporter.service` dosyasına yapıştırın ve kaydedin:

    ```bash
    [Unit]
    Description=Prometheus MySQL Exporter
    After=network.target
    User=prometheus
    Group=prometheus

    [Service]
    Type=simple
    Restart=always
    ExecStart=/usr/local/bin/mysqld_exporter \
    --config.my-cnf /etc/.mysqld_exporter.cnf \
    --collect.global_status \
    --collect.info_schema.innodb_metrics \
    --collect.auto_increment.columns \
    --collect.info_schema.processlist \
    --collect.binlog_size \
    --collect.info_schema.tablestats \
    --collect.global_variables \
    --collect.info_schema.query_response_time \
    --collect.info_schema.userstats \
    --collect.info_schema.tables \
    --collect.perf_schema.tablelocks \
    --collect.perf_schema.file_events \
    --collect.perf_schema.eventswaits \
    --collect.perf_schema.indexiowaits \
    --collect.perf_schema.tableiowaits \
    --collect.slave_status \
    --web.listen-address=0.0.0.0:9104

    [Install]
    WantedBy=multi-user.targetb
    ```

    Not: `User` ve `Group` değerlerini gerektiğinde güncelleyebilirsiniz.
12. systemd'ye servisi tanıtın:

    ```bash
    sudo systemctl daemon-reload
    ```
13. MySQL Exporter'ı başlatın:

    ```bash
    sudo systemctl enable mysql_exporter
    sudo systemctl start mysql_exporter
    ```
14. MySQL Exporter'ın çalıştığını kontrol edin:

    ```bash
    sudo systemctl status mysql_exporter
    ```
15. **Finish** :tada:<br>

    <figure><img src="../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.devopsschool.com/blog/install-and-configure-prometheus-mysql-exporter/" %}

#### Prometheus'a MySQL Exporter'ı eklemek için aşağıdaki adımları izleyebilirsiniz:

1.  Prometheus'un yapılandırma dosyasını açın:

    ```bash
    sudo nano /etc/prometheus/prometheus.yml
    ```
2.  Dosyada `scrape_configs` bölümüne gidin ve aşağıdaki örnekte olduğu gibi yeni bir job tanımı ekleyin:

    ```yaml
    scrape_configs:
      - job_name: 'mysql_exporter'
        static_configs:
          - targets: ['10.90.0.147:9104']
    ```

    Burada `job_name` MySQL Exporter için tanımlanan bir ad, `targets` ise MySQL Exporter'ın çalıştığı IP adresi ve port numarasını belirtir. Eğer MySQL Exporter farklı bir IP ve portta çalışıyorsa, ilgili değerleri güncellemeniz gerekmektedir.
3. Yapılandırmayı kaydedip dosyayı kapatın.
4.  Prometheus servisini yeniden başlatın:

    ```bash
    sudo systemctl restart prometheus
    ```
5. Artık Prometheus, belirttiğiniz MySQL Exporter hedefini scrape edecek ve metrikleri toplayacaktır. Metrikleri Prometheus Web arayüzünden (`http://<Prometheus-IP-Adresi>:9090`) veya Prometheus üzerinden sorgulayarak kullanabilirsiniz.

<figure><img src="../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>

Prometheus'a MySQL Exporter'ı ekledikten sonra, MySQL veritabanına ait metrikleri scrape ederek görselleştirebilir, alarm ve kaynak izleme kuralları oluşturabilirsiniz.



<figure><img src="../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>









