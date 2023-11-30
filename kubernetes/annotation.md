# 🔎 Annotation

* Objelere metadata eklemek için kullanabildiğimiz opsiyon olan label'lerin yanı sıra kullanabileceğimiz bir parametredir.&#x20;
* Label 'da olduğu gibi key-value eşlenikleri şeklinde metadata ekleyebildiğimiz, bir opsiyondur.
* Label'ler gruplama yaparken, ve objeler arası bağ kurmak istediğimiz durumlarda, kullanırız. Bu ilişki kurma durumundan kaynaklı label'ler hassas bilgi sınıfına girer. Yani label eklememiz, veya çıkarmamız kubernetes içerisinde bir şeyleri tetikleyebilir ve etkileyebilir. Dolayısıyla, her türlü bilgiyi label olarak metadata 'ya eklemek doğru değildir. Fakat bazı ek bilgileri, objelere eklememiz gerekebilir.

Misal, oluşturduğumuz objelere, o objenin oluşturma tarihini, kimin oluşturduğu bilgisini eklemek istersek, Bunu label ile yapmak doğru olmayacaktır. Çünkü bu bilgiyi, selector kullanarak seçme isteğimiz yok. Ayrıca bu bilgiyi herhangi bir ilişki kurmak içinde kullanmayacağız. Dolayısıyla, bunu label 'a eklemek yerine, annotation 'a eklemek daha doğru olacaktır.

Bunun dışında, annotation'lar kubernetes 'in ana komponenti olmayan fakat kubernetes ile bağlantısı olan yazılımlar tarafından, ihtiyaç duyulacak bilgilerin, yazıldığı yerdir.

Misal, firmamızın destek ekibinin çağrı kabul yazılımı mevcut,  bu çağrı kabul yazılımının kubernetes'den bilgi çekecek hale getirdik ve şöyle bir sistem kurmak istiyoruz. Bir obje de sorun oluştuğunda, bu çağrı yazılımı, bunu tespit etsin ve objenin destek sorumlusuna mail göndersin. Bu durumda o mail adresi bilgisini objenin metadata kısmına annotation olarak ekleyebiliriz ve destek yazılımı bu bilgiyi çekebilir. Bu sayede, bu insanlara mail gönderebilir.

Annotation bu ve benzeri durumlar için kullanılır. Oluşturma kuralları label'lar ile hemen hemen aynıdır.

```
example.com/notification-email:admin@example.com
  prefix        Key                 Value
  
```

<mark style="color:red;">Objeye onu tanımlayan, fakat label olarak eklememizin sakıncalı olacağı bilgileri Annotation olarak ekleyebiliriz.</mark>

* Prefix zorunlu değildir.
* Key alanı 63 karakter ve daha altı olmalıdır.
* Alfanumaratik bir karakter ile başlamalı veya bitmelidir.
* İçerisinde ( . - \_ ) gibi karakterler içerebilir.
* Value alanında, bu kurallar geçerli değildir. Alfanumaratik olmayan karakterlerde bulunabilir.

```
# kubectl kullanarak annotation oluşturmak ve silmek
kubectl annotate pods pod1 foo:bar

# silmek
kubectl annotate pods pod1 foo-
```



YML;

```
apiVersion: v1
kind: Pod
metadata:
  name: annotationpod
  annotations:
    owner: "mycat"
    notification-email: "admin@example.com"
    releasedate: "01.01.2021"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  containers:
  - name: annotationcontainer
    image: nginx
    ports:
    - containerPort: 80
```
