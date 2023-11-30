# 👉 Other

push : projemizi uzak repoya göndermek için kullanırız.\
pull   : projemizi uzak repodan çekmek için kullanırız.

```
git push [repo url] [branch ismi]

git remote add [origin (takma ad)] [repo url]

git push origin master => Origin takma adına sahip repomuza değişikleri gönderiyoruzz.

git fetch : Remote repodan değişiklileri çek.
```

Pull Request : Fark ettiğimiz bir projenin üzerinde çalışıp, Proje 'de yaptığımız değişiklerin, projenin sahibine pull request olarak gönderebiliriz. Pull request aslında, farklı bir kullacının reposunda bulunan bir projeye geliştirmesinde yardımcı olmaktadır.



Patch : Hata düzeltmeleri, bugfix ile alakalıdır.\
Minor : Küçük,yeni özellikler. Eski kullanıcılar etkilenmez.\
Major : Büyük yeni özelliklerin bulunduğu sürüm.

```
git tag -a v1.0.0 "first tag1" > bulunduğumuz commit'i tagler(-a ile yorum ekledik).

git tag v1.2.3 [commit id] > belirttiğimiz commit id ye sahip commit'e tag ekledik.

```

* Git 'e bir commit gönderirken, tag vermek istersek,

```
git push origin master [tag ismi]
```

### Amend

Son kaydedilen işlemdeki mesajı son commit mesajını değiştirir. Yeni bir kayıt oluşturmak yerine aşamalı değişiklikler bir önceki işleme eklenecektir. Varsayılan düzenleyici açılarak bir önceki commit mesajının değiştirmemizi ister.

```
git commit --amend -m "test" 
son commit mesajını test olarak değiştirdik.

git push origin master --force
commit mesajının güncel halini repoya gönderdik.
```
