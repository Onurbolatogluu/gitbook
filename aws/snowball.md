# 🏐 Snowball

Fiziksel bir veri depolama aleti, alet içerisinde bir mini pc ve birden çok disk bulunmaktadır. Aynı zamanda 256 bit şifreleme desteği sunmaktadır. Biz AWS 'ye veri merkezimizden veri göndermek istediğimizde aws bize bu fiziksel aleti göndermektedir. Bu aleti veri merkezimize bağlıyor ve AWS 'ye göndermek istediğimiz verileri bu diske yüklüyoruz. Daha sonra bu veri şifrelenmektedir. Ve cihaza verileri yükledikten sonra,  Cihazı tekrar AWS 'ye gönderiyoruz. AWS bu veriyi sistemlerine aktarıyor. Şifreleme için kullandığımız anahtar ile verimize erişebiliyoruz.
