# 🔙 Rollout & Rollback

Deployment yaml dosyalarımızda spec altında, strategy argümanı ile bizler, bu Deployment 'ı güncellediğimiz zaman, rollout işlemlerinin nasıl yapılabileceğini belirleriz. Kullanabileceğimiz 2 tip rollout tipi mevcuttur.

1-Recreate : Deployment 'da bir değişiklik yaptığımızda, öncelikle tüm mevcut pod 'ları siler ve ardından yeni podları oluşturur. Genellikle, uygulamanın, yeni versiyonu ve eski versiyonunun kısa bir süre için bile olsa, bir arada çalışmaması için kullanılır.

2-RollingUpdate : Bu seçenek DEFAULT olarak gelir. Yaml dosyamızda herhangi bir değişiklik yaptığımız zaman, gidip tüm podları silip yenisi oluşturmak yerine, bu işi aşamalı olarak yapar. Bu aşamaların nasıl olacağını belirlediğimiz 2 opsiyonumuz mevcut.

2a : MaxUnavailable : Deployment 'da değişiklik yapıldığında, en fazla burada belirttiğim miktarda pod silinir. Misal 10 pod'lu ortamlarlarda, bir güncelleme yaptığımızı varsayalım. Bir güncelleme yaptığımızda bu güncellemeye başladığı anda, en fazla 2 tanesini siler. sonra yeni podları oluşturur. Ardından 2 eski pod daha siler. Yeni 2 pod daha oluşturur. gibi. Döngü bu şekilde devam eder. Pod sayımız 8 'in altına düşmez.

2b : MaxSurge : Güncelleme sırasında toplam pod sayısının en fazla kaç olacağını belirler. Misal bizim desired state 'imiz 10 pod ama, bu geçiş sırasında 12 pod 'a kadar çıkabilir. Deployment 'ı güncellediğimiz de, kubernetes yeni bir replicaset oluşturacak, yeni tanımda 2 pod ayağa kalkacak, dolayısıyla  eski 10 pod + 2 yeni pod toplamda 12 pod olacak. Sonrasında eski pod'lardan 2 tane silinecek ve ardından yeni replicaset 2 pod daha yaratarak, döngü devam edecek. Toplam pod sayısı 12 'yi geçmeyecek ve 8 'in altına düşmeyecek. Bir değer girmezsek MaxSurge ve Maxunavailable %25 olarak çalışır.



<mark style="color:red;">Deployment objesini düzenlerken "kubectl edit" komutunu da kullanabiliriz.</mark>



\--record  : kullandığımız komutların ardına --record  parametresi eklersek, tüm yapılan işlemleri bir history 'de tutar.

Komutunu kullanarak deployment 'da yaşanan tüm değişiklikleri görebiliriz.

```
kubectl rollout history deployment rolldeployment
```

Liste de bulunan 2. değişikliğin detaylarını göster diyoruz.

```
kubectl rollout history deployment rolldeployment --revision=2
```

Bir önceki değişikliğe dönmek için kullanırız.

```
kubectl rollout undo deployment rolldeployment
```

Deployment 'da yapılan 1. değişikliğe döner. Bir öncekine dönmekle sınırlı değiliz.

```
kubectl rollout undo deployment rolldeployment --to-revision=1
```

Bu sayede deployment üzerinde yaptığımız değişiklikler, kataloglama ve sonrasında geri dönme imkanına kavuşuruz.&#x20;

Bir deployment oluşturur oluşturmaz hangi aşamalardan geçtiğini görmek için,

```
kubectl rollout status rolldeployment -w
```

Deployment ortamında bir güncelleme yapıp, bir sorun gözlemlediğimizde durdurmak istersek;

```
kubectl rollout pause deployment rolldeployment
```

Eğer sorunu tespit edip, giderdikten sonra kaldığı yerden devam etmek istersek,

```
kubectl rollout resume deployment rolldeployment
```
