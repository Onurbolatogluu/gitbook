# 🔺 SSL & TLS

#### Symmetric Encryption:

* Symmetric encryption, yalnızca **bir** anahtarın kullanıldığı bir şifreleme yöntemidir. Bu anahtar, hem veriyi şifrelemek (encrypt) hem de şifrelenmiş veriyi çözmek (decrypt) için kullanılır.
* Symmetric encryption'da "public key" ve "private key" kavramları **yoktur**. Bunun yerine, **aynı anahtar** hem şifreleme hem de şifre çözme işlemlerinde kullanılır.
* Anahtarın güvenli bir şekilde paylaşılması ve saklanması büyük önem taşır. Anahtarın ifşa olması durumunda, tüm şifrelenmiş veri tehlikeye girebilir.

#### Asymmetric Encryption:

* Asymmetric encryption, bir çift anahtar kullanır: bir **public key** (herkese açık anahtar) ve bir **private key** (özel anahtar). Bu iki anahtar birbiriyle matematiksel olarak bağlantılıdır, ancak biriyle şifrelenen veri yalnızca diğeriyle çözülebilir.
* Veriyi şifrelemek için public key kullanılır, ve bu şifrelenmiş veri sadece ilgili private key ile çözülebilir. Benzer şekilde, private key ile şifrelenen veri, public key ile çözülebilir.
* Public key herkese dağıtılabilir, ancak private key gizli tutulmalıdır. Bu yapı, özellikle dijital imzalar ve güvenli anahtar değişimi için kullanılır.

SSH, asymmetric encryption kullanan yaygın bir protokoldür. Bir kullanıcı, sunucuya public key'ini gönderir ve Kullanıcı bu veriyi yalnızca kendisinde bulunan private key ile çözebilir.



***

#### SSL (Secure Sockets Layer):

* SSL, 1990'ların ortasında Netscape tarafından web tarayıcıları ve sunucular arasındaki iletişimi güvenli hale getirmek amacıyla geliştirildi.
* SSL, istemci (genellikle web tarayıcısı) ve sunucu arasında şifreli bir bağlantı kurar. Bu, kullanıcıların web sayfalarına güvenli bir şekilde erişmesini sağlar.
* SSL'nin birkaç sürümü vardır (SSL 2.0, SSL 3.0 vb.), ancak bu sürümler güvenlik açıkları nedeniyle artık kullanılmamaktadır.

#### TLS (Transport Layer Security):

* TLS, SSL'nin daha güvenli bir versiyonu olarak geliştirilmiştir. 1999'da, SSL'nin yerini alacak şekilde geliştirilmiştir.
* TLS, SSL ile benzer şekilde çalışır ancak daha güçlü şifreleme algoritmaları ve güvenlik özellikleri sunar. İstemci ve sunucu arasındaki veriyi şifreler ve veri bütünlüğünü sağlar.
* TLS'nin de çeşitli sürümleri vardır (TLS 1.0, TLS 1.1, TLS 1.2, TLS 1.3). Her yeni sürümde güvenlik geliştirmeleri ve performans iyileştirmeleri yapılmıştır.

#### Farkları:

* TLS, SSL'den daha güçlü şifreleme algoritmaları kullanır ve daha güncel güvenlik standartlarına sahiptir.
* SSL, artık güvenli kabul edilmeyen eski bir protokoldür ve büyük güvenlik açıklarına sahiptir. Bu nedenle, modern internet uygulamalarında SSL yerine TLS kullanılır.
* TLS, SSL'nin bir evrimidir. Teknik olarak, TLS 1.0, SSL 3.1 olarak da düşünülebilir.

#### Günümüzde:

* Web tarayıcıları ve sunucular arasında güvenli iletişim sağlamak için genellikle TLS 1.2 veya TLS 1.3 kullanılır.
* HTTPS, web sayfalarının SSL/TLS ile güvence altına alındığını gösterir. Tarayıcıda bir web sayfasının adres çubuğunda kilit simgesi gördüğünüzde, bu sayfanın SSL/TLS ile korunduğunu anlayabilirsiniz.

