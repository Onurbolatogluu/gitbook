# 🤔 Conflict - Rebase

2 farklı branch de, aynı dosyada, aynı satırda yapılan değişiklikler merge edildiğinde, conflict oluşur.

```
git commit -am "file.txt eklendi" # git add yapmadan, aynı komut ile hem staged
hem de commit atabiliriz.
```

### Rebase

![](../.gitbook/assets/yb0g5PRxr.avif)

Git 'de merge ve rebase komutları benzer işlevleri yerine getirmek için kullanılıyor. Her komut da bir branch 'deki değişiklikleri başka bir branch 'e birleştirmek için kullanılır. Ancak iki komut arasında proje tarihçesi oluşturulması ile ilgili ciddi bir farklılık vardır.

* Merge komutu ile A branch 'indeki değişiklikler, B branch 'i ile birleştiğinde, B branch 'inin commit tarihçesinde, merge işleminden kaynaklanan ve merge commit adı verilen otomatik oluşturulmuş bir commit yer alır. Bu commit A ve B branch 'i tarihçelerini birbiriyle ilişkilendirir.
* Rebase kullandığımızda, A branch'inin her bir commit B branch'ine sanki commit işlemi B branch'inde yapılmış gibi davranır. (yeniden yazılır)
* Bu sayede, B branch 'inin, commit tarihçesi, sanki tüm değişiklikler bu branch 'de yapılmış gibi, düz ve kesintisidir.

Birleştirilecek branch içerisinde rebase işlemi yapılmalı.

Master branch 'ine geçip,

```
git rebase test # Test branch'ini master branch ile birleştiriyor.
```

![](<../.gitbook/assets/rebase (1).png>)
