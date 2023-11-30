# 📦 Deployments

Kubernetes 'de container imajına çevirdiğimiz objeleri pod olarak deploy ederiz. En temel obje pod 'dur. Genelde Kubernetes üzerinde singleton (tekil) yönetilmeyen pod 'lar yaratmayız. Pod 'ları yöneten üst seviye objeler yaratırız, pod 'lar bu objeler tarafından, yaratılıp yönetilir.&#x20;

Pod 'ları tek bir worker node üzerinde çalıştırmak, uygulama sağlığı açısından istenmeyen bir durumdur. Worker node 'a fiziksel olarak bir problem meydana geldiğinde, üzerinde bulunan pod 'lara erişim kaybolur. Ve uygulamamıza erişemeyiz.

Ayrı ayrı, yml dosyalarını kullanarak, farklı worker node üzerinde aynı imaj kullanarak pod oluşturduğumuz zaman ise, hem takip edilmesi güç, hem de manuel işlem yapmamız gerekecektir. Her worker node için ayrı bir yml dosyası oluşturup, hepsiyle ayrı ayrı ilgilenmemiz gerekecektir.

Deployment 'da istenilen durumu tanımlarız, ve deployment controller, mevcut durumu, istenilen durum ile karşılaştırıp, gerekli aksiyonları alır.

Deployment bir veya birden fazla pod 'u bizim belirlediğimiz desired(istenilen) state göre oluşturan, sonrasında bu desired state göre mevcut durum ile sürekli karşılaştırıp, gerektiği düzeltmeleri yapan obje tipidir.&#x20;

Deployment obje oluşturmak için tanımı yapar, bu tanımın içinde oluşturmak istediğimiz, pod 'un hangi özelliklere sahip olacağını ve kaç adet olacağını belirtiriz. Deployment objesi oluştuğu zaman tanımladığımız özelliklerde ve adette pod oluşturulur.

Misal, Nginx imajından 3 tane pod oluşturan deployment yaratırız. Deployment bunu desired state olarak alır ve 3 pod oluşturur. Ardından deployment controller devreye girer ve sürekli olarak bu pod 'ları gözler. Bir pod 'da sorun olduğu zaman deployment controller duruma müdahale eder, sorunlu olan pod yerine, yeni bir pod oluşturur. Böylelikle desired state ve current state eşitlenir.

Deployment objeleri kurallar dahilinde pod güncellemeleri yapmamıza da imkan sağlar. Misal Desired state 'e eski imajımız yerine güncel imajımızı yazarsak, Deployment controller istediğimiz imaja göre pod 'ları oluşturacaktır. Bu işlemi kontrollü bir şekilde yapması için, ek parametreler belirtebiliriz.  Misal, yeni imaj 'dan bir pod yarat, eski imaja sahip olan pod 'lardan birini sil ve sorun yoksa, kalan diğerlerini bu sırayla yap.

Böylelikle uygulamamızın yeni versiyonunu kesintisiz geçiş imkanı sağlıyoruz. Sorun olması halinde tek bir komut ile deployment ortamımızı bir komut ile bir önceki duruma alabiliriz.

* Imperative yöntemle Deployment oluşturma.

```
kubectl create deployment firstdeployment --image=nginx:latest --replicas=2
```

* Deployment listeleme.

```
kubectl get deployment
```

* Deployment silme.

```
kubectl delete deployment firstdeployment
```

* Deployment içerisindeki imajı güncelleme.

```
kubectl set image deployment/firstdeployment nginx=httpd:alpine
```

* Deployment replika sayısını değiştirme "Scaling"

```
kubectl scale deployment firstdeployment --replicas=5
```

* Deployment yapılan son değişikliğin geri alınması.

```
kubectl rollout undo deployment firstdeployment
```

* Deployment yapılan değişikliklerin kaydının tutulması için "--record" opsiyonu kullanılır.

```
kubectl set image deployment rolldeployment nginx=httpd:alpine --record=true 
```

* Deployment yapılan değişikliklerin listelenmesi.

```
kubectl rollout history deployment rolldeployment
```

* Deployment yapılan değişikliklerin izlenmesi.

```
kubectl rollout status deployment rolldeployment 
```

* Deployment üstünde yapılan değişikliklerin durdurulması.

```
kubectl rollout pause deployment rolldeployment
```

* Durdurulan rollout'un devam ettirilmesi.

```
kubectl rollout resume deployment rolldeployment
```



* YAML ile deployment oluşturmak;

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: firstdeployment2
  labels:
    team: development
spec:
  replicas: 3
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
        labels:
          app: front-end
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```
kubectl apply -f deploymentcreate.yml
```

* template argümanı altında oluşturmak istediğimiz pod özelliklerini gireriz.
* selector argümanı girerek şunu söylemiş oluyoruz;
  * Deployment, senin yönetebileceğin pod 'lar, app:front-end etiketine sahip olacak, Bu etiketi görürsen anla ki, bu pod 'lar sana ait.
* Selector ve template altında olan label tanımları mutlaka her deployment obje oluştururken bulunmalıdır. Birden fazla deployment bulunan ortamlarda farklı  farklı label kullanılması karışıklık yaşanmaması için önemlidir.
* Label 'ların objeye özel uniq olması çok önemlidir. Farklı farklı obje 'lerde aynı label tanımının olması karışıklığa yol açacaktır.

