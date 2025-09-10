---
icon: j
---

# Introduction to Jenkins

**Jenkins Nedir?**

Yazılım geliştirme sürecini bir yemek yapmaya benzetelim. Bir yemeği yaparken sürekli tekrar eden adımlar vardır: malzemeleri doğra, ocağı yak, tencereyi koy, pişir, servis et... Bu adımları her seferinde elle yapmak hem yorucudur hem de hata yapma olasılığını artırır (örneğin tuzu fazla kaçırmak gibi).

İşte Jenkins, yazılım dünyasının "şef robotu" gibidir. Yazılımcıların sürekli tekrar ettiği şu işleri otomatik olarak yapar:

* Build: Yazılan kod parçalarını bir araya getirip çalışır bir program haline getirme.
* Test: Programın doğru çalışıp çalışmadığını kontrol eden testleri otomatik çalıştırma.
* Deploy: Testlerden geçen programı sunuculara (kullanıcıların erişebileceği yerlere) otomatik olarak yükleme.

Bu sürece CI/CD (Continuous Integration / Continuous Deployment ) diyoruz. Jenkins de bu sürecin en popüler otomasyon aracıdır. Amacı, süreci hızlandırmak, insan hatalarını azaltmak ve yazılımcıların sadece kod yazmaya odaklanmasını sağlamaktır.

**Jenkins Pratikte Nasıl Çalışır?**

Senaryo: Bir yazılımcı, projenin kodunda bir değişiklik yaptı.

1. Commit (Kodu Gönderme): Yazılımcı, yazdığı yeni kodu veya yaptığı değişikliği Git gibi bir versiyon kontrol sistemine gönderir ("commit" eder). Burası kodun merkezi deposudur.
2. Jenkins Tetiklenir: Jenkins, bu Git deposunu sürekli dinler. Yeni bir kod geldiğini fark ettiği anda otomatik olarak devreye girer.
3. İşlem Başlar (Pipeline): Jenkins, önceden tanımlanmış görevleri sırasıyla yapmaya başlar:
   * Clone Repo: Git'teki en güncel kodu kendi çalıştığı sunucuya indirir.
   * Build/Compile: İndirdiği bu kodları derleyerek bir araya getirir. Bu aşamada kodda temel bir yazım hatası (syntax error) var mı diye kontrol edilir.

Şimdi iki farklı sonuç olabilir:

<figure><img src="../.gitbook/assets/Screenshot 2025-09-10 at 11.26.22.png" alt=""><figcaption></figcaption></figure>

* Başarısız Senaryo:
  * Eğer `Build/Compile` aşamasında bir hata çıkarsa, Jenkins süreci anında durdurur.
  * Hemen ilgili yazılımcıya "Kodun derlenirken hata verdi, lütfen düzelt!" diye bir bildirim (e-posta, Slack mesajı vb.) gönderir. Bu sayede hata çok hızlı fark edilir ve düzeltilir.
* Başarılı Senaryo:
  * `Build/Compile` aşaması sorunsuz geçerse, Jenkins bir sonraki adıma geçer.
  * Unit Test: Kodun mantıksal olarak doğru çalışıp çalışmadığını kontrol eden birim testlerini çalıştırır.
  * Docker: Testler de başarılı olursa, uygulamayı bir Docker imajı haline getirir. Yani, uygulamanın çalışması için gereken her şeyi tek bir pakete koyar.
  * K8S Dev Deploy: Bu Docker paketini, testlerin yapılabileceği bir geliştirme ortamına (örneğin Kubernetes - K8S) otomatik olarak dağıtır.
  * Success: Tüm adımlar sorunsuz tamamlandığında, Jenkins ekibe "Yeni versiyon başarıyla geliştirme ortamına yüklendi!" diye bir bildirim gönderir.

Gördüğün gibi, yazılımcının tek yaptığı kodu Git'e göndermekti. Geri kalan tüm süreci Jenkins güvenilir ve hızlı bir şekilde yürüttü.

**Jenkins'in Temel Kavramları:**

Bu süreci yönetirken Jenkins'in kullandığı bazı temel yapı taşları var. Bunları bilmek çok önemli:

* Jobs : Görevin tamamıdır. Bir projenin başından sonuna kadar otomatize edilen sürecin bütünüdür.
* Builds : Bir Job'un her çalıştırılmasına "Build" denir. Jenkins her çalıştırmanın kaydını (başarılı mı, başarısız mı, ne kadar sürdü vb.) tutar. Bu, sorunları geriye dönük incelemek için çok faydalıdır.
* Freestyle Project: Jenkins'te bir "Job" oluşturmanın en temel ve esnek yoludur. Web arayüzü üzerinden tıklayarak komutlarınızı ve adımlarınızı kolayca tanımlayabilirsiniz. Başlangıç için harikadır.
* Pipelines : Birden fazla Job'ı birbirine zincirleyerek oluşturulan bütün bir süreçtir. Yukarıda anlattığımız başarılı senaryo (`Clone -> Build -> Test -> Deploy`) bir Pipeline'dır. Pipeline'lar genellikle `Jenkinsfile` adında bir kod dosyası ile tanımlanır. Bu sayede tüm iş akışını kod olarak yönetebilirsin.
* Stages : Bir Pipeline içindeki mantıksal adımlardır. Görselleştirmeyi kolaylaştırır. Örneğin, "Build" bir stage, "Test" bir stage ve "Deploy" başka bir stage olabilir. Bu sayede pipeline'ın hangi aşamada olduğunu kolayca görebilirsin.
* Nodes : Jenkins'in işleri (Jobs) çalıştırdığı makinelerdir. Tek bir makine (Jenkins Controller) üzerinde çalışabileceği gibi, yükü dağıtmak için birden fazla "Worker Node" (işçi makine) de eklenebilir. Bu, Jenkins'in ölçeklenmesini sağlar.
* Plugins : Jenkins'in en güçlü yanıdır. Tıpkı telefonuna uygulama yükleyerek yeni özellikler eklemen gibi, Jenkins'e de eklentiler (plugin) kurarak onun yeteneklerini artırırsın. Git ile konuşmasını sağlayan bir plugin, Docker ile konuşmasını sağlayan başka bir plugin, Kubernetes'e dağıtım yapmasını sağlayan farklı bir plugin vardır. Neredeyse her teknoloji için bir Jenkins eklentisi mevcuttur.

***

* Ne Öğrendik? Jenkins'in, yazılım geliştirme sürecindeki derleme, test ve dağıtım gibi tekrar eden görevleri otomatikleştiren bir araç (CI/CD aracı) olduğunu öğrendik.
* Nasıl Çalışır? Bir yazılımcı kodunu gönderdiğinde tetiklenir ve önceden tanımlanmış bir iş akışını (pipeline) adım adım (stages) çalıştırır.
* Neden Önemli? Hataları erken tespit etmeyi sağlar, süreci hızlandırır ve insan kaynaklı hataları ortadan kaldırır.

