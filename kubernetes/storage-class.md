# 🌜 Storage Class

![](<../.gitbook/assets/Storageclass blog (1)-1.webp>)

Manuel olarak persistent volume ve bunu talep edecek podlarımıza bağlamakta kullandığımız persistent volume claim konularını inceledik. Fakat tahmin edeceğiniz üzere, persistent volume'lerin statik olarak yaratılması gerekliliği bizleri oldukça zorlayacak bir durumdur.&#x20;

Örnek,

10 ayrı ekip tarafından kullanılan büyük bir kubernetes cluster'mız olduğunu düşünelim. Onlarca farklı worker node, yüzlerce pod oluşturuluyor. Onlarca değişik uygulama, çeşitli ekipler tarafından yönetiliyor, siliniyor, güncelleniyor.&#x20;

Bizimöm böyle bir ortamda persistent volume ihtiyacımız olduğunda, her seferinde, önce storage cihazına gidip, paylaşım oluşturmamız, ardından onun karşılığı olan PV objesini oluşturmamız zahmetli olacaktır.

Bu zahmeti ortadan kaldırmak için Kubernetes biz PVC oluşturduğumuz anda depolama ünitesi ile haberleşerek, gerekli ayarları yaparak bize bir volume ve PV objesi oluşturan bir obje sunuyor. Bunun adı da "Storage Class"dır.

Storage Class yöneticilerin, sundukları depolama "sınıflarını" tanımlamaları için bir yol sağlar. Farklı sınıflar, hizmet kalitesi düzeylerine veya yedekleme ilkelerine göre, cluster yöneticileri tarafından belirlenen isteğe bağlı ilkelere eşlenebilir. Kubernetes sınıflarının neyi temsil ettiği konusunda bilgi sahibi değildir. Bu kavram bazen de diğer depolama sistemlerinde "profiller" olarak adlandırılır.

Yani uzun lafın kısası, PV objesini manuel oluşturmak yerine, PVC objesine göre dinamik olarak oluşturma imkanı getirir. Bu özellikle Cloud Servis Sağlayıcılarının dinamik ortamlarında, her şeyi otomatize edebilirken, Sırf volume oluşturma işini manuel halletme saçmalığını ortadan kaldırır.

Storage Class objelerini bağlantımız olan depolama üniteleri üzerinde, ne çeşit bir volume yaratabileceğimizi belirleyen template'ler olarak düşünebiliriz.

Misal, üzerinde hem hızlı ssd, hem de yavaş mekanik disk olan storage ünitemiz olduğunu düşünelim. Bu durumda biz yavaş disklerin üstünde de otomatik volume yaratan, "yavas" isimli bir storage class ve hızlı diskler üstünde çalışan volume yaratan "hizli" isimli, storage class'lar oluşturabiliriz.

Developer ekipleri, uygulamalarında kullanmak için, volume talep etmek yerine PVC 'de uygulamalarının ihtiyaçlarına göre bu Storage Class'lardan birini seçerler ve istedikleri boyutta bir volume Storage Class objesi tarafından otomatik olarak oluşturulup, PVC'ye bind edilir.

![](<../.gitbook/assets/Ekran görüntüsü 2022-06-15 140448.png>)

```bash
kubectl get sc 

# Sistemde bulunan storage class'ları listeler.
```

Yukarıdaki StorageClass özelliklerini inceleyelim,

Provisioner => Bizlere hangi tipte cihaz üstünde, volume oluşturmak istediğimizi belirtiyor.

Reclaim Policy => Bu disk kullanıldıktan sonra, işimiz bittikten sonra ne olacak? Hatırlarsanız bu ayarı PV oluştururken giriyorduk. Şimdi Storage Class tanımlarında giriyoruz. Ek olarak burada "recycle" özelliğini kullanamıyoruz. "retain" ve "delete" seçebiliyoruz.

volumebindingMode => Immediate :  Bir PVC oluşturudulduğu anda, volume'un oluşturulup, hemen PVC'ye bind edilmesi gerektiğini söyler.

volumebindingMode => WaitForfirstConsumer :  Pod oluşturulup, atanana kadar bu volume'un oluşturulmamasını söyler.

Bu 2 seçenek, altyapıda kullanılan storage özelliklerine göre değişiyor.

{% code title="sc.yaml" %}
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standarddisk
parameters:
  cachingmode: ReadOnly
  kind: Managed
  storageaccounttype: StandardSSD_LRS
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```
{% endcode %}

Yukarıdaki örnekten yola çıkacak olursak, "standarddisk" isimli bir storageclass oluşturuyoruz. Geri kalan ayarlar kullandığımız depolama ünitesine göre değişiyor.&#x20;

AzureDisk üzerinde storage class oluşturmak istediğimiz için, provisioner olarak bunu seçiyoruz. Geri kalan ayarları AzureDisk oluşturacaksak AzureDisk dökümanından, farklı bir storage ünitesi için storage class oluşturacaksak, bunu ilgili cihazın/ürünün dökümanlarından bulup, burayı dökümana göre düzenleyebiliriz.&#x20;

Artık volume oluşturmak için PV oluşturmamıza gerek yok. PVC oluşturduğumuzda, ilgili diski hangi storage class'dan istiyorsak, otomatik/dinamik olarak bize sunuyor.

{% code title="pvc.yaml" %}
```yaml
Version: v1
kind: PersistentVolumeClaim
metadata:
  name: mysqlclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: "standarddisk"
```
{% endcode %}

Storage Class için PVC oluştururken, artık "selector" kullanarak, hangi PV olacağını ve kullanmak istediğimiz PV objesini seçmiyoruz. Bunun yerine yukarıda örnekte gördüğünüz üzere,  StorageClassName kısmında, kullanmak istediğimiz storage class'ı seçiyoruz(ismini giriyoruz). Bu örnekte yukarıda örneğini paylaştığım "standarddisk" storage class'ı kullanıyoruz.

Bu PVC şu manaya geliyor.

Git "standarddisk" storage class tanımındaki, özelliklere göre "5G" ve erişim modu "ReadWriteOnce" olan bir persistent volume oluştur.

PVC 'yi aynı mantık ile pod'a mount edebiliyoruz. PV-PVC bölümünde mount ettiğimiz gibi.

{% code title="deploy.yaml" %}
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysqlsecret
type: Opaque
stringData:
  password: P@ssw0rd!
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysqldeployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
          ports:
            - containerPort: 3306
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              name: mysqlvolume
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysqlsecret
                  key: password
      volumes:
        - name: mysqlvolume
          persistentVolumeClaim:
            claimName: mysqlclaim
```
{% endcode %}

Artık storageclass sayesinde, PV objesini önceden oluşturmamıza gerek kalmıyor. PV objesini nasıl oluşturacağımızı bilmemize gerek kalmıyor.

Yani biz, kullanacağımız disk ünitelerine göre storageclass tanımı yapıp, bunu PVC objesi ile çağırmamız yetiyor. PV objesi, PVC objesini tanımladıktan sonra otomatik olarak oluşacak.

{% hint style="info" %}
Çoğu Bulut servis sağlayıcının birden fazla özel storage class'ları bulunuyor. Eğer bir bulut servis sağlayıcıdan yönetilen Kubernetes hizmeti alıyorsanız, bunları kullanabiliyorsunuz.
{% endhint %}

Hangi storage class'ı kullanmak istiyorsak, bu kullanmak istediğimiz storage class'ı belirttiğimiz bir PVC objesi yaratıp, bu PVC objesini de bir pod'da ihtiyacımız olan pathe mount etmemiz yeterlidir.

PVC yaml dosyasında bulunan, "storageclassname" de belirttiğimiz storage class'a bakarak, bir PV oluşturulur. Yani storage class aslında kullanacağımız disk ünitesine göre değişen, PV objesini otomatik olarak oluşturulmasını sağlayan bir template'dir diyebiliriz.

![](../.gitbook/assets/Right\_Sized\_PV.png)
