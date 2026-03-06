---
icon: layer-minus
---

# Demo

Bir sabah uyandınız ve sisteminizdeki uygulamaların birbiriyle konuşamadığını, dışarıya istek atamadığını veya müşterilerin servislere erişemediğini fark ettiniz. Hata mesajları yetersiz ve her şey birbirine girmiş durumda.

Panik yapmak yerine, Kubernetes ağ mimarisini katman katman test ederek sorunu izole edeceğiz. Bu senaryoda ağ eklentisi olarak Cilium kullandığımızı varsayarak temelden (altyapıdan) uygulamaya doğru bir hata ayıklama (troubleshooting) süreci işleteceğiz.

### 1. Altyapı Kontrolü: CNI (Ağ Eklentisi) Hayatta mı?

Uygulamalara bakmadan önce, "Kablolar takılı mı?" sorusunun cevabını vermeliyiz. Cilium gibi ağ eklentileri `kube-system` namespace'inde çalışır.

Adım 1: Pod'ların Durumuna Bakmak

Ağ eklentimizin bileşenleri ayakta mı, yoksa sürekli yeniden mi başlıyor (CrashLoopBackOff)?

```bash
kubectl get pods -n kube-system
```

Örnek Çıktı:

```bash
NAME                               READY   STATUS    RESTARTS   AGE
cilium-6xfl8                       1/1     Running   0          3m
cilium-operator-58684c48c9-4rntb   1/1     Running   0          3m
```

_Her şey `Running` durumunda. Eğer burada bir hata görseydik, `kubectl logs -n kube-system cilium-6xfl8` komutuyla doğrudan logların içine girecektik._

Adım 2: Cilium CLI ile Derinlemesine Sağlık Taraması

Eğer Cilium CLI aracınız yüklüyse, tek bir komutla tüm ağ alt yapısının röntgenini çekebilirsiniz:

```bash
cilium status
```

```bash
Cilium:                OK
Operator:              OK
DaemonSet cilium:      Desired: 2, Ready: 2/2, Available: 2/2
```

_Altyapımız sapasağlam! Sorun tesisatta değil, bir üst katmana geçebiliriz._

***

### 2. Görünmez Duvarlar: Network Policy Kontrolü

Altyapı çalışıyor ama Pod'umuz internete çıkamıyor veya başka bir servise bağlanamıyorsa, sorunun kaynağı genellikle sessizce trafiği düşüren (timeout verdiren) Network Policies'dir.

Adım 1: Kuralları Listelemek

Ortamımızda trafiği engelleyen bir kural olup olmadığına bakalım:

```bash
kubectl get networkpolicies -A

# ÇIKTI:
# NAMESPACE   NAME                  POD-SELECTOR   AGE
# default     default-deny-egress   <none>         7m
```

Listede şüpheli bir kural gördük: `default-deny-egress`.

Adım 2: Kuralı İncelemek ve Test Etmek

Bu kuralın ne yaptığını anlamak için içine bakalım:

```bash
kubectl describe networkpolicy default-deny-egress
# Çıktı bize bu kuralın "Egress" (dışarı giden trafik) için olduğunu ve hiçbir şeye izin vermediğini söyler.
```

Durumu fiziksel olarak kanıtlamak için geçici bir "test podu" ayağa kaldırıp Google'a istek atmayı deneyelim:

```bash
kubectl run --rm -i --tty debug --image=curlimages/curl --restart=Never -- curl www.google.com --connect-timeout 2
```

_Sonuç: Komut donup kalır (Timeout). Dışarı çıkışımız kesin olarak yasaklanmış._

Çözüm: Sorunlu güvenlik duvarı kuralını silerek (veya düzelterek) trafiği açalım:

```bash
kubectl delete networkpolicy default-deny-egress
```

_Kuralı sildikten sonra curl komutunu tekrar çalıştırdığınızda Google'dan anında yanıt aldığınızı göreceksiniz._

***

### 3. Yanlış Adres, Kör Servis: Service ve Pod Bağlantısı

İç ağ çalışıyor, güvenlik duvarı açık ama dışarıdaki kullanıcı `nginx-service` üzerinden sitemize giremiyor. Sorun nerede? Service'te mi yoksa Pod'da mı?

Adım 1: Sorunu İzole Etmek (Port-Forward Hayat Kurtarır)

Öncelikle uygulamanın (Pod'un) gerçekten çalışıp çalışmadığını anlamak için aradaki `Service` nesnesini bypass edip, kendi bilgisayarımızdan doğrudan Pod'a tünel açalım:

```bash
kubectl port-forward pod/nginx-deployment-56fcf95486-7d2dw 8080:80
```

Tarayıcınızdan `http://localhost:8080` adresine gittiğinizde web siteniz açılıyorsa; tebrikler, uygulamanızda (Pod'da) hiçbir sorun yok! Suçlu tamamen `Service` nesnesindedir.

Adım 2: Service'in Adres Defterini (Endpoints) Kontrol Etmek

Bir önceki yazımızda öğrendiğimiz o meşhur rehbere bakalım. Acaba servisimiz arkadaki Pod'un IP'sini bulabilmiş mi?

```bash
kubectl describe svc nginx-service
```

Kritik Çıktı:

```bash
Selector:  app=nginx-website
Endpoints: <none>
```

İşte sorun tam karşımızda duruyor! Servisin adres defteri bomboş (`<none>`). Kubernetes, servisteki etiketler ile Pod'daki etiketleri eşleştirememiş.

Adım 3: Etiketleri (Labels) Düzeltmek

Hemen Pod'umuzun gerçek etiketine bakalım:

```bash
kubectl get pod nginx-deployment-56fcf95486-7d2dw -o=jsonpath='{.metadata.labels}'
# ÇIKTI: {"app":"nginx"}
```

Pod'un etiketi `app=nginx`, ama servisimiz `app=nginx-website` etiketini arıyor. Ufak bir yazım hatası bütün sistemi çökertmiş!

Servisi düzenleyip hatayı düzeltelim:

```bash
kubectl edit svc nginx-service
# Açılan dosyada "app: nginx-website" yazan yeri "app: nginx" olarak değiştirip kaydedin.
```

Mutlu Son:

Tekrar kontrol ettiğimizde adres defterinin dolduğunu göreceğiz:

```bash
kubectl describe svc nginx-service
# Endpoints: 10.0.0.247:80
```

Bağlantı sağlandı ve kriz çözüldü!
