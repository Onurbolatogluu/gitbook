# 🟩 MySQL

* MySQL, açık kaynaklı bir veritabanı servisidir.
* MySQL, Büyük veri kümeleriyle verimli bir şekilde çalışabilir.
* Yedekleme, geri yükleme ve veri güvenliği özellikleri sunar.
* Yapısal Sorgulama Dili (SQL) kullanan ilişkisel bir veritabanıdır.

### MySQL Edition'lar ve Türleri,

**MySQL Enterprise Edition:** MySQL Enterprise Edition, kurumsal düzeyde ihtiyaçlar için tasarlanmış ve güvenlik, yüksek kullanılabilirlik, yedekleme, denetim ve uyumluluk gibi özellikleri içeren bir sürümdür. İşletmeler için ekstra araçlar ve destek hizmetleri sunar.

**Ayrıca;**

* **MySQL Standard Edition**
* **MySQL Classic Edition**
* **MySQL Cluster CGE (Carrier Grade Edition)**
* **MySQL Embedded (OEM/ISV)**

**MySQL Community Edition:** MySQL Community Edition, MySQL'in ücretsiz ve açık kaynaklı sürümüdür. Geniş bir kullanıcı topluluğu tarafından desteklenir ve katkıda bulunulur.

#### Örnek SQL Sorguları,

```sql
-- Root kullanıcısının şifresini değiştirmek
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';

-- Veritabanlarını listelemek
SHOW DATABASES;

-- Yeni bir veritabanı oluşturmak
CREATE DATABASE school;

-- Oluşturduğumuz veritabanını kullanmak
USE school;

-- Yeni bir tablo oluşturmak
CREATE TABLE persons (
    Name varchar(255),
    Age int,
    Location varchar(255)
);

-- Tabloları listelemek
SHOW TABLES;

-- Yeni bir kayıt eklemek
INSERT INTO persons values ("John Doe", 45, "New York");

-- Tablodan veri seçmek
SELECT * FROM persons;

-- Yeni bir kullanıcı oluşturmak
CREATE USER 'john'@'localhost' IDENTIFIED BY 'MyNewPass4!';

-- Belirli bir IP'den bağlanacak bir kullanıcı oluşturmak
CREATE USER 'john'@'192.168.1.10' IDENTIFIED BY 'MyNewPass4!';

-- Tüm IP'lerden bağlanabilecek bir kullanıcı oluşturmak
CREATE USER 'john'@'%' IDENTIFIED BY 'MyNewPass4!';

-- Yeni kullanıcı ile giriş yapmak
mysql -u john -pMyNewPass4!

-- Kullanıcıya belirli bir tabloda yetki vermek
GRANT SELECT ON school.persons TO 'john'@'%';

-- Kullanıcıya birden fazla yetki vermek
GRANT SELECT, UPDATE ON school.persons TO 'john'@'%';

-- Kullanıcıya belirli bir veritabanında tüm tablolarda yetki vermek
GRANT SELECT, UPDATE ON school.* TO 'john'@'%';

-- Kullanıcıya tüm veritabanlarında tüm yetkileri vermek
GRANT ALL PRIVILEGES ON *.* TO 'john'@'%';

-- Kullanıcının sahip olduğu yetkileri görmek
SHOW GRANTS FOR 'john'@'localhost';

```



