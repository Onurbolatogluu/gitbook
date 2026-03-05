# Demo

Bir önceki yazımızda Kubernetes Service tiplerinin teorik olarak ne işe yaradığını, hangi durumlarda hangisini seçeceğimizi (ClusterIP, NodePort, LoadBalancer, Headless, ExternalName) detaylıca incelemiştik.

Peki bunlar gerçek bir terminal ekranında nasıl görünüyor? Ağ trafiği fiziksel olarak nasıl akıyor? Bu makalede, standart bir Nginx web sunucusu üzerinden tüm bu servis tiplerini adım adım ayağa kaldıracak ve içeriden/dışarıdan nasıl tepki verdiklerini test edeceğiz.

### Sahneyi Kuruyoruz: Nginx Pod'larımızı Hazırlayalım

Testlerimize başlamadan önce, trafiği karşılayacak uygulamalarımıza ihtiyacımız var. Arka planda çalışacak ve `role=nginx` etiketine (label) sahip 3 adet Nginx Pod'u ayağa kaldırdığımızı varsayalım.

Sistemimizin şu anki durumu:

```shellscript
kubectl get pods

# ÇIKTI:
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7ff69d756-8qdv8    1/1     Running   0          3m
# nginx-deployment-7ff69d756-hccjn    1/1     Running   0          3m
# nginx-deployment-7ff69d756-stpmz    1/1     Running   0          3m
```

Uygulamalarımız hazır! Şimdi bu 3 Pod'a farklı yollardan ulaşmayı deneyelim.

***

### 1. ClusterIP (İç Ağ Testi)

Varsayılan servis tipimiz olan `ClusterIP`, dış dünyaya kapalı, sadece Cluster içindeki uygulamaların birbirleriyle konuşabildiği sistemdir.

Adım 1: Servisi Oluşturma

Aşağıdaki YAML dosyasını oluşturup uygulayalım:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-svc
  namespace: default
spec:
  type: ClusterIP
  selector:
    role: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
```

```bash
kubectl apply -f clusterip-svc.yaml
kubectl get svc clusterip-svc

# ÇIKTI: Servisimiz 10.102.157.139 dahili IP'sini aldı.
# NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# clusterip-svc   ClusterIP   10.102.157.139   <none>        80/TCP    5m
```

Adım 2: Bağlantıyı Test Etme

Bu servis dışarıya kapalı olduğu için kendi bilgisayarımızdan tarayıcıya bu IP'yi yazarsak ulaşamayız. Test etmek için Cluster'ın _içine_ sızmamız lazım. Bunun için geçici bir "Test (Debug) Pod'u" yaratıp içine giriyoruz:

```shellscript
kubectl run -i --tty --rm debug --image=curlimages/curl --restart=Never -- sh
```

Artık içerideyiz! Nginx sunucumuza istek atalım:

```bash
# Debug podunun içinden:
curl http://clusterip-svc.default.svc.cluster.local
```

_Ekranda Nginx'in "Welcome to nginx!" HTML sayfasını gördüyseniz, iç ağ bağlantınız kusursuz çalışıyor demektir._

***

### 2. NodePort (Dış Dünyaya İlk Kapı)

Sistemimizi Cluster dışından, doğrudan fiziksel sunucunun IP'si üzerinden test etmek istiyoruz.

Adım 1: Servisi Oluşturma

Statik olarak `30000` portunu dışarı açan NodePort servisimizi kuralım:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-svc
  namespace: default
spec:
  type: NodePort
  selector:
    role: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30000   # Dışarıdan ulaşacağımız port
```

```bash
kubectl apply -f nodeport-svc.yaml
kubectl get svc nodeport-svc

# ÇIKTI: 80 portunun dışarıdaki 30000 portuna bağlandığını görüyoruz.
# NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
# nodeport-svc   NodePort   10.98.229.84   <none>        80:30000/TCP   5m
```

Adım 2: Bağlantıyı Test Etme

Bu kez Cluster'ın içine girmemize gerek yok. Doğrudan Kubernetes'in çalıştığı sunucunun (Node) IP adresini bulalım:

```bash
kubectl get node node01 -o jsonpath='{.status.addresses[?(@.type=="InternalIP")].address}'
# Örnek Çıktı: 192.168.1.50
```

Şimdi kendi bilgisayarımızın terminalinden veya tarayıcısından bu IP ve Port'a gidelim:

```bash
curl http://192.168.1.50:30000
```

_Nginx karşılama sayfasını dış dünyadan başarıyla görüntüledik!_

***

### 3. Headless Service

Trafiği rastgele dağıtan bir Load Balancer istemiyoruz. Uygulamamızın arkadaki 3 Nginx Pod'unun IP listesini doğrudan görmesini istiyoruz.

Adım 1: Servisi Oluşturma

Kritik nokta `clusterIP: None` ayarıdır:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-svc
  namespace: default
spec:
  clusterIP: None       # Bu satır yük dağıtıcıyı iptal eder
  selector:
    role: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
```

Adım 2: Bağlantıyı Test Etme

Bu servisin bize tek bir IP vermek yerine bir liste dönüp dönmediğini kontrol edelim. Tekrar geçici Debug Pod'umuzun içine giriyoruz:

```bash
kubectl run -i --tty --rm debug --image=curlimages/curl --restart=Never -- sh
```

İçeriden DNS sunucusuna soralım: "Bana headless-svc'nin adreslerini ver":

```bash
nslookup headless-svc.default.svc.cluster.local

# ÇIKTI (Sihir burada gerçekleşiyor):
# Address: 10.1.0.5
# Address: 10.1.0.6
# Address: 10.1.0.7
```

_Gördüğünüz gibi Kubernetes aradan çekildi ve arkada çalışan 3 Pod'un gerçek IP adreslerini bize liste halinde teslim etti._

***

### 4. ExternalName

Son olarak, içerideki bir uygulamanın Cluster dışındaki bir API'ye (örneğin `httpbin.org`) gitmesini istiyoruz ancak kodumuzu değiştirmek istemiyoruz.

Adım 1: Servisi Oluşturma

Bu servisin `selector` (Pod seçici) veya port ayarları yoktur. Sadece bir DNS takma adıdır (CNAME).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: externalname-svc
  namespace: default
spec:
  type: ExternalName
  externalName: httpbin.org
```

Adım 2: Bağlantıyı Test Etme

Debug Pod'umuzun içinden, sanki içerideki bir servise gidiyormuşuz gibi istek atalım:

```bash
# Debug podunun içinden:
curl http://externalname-svc.default.svc.cluster.local/get
```

_Siz içerideki Kubernetes servisine istek attığınızı sanırken, sistem sizi anında şeffaf bir şekilde dışarıdaki `httpbin.org/get` adresine yönlendirecek ve dış dünyanın cevabını getirecektir._

