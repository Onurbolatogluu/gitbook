# 📂 ImagePullPolicy & ImageSecret

Bildiğiniz üzere bizler sıklıkla dockerhub gibi public registryler ile çalıştık. İmajları buralardan indirip, kullandık. Gerçek hayatta sadece public registryler ile uğraşmıyoruz. Kendimize ait bir private registry kullanıyor olabiliriz.&#x20;

Private registry olduğu zamanda, tahmin edebileceğiniz üzere, bunlardan imaj çekebilmek için authentication olmamız gerekmektedir. Bu authentication işlemini yapmadan, kubernetes cluster bu private registrylerden image çekemeyecek. Dolayısıyla da, containerlarımızı çalıştıramayacağız.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imagesecrettest1
  labels:
    app: imagesecret
spec:
  containers:
  - name: imagesecretcontainer
    image: registry.gitlab.com/onurbolatoglu/images:nginx
    ports:
    - containerPort: 80
```

* Yukarıda basit bir pod tanımımız mevcut. Bir adet pod oluşturmak istiyoruz. Bu pod'a bir isim veriyoruz ve bir label atıyoruz.
* Spec kısmında, container'a bir isim veriyoruz. Ardından da, container'ın hangi imajdan oluşmasını istediğimizi söylüyoruz.  Ancak bu haliyle pod''u oluşturamayız. Çünkü bu pod tanımında, ilgili registry'e erişmek için kullandığımız bir authentication bilgisi yer almıyor.



Private registrylere bağlanıp, imaj çekebilmek için, registry urli, username ve password bilgisine ihtiyacımız bulunuyor. Bu bilgileri  edindikten sonra, bu bilgileri bir secret olarak kubernetes'e gönderiyoruz, kubernetes de bir secret oluşturuyoruz. Daha sonra bu secret'ı da kullanmak istediğimiz, pod tanımlarında özel bir parametre ile set ediyoruz.&#x20;

Bu sayede bizim oluşturduğumuz secret içerisinde, kullanıcı adı,parola,url bilgilerini kullanarak, kubernetes; private registry'e authentication olabiliyor ve imajı lokaline indirebiliyor.

Örnek secret oluşturma;

```bash
kubectl create secret docker-registry "secret_ismi" --docker-server="registry_url" --docker-username="kullanıcı_adı" --docker-password="şifre"

Misal,

kubectl create secret docker-registry regcred --docker-server=registry.gitlab.com --docker-username=onrblt --docker-password=glpatdasd

```

* docker-registry tipinde bir secret oluşturuyoruz.&#x20;
* Secret oluştururken, ilgili private registry'de yetkili olan, kullanıcı ve parolası yanı sıra, url bilgisi de gereklidir.

Private registry 'e erişmek için, oluşturduğumuz secret'ı aşağıdaki örnekteki gibi kullanabiliyoruz;

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imagesecrettest2
  labels:
    app: imagesecret
spec:
  containers:
  - name: imagesecretcontainer
    image: registry.gitlab.com/onurbolatoglu/images:nginx
    imagePullPolicy: Always # IfNotPresent, Never
    # If the tag is latest, k8s defaults imagePullPolicy to Always. Otherwise k8s defaults imagePullPolicy to IfNotPresent
    ports:
    - containerPort: 80
  imagePullSecrets:
  - name: regcred
```

Hangi pod için bu registry kullanılacaksa, pod yaml dosyası içerisine "ImagePullSecrets" parametresi ile oluşturduğumuz secret'in ismini giriyoruz. Pod imajı indirmek için, "regcred" secret'ını kullanarak authentication olabilecek ve imajı kullanabilecek.

Diğer bir kısım ise, imagePullPolicy parametresidir;

Kubernetes nodelara imaj indirmek için, 3 farklı seçeneği kullanabiliriz.  Bu seçenekleri "imagePullPolicy" parametresi ile tanımlıyoruz.

imagePullPolicy parametresi eğer Always olarak seçili ise, her container oluşturulmaya çalışıldığında, her pod oluşturulmaya çalışıldığında, bu imaj tekrardan registry'den çekilecek. Node'da olup, olmaması önemli değil. Her pod oluşturulma esnasında, bu imaj ilgili registry'den indirilecek.&#x20;

Never seçili ise, bu sefer registry'e gidip, ilgili imajı çekmeye çalışmayacak. Kendi lokalindeki imajı kullanacak. Eğer ilgili imaj yoksa hata verecek. Hiçbir şekilde imajı gidip registry'den indirmeye çalışmaz, her zaman lokalindeki imajları kullanır.

IfNotPresent seçili ise, İlk önce lokale bakacak, kendi üzerinde bu imaj var mı, yok mu kontrol edecek. Yani pod'un schedule edileceği node, üzerinde ilgili imajı arayacak. Varsa kendi lokalindeki imajı kullanacak, eğer yoksa gidip registry'den ilgili imajı indirecek.



{% hint style="info" %}
Pod tanımlarında "latest" tanımlı bir imaj kullanıyorsanız, kubernetes varsayılan olarak "imagePullPolicy" parametresini, "Always" olarak set etmektedir. imagePullPolicy parametresini pod tanımı içerisinde eklemesek dahi, eğer kullandığımız imajın tag i "latest" ise, her zaman "Always" değeri kubernetes tarafından set edilir.

Misal olarak, "latest" tag i yerine, "v1" vb.. tagler kullanıyorsak, kubernetes, biz pod tanımına "imagePullPolicy" parametresini eklemesek bile, "IfNotPresent" değerini seçer.
{% endhint %}



{% embed url="https://juju.is/tutorials/using-gitlab-as-a-container-registry#7-pull-your-container" %}

{% embed url="https://kubernetes.io/docs/concepts/containers/images/" %}
