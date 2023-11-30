# 🇸🇴 Alias

* Git üzerinde bulunan bazı uzun komutlara kısa-kısa alias tanımlayabiliriz. Böylelikle akılda kalıcı ve kolay bir kullanımı olur.
* Projeye özel bir alias vb. ayarlamalar yapılırsa, proje klasörü içerisinde bulunan .git>config içerisinde tutulur. Bu tanımlamalar sistem genelinde yapılırsa, /etc/ ve Windows için /users/ içerisinde bulunur.

```
git log --all --decorate --oneline --graph # Bu komuta alias tanımlamak için.
git config --global alias.dog "log --all --decorate --oneline --graph" # Komutunu
kullanabiliriz.

git dog # şeklinde çalıştırabiliriz.
```

