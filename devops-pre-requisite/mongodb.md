# 🟪 MongoDB

MongoDB, açık kaynaklı ve NoSQL türünde bir veritabanıdır. Geleneksel SQL tabanlı veritabanlarından farklı olarak, verileri tablo ve satır formatında değil, JSON document olarak saklar. Bu sayede, veri yapıları zamanla değiştirilebilir ve genişletilebilir.

#### MongoDB'nin Sürüm ve Kullanım Alanları:

* **Community:** MongoDB'nin topluluk sürümüdür. Ücretsizdir ve geliştiriciler tarafından kullanılır. Bu sürüm, temel MongoDB özelliklerini sunar ve daha çok öğrenme, geliştirme ve prototip oluşturma süreçlerinde tercih edilir.
* **Enterprise:** Kurumsal düzeyde kullanım için optimize edilmiş bir sürümdür. Bu sürümde öncelikli güvenlik, performans ve yönetim özellikleri sunulur. Kurumlar için profesyonel destek sağlar.

#### Dağıtım Seçenekleri:

* **Cloud:** MongoDB'nin bulut tabanlı sürümleri, paylaşımlı ya da izole kaynaklarla çalışabilir. İstendiğinde kolayca ölçeklenebilir ve güvenlik açısından gelişmiş özellikler sunar.
* **Server:** MongoDB'nin sunucu tabanlı sürümü, hem topluluk (community) hem de kurumsal (enterprise) sürümlerini içerir. Sunucu tabanlı dağıtımlar, daha geleneksel bir kurulum yöntemi sunar ve kurum içi veritabanı çözümleri için idealdir.

```mongodb
# MongoDB'ye bağlanmak için
mongo

# Veritabanlarını listelemek
show databases

# Bir veritabanı seçmek veya oluşturmak
use myDatabase

# Seçili veritabanındaki koleksiyonları (tabloları) listelemek
show collections

# Bir koleksiyon oluşturmak
db.createCollection("myCollection")

# Bir koleksiyona belge (document) eklemek
db.myCollection.insertOne({ name: "John Doe", age: 30, location: "New York" })

# Bir koleksiyona birden fazla belge eklemek
db.myCollection.insertMany([
  { name: "Jane Doe", age: 25, location: "Los Angeles" },
  { name: "Mark Smith", age: 40, location: "Chicago" }
])

# Tüm belgeleri listelemek (tüm verileri almak)
db.myCollection.find()

# Belirli bir kriterle belge aramak (örneğin, yaşı 30 olan kişileri bulmak)
db.myCollection.find({ age: 30 })

# Belirli alanları göstererek belge aramak (örneğin, sadece isim ve yaşı göster)
db.myCollection.find({ age: 30 }, { name: 1, age: 1, _id: 0 })

# Belgeyi güncellemek (örneğin, "John Doe" yaşını 35 olarak güncellemek)
db.myCollection.updateOne({ name: "John Doe" }, { $set: { age: 35 } })

# Belgeyi silmek (örneğin, "Jane Doe" isimli kişiyi silmek)
db.myCollection.deleteOne({ name: "Jane Doe" })

# Koleksiyonu silmek
db.myCollection.drop()

# Veritabanını silmek
db.dropDatabase()

```

