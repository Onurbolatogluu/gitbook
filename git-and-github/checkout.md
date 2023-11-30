# ◀ Checkout

```
git checkout [commit id] 
belirtilen commit id 'ye geri döneriz. (commitler silinmez).

Bunun üzerine geçmişte gidip commit atarsak,yeni bir branch oluşturmamızı
isteyecektir. Önceki commitler silinmeyecektir.
```

### RESET

```
git reset --hard [commit id] 
belirtilen commit ıd 'ye geri döneriz. (öncekiler silinmez)

git reset --soft [commit id]
belirtilen commite geri döneriz. ancak dosyalar silinmez stage area 'da durur.
git restore --staged [file name] ile dosyayı staged alandan çıkarabiliriz.

git reset --mixed [commit id]
Dosyaları belirtilen commit haline döndürür. dosyalar silinmez staged alandan
kendisi çıkarır. Önceki dosyalar repoda tutulur. commitler silinir.
```
