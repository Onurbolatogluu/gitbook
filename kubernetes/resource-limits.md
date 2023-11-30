# 🥐 Resource Limits

Containerlar biz aksini belirtmediğimiz sürece varsayılan olarak, üzerinde çalıştıkları hostun CPU ve Memory kaynaklarına sınırsız şekilde erişirler. Yani biz 8 GB memory sahip bir sistem üzerinde container çalıştırdığımız zaman, bu container içerisindeki uygulama 8 GB memory 'nin tamamını kullanabilir. Keza aynı şekilde, tüm CPU çekirdeklerini de kullanabilir.&#x20;

Bu durum özellikle kubernetes gibi yüzlerce container çalıştırdığımız dağıtık yapılarda, bizim için sorun oluşturur. Eğer bir pod koştuğu hostun üzerindeki tüm kaynakları tüketiyorsa, bu diğer podların işlerini düzgün yapmayacağı anlamına gelir. Bu nedenle, bu podların içerisinde çalışan, containerların ne kadar cpu ve memory kullanabileceklerini kısıtlamamız gerekmektedir. Pod tanımlarımızda, kullanabileceğimiz request ve limit opsiyonları bize bu imkanı verir.

Cpu kaynağı, CPU birimleri olarak ölçülür. Kubernetes de bir CPU şuna eşdeğerdir;

* AWS vCPU
* GCP Core
* Azure vCore
* Hyperthread Bare-Metal

Kesirli değerlere izin verilir. 0.5 CPU talep eden, bir container 1 CPU talep eden, bir container 'ın, yarısı kadar CPU alır. Mili alamında M son ekini kullanabiliriz.

Örneğin, 100m CPU, 100Milicpu ve 0.1 CPU aynıdır. 1M'den daha fazla ince hassasiyete izin verilmez. CPU her zaman, mutlak bir miktar olarak talep edilir. Asla göreli olarak talep edilmez.&#x20;

Misal;

1 CPU core kullanmak için,

cpu:"1" = cpu="1000" = cpu:"1000m"

1 CPU 'nun %10nu kullanması için,

cpu:"0.1" = cpu:"100" = cpu:"100m"



Kubernetes de CPU kısıtlamaları şu şekilde ayarlanır;

Her worker node'un belirli CPU kaynakları mevcuttur. Bu kaynaklar cloud üstünde yada sanal olarak çalşan stemlerde vCPU olarak, bare-metal sunucularda hyper thread olarak adlandırılır.

Biz bir pod tanımına cpu:1 tanımı eklersek, O pod 'un çalıştığı node üstünde core'lardan, sadece 1 tanesini kullanacağı anlamına gelir. Başka bir yazım şekli daha vardır. Her core 1000ms yani 1000mili cpu'luk güç demektir. Biz eğer cpu:100m yaparsak, pod çalıştığı host üstünde 1 core'un 10'da 1' gücüne erişebilir anlamına gelecektir. Yani, cpu:1 ile cpu:1000m aynı şeydir. Keza 0.1 cpu ile cpu:100m de aynıdır.

Memory tanımı daha kolaydır;

Cluster da mevcut durumda kullanılabilir bellek varsa, bir container belirlenen bellek isteğini aşabilir. Ancak bir container bellek sınırından fazlasını kullanmasına izin verilmez.  Bir container sınırından daha fazla bellek ayırırsa, container sonlandırma için aday olur. Container limitinin ötesinde, bellek tüketmeye devam ederse, container sonlandırılır. Sonlandırılmış container yeniden başlatılabiliyorsa, diğer herhangi bir runtime hatası türünde, olduğu gibi kubelet onu yeniden başlatır.

Containerın kullanabileceği maksimum memory 'i, byte cinsinden tanımlayabiliriz. Örneğin, memory:64m dersek, bu containerın, en fazla 64megabyte memory kullanmasına izin verdiğimiz anlamına gelir.

Buradaki m: megabyte g:gigabyte k:kilobyte kısaltmasıdır. Bunların yerine memory hesaplamasında kullanılan 2nin katlarını kullanan gösterim şekli olan, kibibyte,mebibyte,gibibyte tanımlarıda desteklenir.

CPU ve Memory kısıtlama tanımlarını, iki ayrı bölümde tanımlayabiliyoruz. \
Bunlar, request ve limit parametreleri.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: requestlimit
  name: requestlimit
spec:
  containers:
  - name: requestlimit
    image: obnet/stress
    resources:
      requests:
        memory: "64M"
        cpu: "250m"
      limits:
        memory: "256M"
        cpu: "0.5"
```

Request altında tanımladığımız değerler şu anlama geliyor, Kubernetes bu pod'u oluşturmaya başladığı zaman, scheduler bu podun oluşturacağı node seçecek. Bu seçme aşamasında kubernetese şunu diyoruz,  Bu podu en az 64megabyte memory, 250m milicpu yani çeyrek cpu core'un erişilebilir olduğu bir node üzerinde oluştur. Kısacası request kısmı bu podun oluşturulabilmesi için, minimum ne kadarlık boş kaynağın olması gerektiğini belirtiyor. Scheduler node seçiminde bu parametreleri de göz önüne alacak ve en az bu kadarlık kaynağın boş olduğu bir node seçecek. Eğer boşta 64mbyte memory ve  250m cpu sahip(çeyrek cpu), bir node bulamazsa, bu pod oluşturulmayacak.

Limit kısmı ise, tahmin edebileceğiniz üzere, bu containerın en fazla kullanabileceği sistem kaynağını belirtiyor. Yukarıdaki örnekte şunu diyoruz,

Bu pod içerisinde oluşturulan container, en falza 256megabyte memory kullanabilsin, en çok da yarım cpu (0.5) core 'na erişebilsin. Gördüğünüz gibi cpu kısmını istersek "0.5"  yada "1" gibi belirtebiliyor, yada "250m" gibi milicpu halinde yazabiliyoruz. Yani buradaki "0.5" yerine "500m" de yazsak aynı olurdu.&#x20;

Bu podu oluşturduğumuz da şu olacak, Öncelikle scheduler üstünde en az 64megabyte memory ve çeyrek cpu core kaynağının boşta olduğu bir node bulacak, ve bu pod o node üzerinde oluşturulacak. Pod oluşturulacak ve container çalışmaya başlayacak. Bu container ise, en fazla yarım cpu (0.5) core 'unun gücüne denk gelecek şekilde bir CPU kullanabilecek ve sistemden en fazla 256mbyte allocate edecek.

Bu Container eğer yarım core 'dan fazla CPU kaynağına erişmek isterse ne olacak?

Bu durum mümkün olmayacak, CPU da belirlediğimiz Limit neyse, bu container en fazla o kadar CPU kaynağına erişebilecek.&#x20;

Memory kısmında belirlediğimiz değer, bu containerın maksimum kullanabileceği memory değeridir. Fakat container bu değere geldiği zaman, daha fazlasını kullanamaz diye bir durum söz konusu değildir. Memory allocation CPU gibi çalışmıyor. teknik olarak farklı şeyler.

Bu nedenle, bu container içerisindeki uygulama sistemden daha fazla ram talebinde bulunabiliyor.  Sistem varsayılan olarak bunu engelleme şansına sahip değil. İşte bu nedenle, eğer container memory limitine geldiğinde daha fazla memory allocate edilmesi için bir mekanizma olmadığından, bu duruma geldiğinde OOMKilled (out off memory) kill durumuna geçerek restart ediliyor. Eğer uygulamamız burada belirttiğimiz, memory kısıtından daha çok memory talep ederse pod restart edilecek. Bu nedenle, bu kısıtları uygulamamızın davranışına göre belirlememizin davranışına göre belirlememiz gerekiyor.

Kubernetes node'lar üstünde cpu ve memory kullanımının kontrol edilmesi.

```bash
kubectl top node 
# kubectl top node "node_ismi" ile tekil bakılabilir 
```

Kubernetes pod'lar üstünde cpu ve memory kullanımının kontrol edilmesi.

```bash
kubectl top pod 
# kubectl top pod "pod_ismi" ile tekil bakılabilir 
```

