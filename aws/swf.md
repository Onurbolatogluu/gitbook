---
description: Simple Workflow Service
---

# 🎇 SWF

Yazılım geliştirme işleriyle ilgilenen kişiler için üretilmiş bir servistir. SWF yapılan bir işin düzgün bir şekilde yapılıp, yapılmadığını kontrol etmektedir. Temel de 3 bölümden oluşur.

Workflow Starter : İş akışı yürütmelerini başlatabilecek herhangi bir uygulamadır. Örn, sipariş verdiğimiz X bir web sitesi.

Workflow Decider : Karar verici, tüm bu iş akışının kordinasyonunu sağlayıp, işlerin olumlu,olumsuz bilgisine göre işleri bir sonraki adımı ya da farklı adımı tetiklemeye yarıyor.

Workflow worker : Esas işi yapıp, belirlediğimiz işi yapıp sonucu veren programlar. Örnek siparişe göre ürünün stoklarda olup, olmadığını kontrol eden, geriye olumlu ya da olumsuz sonuç dönen sipariş onayı backend worker programına gönderiliyor. Yönetilen iş akışı hizmeti sunuyor.

#### Süreç,

Müşteri => Sipariş => Tahsilat => Paketleme => Gönderim => Kayıt => Sipariş onayı.
