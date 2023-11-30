# 🕹 Job & Cronjob

Şu ana kadar gördüğümüz tüm kubernetes objelerinin tamamı müdahale edilip, kapatılmadığı sürece çalışmaya devam etmesi gereken uygulamalar ile ilgiliydi. Fakat bazı iş yükleri bu şekilde uzun süre çalışması gereken uygulamalar değildirler. Çalıştırılıp, işini yapıp, kapanması gereken bir çok uygulamalar mevcuttur.&#x20;

Eğer bu uygulamaları, diğer kubernetes objeleri (pod,deployment vb) ile oluşturursak, bunlar her kapandığında yeniden başlatılacak. Misal bir veritabanı uygulamamız var, biz buna bağlanıp "demo" tablosu adında tablo yaratan başka bir uygulama oluşturduk. Bu "demo" tablosunu yaratan uygulamayı gün içerisinde, ihtiyacamız olduğu anda çalıştırarak, bu tabloyu oluşturmak istiyoruz. Bu tablonun oluşup, verilerin set edilmesi yaklaşık 10 dakika sürdüğünü düşünelim. Biz bu uygulamayı container haline getirip, single pod olarak deploy etsek, uygulama çalışıp, işini halledip düzgün bir şekilde kapanırsa sıkıntı olmayacak. Fakat uygulama çakılırsa, single pod bunu tespit edip, yeniden başlamayacak. Bunun yanında, tek pod değilde aynı anda paralel olarak 3-5 pod birden çalıştırmak istesem, bu seferde single pod işimize yaramayacak. Mecburen Deployment objesi oluşturacağız, fakat bu seferde sıkıntı şu ki, benim bu uygulamamın, tek sefer çalışıp, işini düzgün yaparsa kapanması gerekiyor. Fakat deployment container kapandığında yeniden başlatacak. İşte tüm bu sıkıntıları gidermek adına kubernetes job objesi mevcut.

Bir job objesi, bir veya daha fazla pod oluşturur ve belirli bir sayıda, pod başarıyla sonlanana kadar pod yürütmeyi yeniden denemeye devam eder. Job belirtilen sayıda podun başarıyla tamamlanma durumunu izler, belirtilen sayıda podun başarıyla tamamlandığında job tamamlanır. bir job'un silinmesi oluşturduğu podları temizleyecektir.

Pod içersinde uygulama, başarıyla tamamlanıp, kapatılana kadar bekler. Eğer container fail ederse yeniden oluşturulur. job'un bir özelliğide, iş bittiğinde podları silmemesidir. Bu sayede podlar tarafından oluşturulan loglar kontrol edilebilir.

Job temelde 2 senaryo için kullanırız; ilk senaryo yukarıda da bahsettiğimiz senaryodur. Tek seferlik çalışıp, yapması gerekeni yaparak kapanan uygulamalarımızı job olarak deploy ederiz. Misal scriptler vb.

2\. senaryo ise, bir kuyruk veya işlenmesi gerken bir çok işimiz olduğunda, bunları eritmek adına, bu işler eriyene kadar çalışacak uygulamaları job şeklinde deploy ederiz.

Örnek job yaml dosyası içeriği;

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  parallelism: 2
  completions: 10
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl",  "-Mbignum=bpi", "-wle", "'print bpi(2000)'"]
      restartPolicy: Never
```

apiVersion: batch/v1 olan, Kind:job olan ve ismi de, pi olan bir job oluşturmamıza yarayan yaml dosyası içeriği yukarıdaki gibidir.

Spec kısmında 4 seçenek mevcuttur; Bunlardan ilk ikisi parallelism ve completions buradaki gibi aksini belirtmediğimiz sürece varsayılan değeri "1"dir.&#x20;

completions bu job altında, kaç tane başarılı pod oluşturmasını istediğimizi belirttiğimiz kısımdır. Bu örnekte 10 pod çalıştırmak istediğimizi söylüyoruz.

parallelism ise, adından anlaşılacağı üzere, bu 10 pod oluşturma işini, aynı anda kaçar kaçar yapacağını belirttiğimiz kısımdır.

Biz bu job objesini oluşturduğumuz zaman, "template" parametresi altındaki ayarlara göre "2" adet pod oluşturulacak ve bu podlar çalışacak container oluşacak. Container içerisinde uygulama çalışacak. Ve uygulama işini yapıp kapanacak. Eğer işlem hata almadan, bu şekilde olduysa, Kubernetes hemen "2" pod daha oluşturacak ve aynı şekilde onlarda çalışacak ve kapanacak ki 10 pod bu şekilde tamamlanana kadar devam edecek.&#x20;

backOfflimits değerinin  5 olarak ayarlandığını görüyorsunuz. Bu değer, toplam kaç kere hata oluşursa, tüm işlemleri bırakarak, job'un fail etmesini istediğimizi belirttiğimiz kısımdır. Bu, şu demektir.  podları oluşturmaya başla, düzgün bir şekilde devam ederse sıkıntı yok. Ancak podlar fail ederse, 5 defa daha denemeye devam et, toplamda 5 fail alırsan da, artık job'a devam etme, çünkü belli ki bir şeyler yolunda gitmiyor ve job'u fail et.&#x20;

activeDeadlineSeconds değeri ise aynı mantık fakat, bu sefer fail sayısı yerine, saniye belirtiyoruz. Eğer bu görev 100 saniye içerisinde tamamlanamazsa, anlaki sıkıntı var ve job'u fail et diyoruz.

"template" altında belirttiğimiz değerler ise, bildiğimiz pod oluştururken kullandığımız argümanlardır.

Şu anda bu job ile, pi sayısının ilk "2000" rakamını listeleyen ve ardından da kapanan podlar yaratıyoruz. Bu örnekte toplam 10 pod oluşturulacak ve her pod pi sayısının ilk 2000 rakamını çıkaran bir uygulama çalıştıracak. Uygulama düzgün bir şekilde bu işlemi yaparsa, uygulama kapanacak, dolayısıyla pod da düzgün şekilde tamamlandı olarak işaretlenecek.

Son olarak da, restart policy parametresinin mantıken "never" veya "on failure" olarak set edilmesi gerekmektedir.

Job oluşturulduktan sonra, podlar template parametresi altındaki belirttiğimiz görevleri yapıp, tamamlandı olarak işaretleniyor. Podlar işini bitirince silinmez. Böylelikle podların oluşturduğu logları gözlemleyebiliriz.

### Cronjob

Bir cronjob objesi, bir crontab dosyasının bir satırı gibidir. Belirli bir zamanlamaya göre, cron talimatında yazılmış bir job'u periyodik olarak çalıştırır.

Bir job objesini manuel olarak başlatmak yerine, örneğin istediğim zamanlarda tekrarlayan, istediğim günlerde çalışacak şekilde schedule etmek istersek, cronjob kullanabiliriz. Cronjob objesi, Linux crontab 'ın Kubernetes halidir. Syntax olarak da aynıdır.&#x20;

Kubernetes Job'dan farklı olarak, schedule parametresi mevcuttur. Bu parametreye göre job'un ne zamanlar çalışmasını istediğimizi belirtebiliyoruz. Misal aşağıdaki örnekte, Her dakika da bir çalışacak ve bir pod oluşturacak.

```yaml
apiVersion: batch/v1beta1 # not stable until kubernetes 1.21.
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

