# 🔦 Stash

Commit işlemi ile dosyalarda yapılan değişiklikler kalıcı olarak repo da kayıt altına alınır. Ancak günlük çalışmamızda bazen tam olarak bitmeyen değişiklikleri de kayıt altına almak isteyebiliriz. Örneğin bir değişiklik üzerine çalışırken, başka bir konu ile ilgili kritik bir sorun bildirildiğinde yapmakta olduğumuz işi yarım bırakıp, yeni soruna odaklanmak zorunda kalabiliriz. Bu gibi durumlarda, yeni bir sorun ile ilgilenmeye başlamak için önceki değişiklikleri kaybetmeden yeni ve temiz branch oluşturmalıyız.  Yarım kalan değişikliklerin kayıt altına almak için, git stash komutunu kullanabiliriz. Git stash ile üzerinde çalıştığımız, ancak commit etmediğimiz değişikliklerin, geçici olarak git tarafından kayıt altına alınmasını ve aktif branch 'imizin, herhangi bir değişikliğin olmadığı temiz bir duruma getirilmesini sağlayabiliriz. Git stash komutu çalıştırıldıktan sonra, tekrar git status çalıştırırsak,  önceki bölümden commit edilmemiş dosya olarak gözükmek. Çünkü master branch 'i git stash sonrası temiz duruma geldi. Git stash list ile aktif branch 'de geçici olarak kayıt altına aldığımız değişikliklerin listelenmesini sağlayabiliriz. Stash 'de yer alan değişikliği geri yüklemek için iki seçeneğimiz var.&#x20;

```
git stash pop
komutu ile yukarıdaki (listenin en üstünde) yer alan değişiklik yüklenecek
ve bu değişiklik listeden silinecek.

git stash apply
komutu ile istediğiniz değişiklikleri geri yükleyebiliriz. Ancak bu işlem
sonrasında yüklediğimiz değişiklik listeden silinmeyecek. Listeden silmek için
git stash drop komutunu kullanabiliriz.
```

Üzerinde çalıştığımız, aktif branch 'imizi temiz bir duruma getirmek için kullanabiliriz. Farklı bir branch 'i aktif hale getirmeden önce kullanabiliriz. Remote repo 'da değişiklilerimizi lokale indirmeden kullanabiliriz. Branch merge etmeden kullanabiliriz.

```
git stash > çalışmanın yapıldığı dosyayı çalışma alanına sakla.
git stash apply [stash id] > geri dön
git stash save "yarım kalan" > stash 'a isim verebiliriz.
```

### Cherry

Bir branch'deki commit 'i (değişikliği) farklı bir branch 'e kopyalamaya denir.

```
git cherry-pick [commit id] 
Commit 'i kopyalamak istediğimiz branch 'e geçip, hangi commit 'i kopyalamak
istiyorsak commit id belirtmemiz gerekiyor. Kopyalandıktan sonra farklı bir
commit id 'ye sahip olarak branch 'imize kopyalanır.
```

{% embed url="https://www.youtube.com/watch?ab_channel=KadirKas%C4%B1m&v=dt7ALS6NksI" %}
