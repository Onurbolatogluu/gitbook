# 🥢 Diff

Git 'de iki versiyon arasındaki farkları görmek için kullanılır. Karşılaştırılan dosyalar A/B diff komutu iki dosyayı birbiriyle karşılaştırır. A dosyası ve B dosyası, Bu A ve B dosyası genelde aynı dosyanın farklı versiyonlarıdır. Çok sık olmasa da, diff işlemi ile tamamen farklı olan dosyaları da karşılaştırabiliriz.  Hangi dosyanın karşılaştırıldığını açıkca belirtmek için için Diff komutunun çıktısı her zaman hangi dosyanın da B olduğunu belirterek bakar.

```
git diff 1234 5678 dosya1.txt
# Dosya1.txt 'nin 1234 ve 5678 hashlerindeki commitlerin farkını alıyoruz.
```

Dosya içerisinde hangi kısmın A, hangi kısmın B dosyasına ait olduğunu belirtmek için kullanılan - ve  + işaretleri vardır. Diff komutu ile sadece  iki dosya arasındaki (versiyon) farkların olduğu satırları gösterir. Dosyanın tamamını değişmeyen satırları göstermez. @@ ile başlayan semboller A ve B versiyonları arasındaki  farklı satırların, hangi satırdan başlayıp, kaç satır olduğunun bilgisini verir.

@@-1.4 +1.2@@ bize, - simgesi ile tanımlanan A dosyasından 1.satırdan başlayarak 4 satır. + simgesi ile tanımlanan B dosyasından 1.satırdan başlayarak  2 satır birbirinden farklı olduğunu söyler.

Değişen her satırın başına - veya + simgeler yer alır. Bu simgeler A ve B versiyonlarının içeriğinin ne olduğunu anlamamıza yardımcı olur.&#x20;

```
git diff --staged
staged bölgedeki dosyaların farklarını inceleriz.
```

```
git diff master tester
iki farklı branch'i inceleyebiliriz.
```

Hangi commitleri, hangi versiyonları karşılaştıracaksak, git log üzerinden ilgili commit bulup, hash Id 'sini öğrenmemiz gerekir.

```
git diff 1234 5678
1234 versiyonunun 5678 versiyonundan farkı
```

### Restore

```
git restore --staged a.txt 
a.txt dosyasını staged alandan çıkarır.
```

```
git restore a.txt
Son commit üzerine dosyada yapılan değişikleri geri alır.
```
