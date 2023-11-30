# 🫐 Helm Nedir?

Helm bize kubernetes üzerinde paket yükleme imkanı sağlıyor. brew, chocolatey gibi bir paket yöneticisidir. helm bu paket yöneticilerinin kubernetes için kullanılan bir paket yöneticisidir. Biz uygulamalarımızı paketler haline getirmemizi veya hazır paketleri tek bir komut ile kubernetes'e yüklememizi sağlıyor.

```bash
helm version

# komutu ile sistemde yüklü olan helm'in sürümünü öğrenebiliyoruz.
```

Helm 'de bilmemiz gereken ilk kavram; chart 'dır. Chart bizim kubernetes üzerine kurabileceğimiz uygulamanın paketlenmiş haline denir. Helm de chartlar yaratırız veya yaratılmış olan chartları alırız.&#x20;

Herhangi bir Chart 'ı alıp, kubernetes üzerine yüklersek, buna da "Release" denilir. Yani Chart paketin kendi adıdır. Release ise, bunu biz kubernetes cluster'a yüklediğimiz zaman, buna verilen isimdir.

Diğer bilinmesi gereken kavram Repository'dir.  Helm 'i şöyle düşünün, helm bir client-side uygulamadır. Yani bizim kendi cihazlarımıza yükleyip, çalıştırdığımız bir uygulamadır. Bu uygulamaya biz repository eklemek zorundayız. Repository dediğimiz şey ise, helm chart 'larının bir arada tutulduğu çeşitli yerlerdir.

Misal biz bir repository oluşturuyoruz ve bu repository altına çeşitli helm chart 'lar ekleyebiliyoruz. Daha sonra, biz bu helm chart'ları kendi cihazlarımıza kurmak istiyorsak; öncelikle kurmak istediğimiz chart'ın bulunduğu repository'i kendi helm 'mize ekliyoruz. Yani helm'in chartları arayacağı listelere bu repository adresini ekliyoruz. Bu listelere repository ekledikten sonra, bu repository altında ilgili chartları kendi kubernetes cluster'mızda kurabiliyoruz. Kurduğumuz zaman da, bunlara release diyoruz.

Özetle, helm chartlarının bir arada tutulduğu repository'ler mevcut. Bu repository'leri helm'mize ekliyoruz. Onun üzerinden helm releaseler oluşturabiliyoruz.

Geçtiğimiz senelerde, artifacthub adında bir kubernetes projesi yapıldı. Artifacthub'da şu sorunu çözmeye yarıyor; Biz kubernetes üzerine bir çok şey kuruyoruz. Misal, helm chartlar, prometheus paketleri, kubectl pluginleri vb.. Bir çok artifact'i, bizim kubernetes üzerinde kullanmamız gerekebiliyor.  Fakat bu artifactler hep farklı farklı yerlerde duruyor. Misal helm repositoryleri her uygulama için farklı yerlerde, kubectl pluginleri kurmak istediğimizde zaman, zaman bunları çeşitli yerlerde github vs. aramak durumunda kalıyoruduk. Bu paketleri ortak arayabileceğimiz bir çözüm yoktu. ArtifactHub bu sorunu çözmek için, ortaya çıkmış bir projedir. Helm chart'ları dahil, artifacthub üzerinden kubernetes için paketleri arattırabiliyoruz.

Tıpkı dockerhub gibi, artifacthub ise helm chartlar ve diğer kubernetes artifactleri için kullanabiliyoruz.&#x20;

Helm üzerinden paketleri yani chartları arayabileceğimiz komutlar:

```bash
helm search hub wordpress 
# Artifacthub üzerinde ihtiyacımız olan chartı arattırabiliriz.

helm search repo wordpress
# Sistem de mevcut olan, repository listeleri üzerinde ihtiyacımız olan chartı arattırabiliriz.
```

Kuracağımız chart'ın bulunduğu repository adresini helm'mize ekliyoruz, helm böylelikle chart'ı nerede arayacağını anlıyor. Eğer repository mevcut değilse, chart bulunamaz ve chartı kuramayız.

Repository adresini helm'e ekledikten sonra;

```bash
helm install "release adi" "chart ismi" 
# Yukarıdaki komut ile chart'ı kurabiliriz.

helm install wordpress-example bitnami/wordpress

helm status wordpress-example
# Komutu ile, kurulan release hakkında bilgi edinebiliriz.
```

Misal chart kullanarak release oluşturmak istiyoruz.  Hiç bir ek parametre girmezsek chart sistemimize default hali ile kurulur. Yani varsayılan şekli ile oluşturulacak. Fakat bazen biz chart üzerindeki değişkenleri değiştirmek isteyebiliriz.

Misal kullanılacak olan mysql versiyonunu değiştirebiliriz, wordpress bilgilerini kendimiz belirleyebiliriz. vb.. Bunları bu tarz değişkenleri eğer chartı oluşturan bize değiştirme imkanı veriyorsa, bu değişkenleri değiştirebilir ve kendi istediğimiz şekilde düzenleyebiliriz. Bu değişkenleri values.yaml adında bir dosya oluşturarak ve helm'e göstererek release'in bu şekilde kurulmasını sağlayabiliriz.

```bash
helm show values bitnami/wordpress
# Komutu ile, bitnami/wordpress chartı içerisinde, hangi değerleri değiştirebileceğimizi ve kullanabileceğimizi görebiliyoruz.
```

```bash
echo '{mariadb.auth.database: user0db, mariadb.auth.username: user0}' > values.yaml
helm install -f values.yaml bitnami/wordpress --generate-name
```

Yukarıdaki örnekte şunu diyoruz, values.yaml isimli bir dosya oluştur. Mariadb'nin database ismini, mariadb'nin username ismini belirttiğim şekilde values.yaml dosyası içerisine ekle. Böylelikle 2.komut da görüleceği üzere, values.yaml dosyasını helm'e gösterek mariadb.auth.database ve mariadb.auth.username değerleri bizim values.yaml dosyasında belirttiğimiz şekilde baz alınarak release kurulacaktır.

Bizler bu şekilde, varsayılan değerleri kullanmak yerine kullanabileceğimiz değerleri "helm show values "chart ismi" komutu ile gözlemleyip, ardından bu değerler içerisinde değiştirmek istediğimiz kısımlar var ise, values.yaml isimli dosya oluşturup içerisine yazıyoruz.

Yukarıdaki örneği tekrarlamak gerekirse;

mariadb.auth.database  ve mariadb.auth.username  değerlerini kendimize göre özelleştirip, values.yaml dosyası içerisine ekliyoruz.  Ardından helm install -f values.yaml  ile özelleştirdiğimiz değişkenleri helm'e söylüyoruz ve release bu şekilde kuruluyor. -f values.yaml şeklinde belirtmezsek, default hali ile kurulur.

```bash
helm uninstall wordpress-example
# Komutu ile sisteme kurmuş olduğumuz wordpress-example release kaldırılır.
```

Bizler helm üzerinde upgrade işlemleri yapabiliyoruz. Misal biz bitanami'nin bize sağladığı wordpress chartını sisteme release olarak kurduk. Fakat zaman sonra bu release üzerinde bir takım değişiklikler yapma ihtiyacı duyduk.  Değiştirmek istediğimiz değerleri bildirdiğimiz bir dosya yaratıp, daha sonra, (Değiştirebileceğimiz değerleri show values ile görebildiğimizden yukarıda bahsettik)

```bash
helm upgrade -f yenienv.yaml wordpress-example bitnami/wordpress
```

Komutu ile, bizim değiştirdiğimiz/güncellediğimiz değerler ile belirttiğimiz release 'in upgrade edilmesini sağlayabiliyoruz. Böylelikle güncellemek istediğimiz release'i silip, tekrardan kurmak yerine veya release üzerinde kullanılan bir imajın yeni bir versiyonu çıktığı zaman o release'i silmek yerine, yeni istediğimiz değişkenleri/özellikleri bir dosya da belirtip, helm upgrade komutu ile yeni belirlediğimiz değişkenlerin bulunduğu dosyayı göstererek release'in bizim istediğimiz şekilde güncellenmesini sağlayabiliriz.&#x20;

Yapılan upgrade işlemi işimize yaramadıysa veya bazı şeyleri bozmasına neden olduysa, rollback ile bu değişiklikleri geri alabiliriz.

```bash
helm history wordpress-example
# Komutu ile wordpress-example isimli release'in revizyonlarını görebiliriz.

helm rollback wordpress-example 1
# Komutu ile wordpress-example release'in 1 numaralı revizyona döndürülmesini sağlayabiliriz.
```



Chart oluşturmak için;

{% embed url="https://helm.sh/docs/topics/charts/" %}

{% embed url="https://phoenixnap.com/kb/create-helm-chart" %}

Helm Docs;

{% embed url="https://helm.sh/docs/" %}

Artifacthub;

{% embed url="https://artifacthub.io/" %}
