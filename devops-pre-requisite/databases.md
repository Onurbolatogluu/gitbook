# 🟨 Databases

* Veri tabanları, herhangi bir sistem tasarımının ana bileşenlerinden biridir. Birçok uygulama ve sistem, verilerin saklanması, işlenmesi ve geri getirilmesi için veri tabanlarına güvenir. Bu nedenle, veri tabanlarının nasıl çalıştığını anlamak, etkili ve verimli sistemler tasarlamak için kritik önem taşır.
* Sistem sorumluları (ya da DevOps mühendisleri), veri tabanlarının kurulumu ve yönetiminden sorumludur. Bu, veri tabanlarının uygun şekilde çalışmasını sağlamak, performanslarını optimize etmek ve veri güvenliğini korumak anlamına gelir.
* Uygulamalar, veri tabanlarına veri yazar (kaydetme) ve veri okur (sorgulama). Veri tabanları, uygulamaların çalışması için hayati bir rol oynar. Örneğin, bir e-ticaret uygulaması, ürün bilgilerini, müşteri siparişlerini ve kullanıcı verilerini saklamak için veri tabanını kullanır.
* Uygulamaların ve kullanıcıların veri tabanlarına nasıl bağlanacağını anlamak önemlidir. Bu, veri tabanı istemcileri veya programlama dilleri aracılığıyla veri tabanlarına erişmeyi içerir.

### SQL (Structured Query Language)

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* SQL veritabanları, ilişkisel model üzerinde çalışır. Veriler, satırlar ve sütunlar şeklinde tablolarda organize edilir. Her tablo, belirli bir veri türüne sahip olabilir ve veriler, bu önceden tanımlanmış yapıya uymalıdır.
* Veriler üzerinde işlem yapmak için SQL dilini kullanırız. Örneğin, belirli yaşın üzerindeki kişileri listelemek için `SELECT * FROM persons WHERE AGE > 10` gibi bir sorgu yazılır.
* Genellikle dikey olarak ölçeklendirilir, yani bir sunucunun donanım kapasitesini artırarak performans artırılır.
* Örnek Veritabanları: MySQL, PostgreSQL, Microsoft SQL Server, Oracle.

### NoSQL (Not Only SQL)

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* NoSQL veritabanları, yapılandırılmamış veya yarı yapılandırılmış veri türlerini saklayabilir. JSON, BSON veya XML gibi formatlarda veriler saklanır. Veri yapısı sabit değildir, bu da veri şeması üzerinde daha fazla esneklik sağlar.
* Veriler, tablolardan ziyade belgeler olarak saklanır ve bu belgeler, çeşitli anahtar-değer çiftlerinden oluşur. Örneğin, MongoDB'de veriler JSON benzeri belgeler olarak saklanır.
* Genellikle yatay olarak ölçeklendirilir. Birden fazla sunucuya yayılarak veri depolama ve işleme kapasitesi artırılabilir.
* Big Data, gerçek zamanlı analiz ve çeşitli veri türleriyle çalışma gibi durumlarda tercih edilir. NoSQL veritabanları, yüksek trafik ve esneklik gerektiren uygulamalar için idealdir.
* Örnek Veritabanları: MongoDB, Cassandra, DynamoDB, Couchbase.

### Farklılıklar ve İhtiyaçlar:

* SQL veritabanları, katı yapıya sahip, verilerin bütünlüğünü ve tutarlılığını sağlamak için daha uygundur. Bankacılık sistemleri veya muhasebe uygulamaları gibi veri bütünlüğünün kritik olduğu durumlarda SQL tercih edilir.
* NoSQL veritabanları, hızlı veri erişimi, esnek veri yapıları ve ölçeklenebilirlik sağlamak için daha uygundur. Sosyal medya uygulamaları, büyük veri işleme veya dinamik veri yapılarına sahip projeler için idealdir.
* Yatay ölçeklenebilirlik gerektiren uygulamalarda, NoSQL daha verimli olabilir. Ancak, dikey ölçeklenebilirlik yeterli olduğunda ve veri bütünlüğü kritik olduğunda SQL tercih edilir.





