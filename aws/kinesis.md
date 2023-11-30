# 📦 Kinesis

Veri üretici ve veri tüketici arasında mesajları yayınlamak için kanal görevi gören akışkan gerçek zamanlı veri toplama ve dağıtma sistemidir.&#x20;

![](<../.gitbook/assets/architecture (2).png>)

Kinesis sistem ve ya web günlüklerini sosyal ağ verileri finansal ticaret bilgileri, cografi veriler, mobil uygulama verileri ve ya bağlı IOT cihazlarından telemetri verileri gibi verileri gerçek zamanlı olarak toplayarak belirli bir süre üzerinde barındırmaya ve daha sonra bunu veri tüketicilere ulaştırmaya yarıyor.

#### Kinesis Özellikleri,

#### Amazon Kinesis Data Streams,

* Veriler shard adı verilen kümelerde tutuluyor.
* Shard'da tutulan benzersiz verilere record (kayıt) denir.
* Her kayıt sequence number yani shard 'da giriş sırasını belirleyen değer, partition key, yani basitçe hangi shard 'da bulunduğunu belirten değer ve data blob yani gerçek içerikleri oluşturur.
* Record maksimum 1mb boyutunda olabilir.
* 1 shard 1 saniye de 1mb 'a ve ya 1000 put işlemine kadar depolama ve saniyede 2mb hızında okunma kapasitesine sahiptir.
* Shard'lar otomatik olarak büyüyüp küçülmez ve kaç shard kullanılacağını başlangıçta bizim belirtmemiz gerekir.&#x20;
* Sonuçları da veri depolama ambarımıza yazar.

Belirli bir anahtar kullanarak kayıtların yönlendirilmesini, kayıtların sıralanmasını, birden fazla istemcinin iletileri aynı anda okumasını, geçmiş 7 güne kadar mesajların tekrarlanmasını sağlıyor. Kinesis de mesajlar birden fazla tüketici (istemci) tarafından okunabilir.

#### Kinesis Firehose;

* Akışlı verileri düzenleyip, s3 yada redshift gibi hedeflere kaydeder.
* Otomatik genişleyebilir.

#### _Kinesis_ _Analytics_

* _Firehose ve streams servislerine bundle edilmiştir._
* _Akışlı veriler üstünde standart SQL sorguları çalıştırma imkanı verir._

#### _Kinesis Video Streams_

* _Video ve ses akışlarını AWS bulutuna aktararak, daha sonra makine öğrenimi uygulamaları gibi uygulamalar tarafından işlenmesine imkan tanır._
