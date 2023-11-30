# 📲 Namespaces

Şöyle bir senaryo düşünelim, Misal, bi firmanın bilgi işlem sorumlusuyuz, ve firmada çalışan tüm ekiplerin ortak çalıştıkları, dosyalar barındırabilecekleri bir yapı tasarlıyoruz. 10 ayrı ekip var ve bu 10 ayrı ekibinde erişip, dosya barındırabileceği bir alt yapı kuracağız. Bu sistemi bir file server yapılandırarak kurabiliriz. Bir file server kurup, bunun altında tüm ekiplerin erişebileceği bir paylaşım alanı yaratıp, sonra tek tek kullanıcı bilgisayarlarının buraya erişmesini sağlarız. Böylece herkes dosyaları buraya koyup, buradan erişebilir. Sistemi böyle kurduğumuzda ilk etapta her şey düzgün çalışır. Hatta file server problem çıkarmadan, bu çözüm sorun çıkarmadan çalışır.  Ama birden fazla ekibin kullanacağı yapıyı, bu yaklaşım ile kurarsak, zaman içerisinde şu tarz problemler meydana gelir;

Öncelikle tüm kullanıcılar dosyalarını buraya yazdıkları için, tek bir paylaşım altında yüzlerce dosya oluşur. Ve bir zaman sonra yönetilemez hal alabilir. Bir diğer sıkıntı da, şu olur diyelim ki ben a.txt adında bir dosya oluşturdum. Başka biri de a.txt adında dosya oluşturup aynı yere atamaz. Dosyaların insanlara görünürlüğünü değiştirmek için, tek tek dosya bazından izinler yazılması gerekir.

Bu tür problemleri gidermek için, bu paylaşım altında ekip ve projeler için ayrı birer klasörler yaratırsak, tüm bu problemleri ortadan kaldırmış oluruz. Örneğin insan kaynakları için ik klasörü yaratırız. Böylece her ekibe özel bir dizin bulunur. Her ekip, kendi dizinlerinde dosyalarını barındırabilir. Bunu daha da genişletip, proje bazlı da klasörler yaratabiliriz.

Örneğin, yeni bir ürün geliştirmesi yapıyoruz. Bu ürün ile ilgili, tüm dosyaların tutulduğu ürün adında bir dizin oluşturup, ve projeye özel dosyaları burada saklayabiliriz. Bu sayede güvenlik\&erişim gibi ayarları klasör bazında yapabiliriz. Misal, şu X klasörüne Y kullanıcısı erişebilsin ancak değiştiremesin vb. Görüldüğü gibi klasörler kullanılarak binlerce dosya içerisinde kaybolmuyoruz. Klasörler bize gruplama imkanı tanır. Her kullanıcı grubu kendi dizinlerinde dosyalarını tutabilir. Dosya bazında tek tek, erişim izni, güvenlik ayarı yapmamıza gerek kalmaz. Çünkü bu ayarları klasör bazında yaparız. Ve yapılan bu ayarlamalar tüm klasör altında ki tüm dosyalar için geçerli olacaktır. Klasör bazında kotalar uygulayabiliriz.&#x20;

Kubernetes 'de namespace 'de tam olarak bu işi yapar.

* Namespace 'ler adlar için bir kapsam sağlar.
* Kaynak adlarının bir namespace içerisinde benzersiz olması gerekir.
* Namespace 'ler birbirlerinin içerisine yerleştirilemez ve her kubernetes kaynağı yalnızca bir namespace içerisinde olabilir.
* Namespace 'ler cluster kaynaklarını, birden çok kullanıcı arasında bölmenin bir yoludur.
* Kubernetes cluster'ı, örnekteki file server olarak düşünürsek, Namespace ise burada tanımladığımız klasörlerdir.
* Kubernetes altında objeleri, sanal klasörler altına oluşturabilir. Sonrasında bu sanal klasörler bazdında erişim ve kota ayarlamalarını yapabiliriz.
* Her kubernetes kurulumda varsayılan olarak 4 namespace oluşturulur.

kube-system : Kubernetes tarafından oluşturulan objelerin tutulduğu namespace 'dir.\
kube-public   : Kimliği doğrulanmamış olanlarda dahil, tüm kullanıcılar tarafından erişilmesine, ihtiyaç duyulan objelerin oluşturulabileceği yerdir.\
kube-nodel-lease : Node hard link işlemleri için gerekli özel bir namespace'dir.\


Kısacası kube ile başlayan namespace'ler, kubernetes tarafından oluşturulur. Cluster'ın işleyişi ile alakalı objelerin tutulduğu yerdir. Bunun yanı sıra, default adında namespace daha oluşturulur. Bizler aksini belirtmediğimiz sürece, tüm objeler default olarak burada oluşur.&#x20;

Birden fazla ekip tarafından yönetilen, birden fazla uygulamanın deploy edildiği kubernetes cluster 'ında, namespace bazında objeleri ayırmak sağlıklı olacaktır.

Namespace oluşturulurken, kubectl ve yaml dosyalarıdan yararlanabiliriz.

```
# Mevcut namespace'de çalışır.
kubectl get pods

# namespacecustom namespace 'i içerisinde bulunan podları getirir.
# -n argümanıda kullanılır.
# kubectl get pods --namespace namespacecustom

# app adında namespace oluşturmak.
kubectl create namespaces app

Namespace içerisinde bulunan objeler ile iletişim kurarken objenin namespace'ni 
belirtmemiz gerekir.

# Farklı bir namespace'de bulunan containera bağlandık.
# kubectl exec --it namepod -n development -- /bin/bash

# Mevcut komut satırı namespace değiştirme. Bu komutdan sorna tüm komutlarımız
# development namespace içerisinde çalışacak.
kubectl config set-context --current --namespace:development

# namespace silmek
kubectl delete namespace development
```



YML;

```
apiVersion: v1
kind: Namespace
metadata:
  name: hr
```

```
kubectl apply -f create-namespace.yml
```
