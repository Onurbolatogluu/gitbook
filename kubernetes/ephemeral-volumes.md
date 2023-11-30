# 📃 Ephemeral Volumes

Containerlar, stateless uygulamalar için oldukça uygundur. Bildiğimiz üzere containerlar, container imajı dediğimiz objelerden oluşturulur. Container içersinde çalışması istenilen uygulama ve bu uygulama bağlılıkları read-only bir paket haline getirilir ve bu pakete container imajı deriz. Bu paketten bir container yaratılır. Container yaratıldığı zaman, bu containera ait tüm veriler, bu containera atanan layer da tutulur. Yani container içerisinde oluşturulan her türlü dosya containera ait ayrı bir layer da durur ve container ayakta olduğu sürece bu veri erişilebilirdir.

Ancak container silinirse, bu veriler de silinir. Aynı imajdan yeni bir container yarattığımız zaman, bu container da, eski container da yazılan dosyalar bulunmayacaktır.

Bu durum stateless veya yazdığı verinin container kapatıldığında, silinmesinin sorun olmadığı uygulamalar için sıkıntı değildir.

Misal, gelen paketi işleyen, sonucu başka bir servise gönderen bir backend uygulaması üstünde bir veri barındırmadığı için ve herhangi bir state tutmadığı için sıkıntı çıkarmaz.&#x20;

Misal, bir frontend uygulamamız, bir web sitemiz olduğunu düşünürsek, kullanıcılarımız buraya browser arayıcılığıyla bağlanıyorlar.  Ve bu frontend uygulamamız da, başka bir servise bağlanıp, çeşitli resim dosyalarını çekip, bunları kullanıcılara gösteriyor. Uygulama bu resim dosyalarını, bir defa çektikten sonra, bunu container içerisinde, bir dizine saklıyor ki, bir daha bu resmi göstermesi gerekirse, diğer servise bağlanıp, tekrar çekmekle zaman kaybetmesin.  Yani bir nevi bu görselleri kendi üzerinde cacheliyor.&#x20;

Bu uygulamayı kubernetes üzerinde deploy etmek için, bir deployment objesi oluşturduk ve bu uygulama bir replika olacak şekilde deploy ettik. Podumuz oluşturuldu ve çalışmaya başladı. Buraya erişebilecek servis objesini de oluşturduk. Böylece bu uygulamaya kullanıcılar bağlanabilir duruma geldi.

Ardından da, backend uygulaması ve servisini de oluşturduk. Kullanıcıları web sitemizi kullanmaya başladılar. Diyelim ki çeşitli resimleri seçtiler ve bizim uygulamamızda, yapması gereken işlemlere başladı. Diğer servise bağlandı ve bu resimleri çekti, işledi ve bağlı kullanıcılara servis etti.&#x20;

Ama aynı zamanda, bu resimlerin, container içinde cache isimli dizine de kaydetti. Bir başka kullanıcı daha aynı resmi istediği zaman, tekrar zahmet edip diğer uygulamaya bağlanmasına gerek kalmadan, resimleri cache dizininden, çekerek gösterebilir duruma geldi. Böylece hız kazanmış olduk.

Misal, pod tanımında, liveness probe da var. Bir problem çıktığını düşünelim ve liveness probe fail etti. Bu durumda kubelet hemen devreye girip, bu container'ı silip, yeniden oluşturdu. Yani "kubectl get pods" kısmında restart olarak gözüken şey aslında container'ın yeniden oluşturulmasıdır.&#x20;

Container silindiği zaman içindeki veriler de siliniyor. Dolayısıyla, yeni container, içerisinde bu cache dizini boş olacaktır. Çok büyük bir sıkıntı değil. Sonuç olarak bunlar cache dosyaları. Tekrar diğer servise bağlanarak resimleri çekebiliriz.

Ama sırf yeni container oluştuğunda cache dosyalarının silinmese, ben bu pod içerisinde tüm containerların erişebileceği ve podun yaşam süresi boyunca ayakta kalabilecek, bir mekanizmaya sahip olsam ve dosyaları bu belirttiğim yere yazabilsem.

Böyle bir senaryom olsa, ben bu cache dizinini, buraya bağlarım ve içerisine yazılan tüm dosyalar buraya yazılır. Yeni bir container oluşturulduğunda da, bu dosyalara erişmeye devam edebiliriz. Böylece cache den mahrum kalmayız.&#x20;

Pod silinirse bu dosyalar da silinebilir. Önemli olan, pod ayakta durduğu sürece erişebileceğim ve containerın yaşam süresinden bağımsız bir şekilde, dosya saklayabileceğim, bir yapıya sahip olalım. Böyle bir yapı, bu senaryo da sorunumuzu çözerdi.

Kubernetes de bu sorunu Ephemeral Volume ile çözüyoruz.

Ephemeral volume sayesinde, bizler bu tarz verileri fiziksel olarak container dışında tutma imkanına sahip oluyoruz. Bu sayede container silindiği zaman, yeni oluşturulan containerında, bu verilere ulaşabilmesini sağlıyoruz. Kubernetes pod tanımlarında kullanabilmemiz adına 2 tip ephemeral volume mevcuttur.

### 1 - EmptyDir

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir
spec:
  containers:
  - name: frontend
    image: onurbolatoglu/apps1:latest
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthcheck
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: cache-vol
      mountPath: /cache
  - name: sidecar
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "sleep 3600"]
    volumeMounts:
    - name: cache-vol
      mountPath: /tmp/log
  volumes:
  - name: cache-vol
    emptyDir: {}
```

emptyDir volume ilk olarak bir pod, bir node 'a atandığında oluşturulur ve bu pod, o node 'da çalıştığı sürece var olur. Adından da anlaşılacağı gibi emptyDir başlangıçta boştur. Pod içindeki tüm containerlar emptydir volume 'deki  aynı dosyaları okuyabilir ve yazabilir. Bu volume her kapsayıcıda aynı veya farklı path'lere mount edilebilir.&#x20;

Bir pod herhangi bir nedenle silindiğinde, emptyDir içindeki veriler kalıcı olarak silinir. Biz bir emptyDir tipinde volume yaratmak için, Pod tanımında gerekli ayarları yaptığımız zaman, kubernetes bu podun oluşturulduğu node  üzerinde boş bir dizin yaratır.&#x20;

Daha sonra biz bu klasörü container içerisindeki, herhangi bir dizine mount edebiliriz. Bizim örneğimizde ki /cache klasörüne, bu containerın bu mount ettiğimiz pathe yazdığı her türlü dosya, fiziksel olarak node üstünde oluşturulmuş olan, boş dizine yazılır. Dolayısıyla container silinse bile, bu dosyalar silinmez. Yeni container oluşturulduğu zaman tekrar bu volume, container da, aynı pathe mount edilir. Bu sayede yeni oluşturulmuş container da, bu dosyalara erişmeye devam eder.

<mark style="color:red;">Bu dosyalar, POD 'un yaşam süresi boyunca erişilebilir durumdadır. Yani POD silinirse bu volume de silinir. Pod içerisinde bulunan container silinip, yeniden oluşturulabilir. Bu durumda var olan dosyalar etkilenmez ancak POD silinirse bu dosyalar kalıcı olarak silinir.</mark>

İster biz, ister kubernetes bir sorun anında, POD u silip yeniden başka bir pod oluşturursa, volume de silinmiş olur. Bu nedenle bu volume'lere ephemeral volume yani geçici volumeler diyoruz.

Bu volumeler sadece, bir şekilde geçici olarak podun yaşam süresi boyunca, containerın, yaşam süresinden bağımsız olarak saklamamız gereken geçici dosyaları tutmak için kullanırız.

### 2 - Host Path

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  containers:
  - name: hostpathcontainer
    image: ubuntu:latest
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthcheck
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: directory-vol
      mountPath: /dir1
    - name: dircreate-vol
      mountPath: /cache
    - name: file-vol
      mountPath: /cache/config.json       
  volumes:
  - name: directory-vol
    hostPath:
      path: /tmp
      type: Directory
  - name: dircreate-vol
    hostPath:
      path: /cache
      type: DirectoryOrCreate
  - name: file-vol
    hostPath:
      path: /cache/config.json
      type: FileOrCreate
```

Bir hostpath volume, worker node dosya sisteminden podumuza bir dosya veya dizini bağlayabilme imkanı verir. Bu çoğu podun ihtiyaç duyacağı bir şey değildir. Ancak bazı uygulamalar için, güçlü bir kaçış kapısı sunar.&#x20;

Hostpath, temel de emptydir ile mantık olarak aynıdır. Yine biz volume yaratılması için, pod tanımlarımıza gerekli parametreleri gireriz. Fakat bu sefer rastgele boş bir klasör yaratmasını söylemek yerine, podun oluşturulacağı worker node üstündeki, spesifik bir dizini veya dosyayı belirtiriz.&#x20;

Misal, worker node üstündeki /tmp dizinini volume olarak kullanılmasını istediğimizi belirtebiliriz. Daha sonra bu dizini veya dosyayı container içerisindeki herhangi bir dizine mount edebiliriz.&#x20;

Aynı şekilde, containerdan yazılan dosyalar bu mount ettiğimiz dizine yazılır ve container silinse de dosyalar silinmez. Genellikle podların çalıştıkları worker node üzerinde spesifik bir dizin ve dosyalara erişmesi gerektiği durumlarda işimize yarar. Misal bir path de konumlandırılmış bir unix soketine containerın bağlanması gerekirse, bunu hostpath ile containera mount edebiliriz.



Volume oluşturmak için,

* Volume parametresi ile oluşturulmak istenilen volume specs altında tanımlamak gerekiyor. Name anahtarı ile isim verilmeli ve volume tipini girmeliyiz.
* Bu volume containerda belirlediğimiz, ihtiyacımız olan pathe mount etmeliyiz. Volumemount anahtarı kullanarak mount etmek istediğimiz volume ismini ve mountpath ile bu volume bağlamak istediğimiz pathi belirlemeliyiz.

Hostpath için MountPath tipleri,

* directory => Bağlanacak Dosya(Dizin) zaten hali hazırda ilgili worker nodelar üzerinde mevcutsa, ilgili dosyayı veya dizini direkt olarak containera bağla.
* DirectoryOrCreate ve FileOrCreate => Bağlanacak Dosya veya Dizin worker nodelar üzerinde varlığı bilinmiyorsa, kullanılır. Yani ilgili dizin ve ya dosya mevcutsa bağla, mevcut değilse oluştur ve ardından containera bağla.



{% hint style="info" %}
Ephemeral volume bizim depolama sıkıntımızın tamamını çözmez. Bizim bazı durumlarda pod yaşam süresinden bağımsız şekilde,  uzun süreli saklamamız gereken dosyalar da olabiliyor ve bunları ephemeral volume ile çözmemiz mümkün değil. Fakat bunun da kubernetes de farklı bir çözümü var. Bu çözümden de ilerleyen yazılarımda bahsedeceğim.
{% endhint %}
