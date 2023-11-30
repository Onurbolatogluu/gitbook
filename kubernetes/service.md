# 🌏 Service

* Bir dizi pod üzerinde çalışan uygulamayı, ağ hizmeti olarak göstermenin soyut bir yoludur.
* Kubernetes ile, uygulamanızı tanıdık olmayan, bir hizmet bulma mekanizması kullanılacak şekilde, değiştirmeniz gerekmez. Kubernetes pod 'lara, kendi IP adreslerini ve bir dizi pod için, tek bir DNS adı verir. Ve bunlar aralarında yük dengeleyebilir.

4 Tip Service Mevcuttur.

### ClusterIP

![](../.gitbook/assets/1\_a8lwtd3acs-hiiotbcdf2q.png)

ClusterIP tipinde, service objesi yaratıp, bunu label selector arayıcılığıyla pod 'lar ile ilişkilendirebiliriz. Bu objeyi yarattığımız zaman, bu obje cluster içerisindeki tüm pod 'ların ve kaynakların çözebileceği uniq bir dns ismine sahip olur.

Bunun yanında, bizler her kubernetes kurulumunda  --service-cluster-ip-range ile sanal IP bloğu belirleriz. Misal 10.100.0.0/16

Cluster IP service objesi, yarattığımız zaman, bu objeye, bu IP bloğundan bir sanal IP adresi, Cluster 'ın DNS mekanizmasın da kaydedilir.

Bu IP adresi, Virtual bir IP adresidir. herhangi bir interface 'e map edilmez. kube-proxy servisi tarafından tüm node 'lardaki IPTable 'lara eklenir ve buraya ulaşan trafik, servisin ilişkili olduğu pod 'lara round-robin algoritmasıyla yönlendirilir. Yani Cluster içerisinden, Bu IP adresine gönderilen paketler, bu service ile ilişkilendirilmiş pod 'lardan bir tanesine yönlendirilir.

Şöyle bir senaryo düşünelim, Frontend ve Backend deployment adlarında 2 farklı deployment'ımız var. Ancak frontend altındaki pod 'lar, Backend altındaki podları tanımıyor. bilmiyor.

Bu durumda,

Backend pod 'lar ile ilişki kuracak, backend isimli bir servis yaratıp, Frontend'lere de backend pod 'lara ulaşmak istediğinde, gitmek gereken adres, backend isimli servis(adres) dersem, şu olur. Cluster içerisindeki herhangi bir pod, backend ismini çözmek istediğinde, ona cevap olarak, bu backend servisinin IP adresi döner. Pod o IP adresine ulaşmak istediğinde de, trafik backend pod'lardan birine yönlendirilir.

{% hint style="info" %}
Farklı namespace 'lerden bu service erişmek istenirse, tam isminin yazılması&#x20;

gerekmektedir. Aynı namespace içerisinde bu gerekli değildir.
{% endhint %}

Dolayısıyla tek tek frontend ' pod 'lara bağlanıp, her seferinde yeni eklenen pod 'ların IP adreslerini belirtmemiz gerekmez. Sadece gitmek istediğim yer backend deriz, O trafiği arkada bulunan backend servisine gönderir. Gerisini cluster halleder.  Yeni eklenen pod'lar otomatik olarak bu service altına eklenir. Çıkarılan/Silinen pod'lar otomatik olarak bu servisten çıkarılır.

ClusterIP kubernetes altında, basit de olsa service discovery ve Load Balancing hizmetleri sunan obje tipidir. Böylece cluster içerisinde Frontend-Backend bağlantı sorunlarını çözdük.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
```

### NodePort

2.sorun, frontend pod'lara dışarıdan erişilmesi gerekiyor.

Bu sorunu, nodeport service objesi ile çözeriz. Aynı cluster IP service objesi gibi, nodeport 'da service obje tipidir.&#x20;

![](../.gitbook/assets/1\_vE8I8ee8JBPrM54DksA8cA.png)

Biz bir nodeport service objesi yaratır. Bunu label selector sayesinde, ihtiyacımız olan pod 'lar ile ilişkilendirebiliriz.

Bu obje yaratıldığı zaman şu olur, 30000-32767 arasında bir port seçilir ve tüm worker node 'larda bu port bizim frontend deployment 'mıza proxy edilir. Ve dışarıdan bu servisi oluşturduğumuz anda bizim uygulamamız için tanımladığı port üzerinden erişebiliriz. Kısacası, worker node 'un, IP adresinin, bu portuna gelen paketler, ilişkilendirdiğimiz pod 'ların publish ettiğimiz portlarına yönlendirilir. Bizlerde worker node 'ların, önüne bir Reverse Proxy veya Load Balancer koyarak, dış dünyadan erişilmesini sağlayabiliriz.  Buraya ulaşacak paketler, worker node'lara, worker node'lara ulaşan paketlerde pod 'lardan birine ulaşarak kullanıcıların uygulamamıza gelmesini sağlayabiliriz.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### Load Balancer

Load Balancer tipi service objelerini, sadece cloud service sağlayıcılar kullandığımızda kullanabiliriz.   Load balancer tipi bir service oluşturduğumuzda, cloud service sağlayıcı, dış dünyadan erişilebilecek bir public IP adresine sahip, bir LB oluşturur, sonrasında bu LB bizim service  objesine ilişkilendirilir. Dolayısıyla bu public adrese ulaşan paketler içeriden pod 'lara yönlendirilir.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontendlb
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

NodePort ile uygulamızın önüne bir LB ve Proxy koymaktan bahsetmiştik. Cloud service sağlayıcıları bunu otomatik sağlıyor. Buna da Load Balancer Service obje diyoruz.&#x20;

Biz bir servis oluşturduğumuz zaman, bu servis endpoint adında bir obje oluşturur. Ve bu endpoint objesi, bizim belirlediğimiz selector tarafından seçilen pod 'ların, IP adreslerini üstünde barındırır. Service trafiği nereye yönlendireceğini bu listeye göre düzenler Bu liste dinamiktir, pod sildiğimizde o pod bu listeden IP adresi silinir. Yeni pod oluşturduğumuz da o pod 'un IP adresi bu listeye eklenir.



{% embed url="https://mvallim.github.io/kubernetes-under-the-hood/documentation/kube-metallb.html" %}
\
BareMetal sunucularda çalışan clusterlarınızda MetalLB kullanabilirsiniz.
{% endembed %}

## **ExternalName** <a href="#b7e0" id="b7e0"></a>

ExternalName türü ile tanımlanan bir Kubernetes Service’i önceki türlerde olduğu gibi selector kullanmak yerine DNS adını kullanacaktır. Bu türde, önceki türlerde olan proxy ya da forward işlemleri kullanılmamaktadır. Yönlendirme işlemi DNS seviyesinde gerçekleşmektedir.





```
Service objelerinin listelenmesi

kubectl get service


Bir deployment'in expose edilmesi "imperative olarak service objesinin oluşturulması"

kubectl expose deployment "deployment_ismi" --type="service_tipi" --name="servis_ismi"

Ör: kubectl expose deployment backend --type=ClusterIP --name=backend


Service objelerinin silinmesi

kubectl delete service "servis_ismi"

Ör: kubectl delete service backend
```
