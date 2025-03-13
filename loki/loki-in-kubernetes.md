---
icon: square-ring
---

# Loki in Kubernetes

<figure><img src="../.gitbook/assets/Screenshot 2025-03-13 at 16.18.03.png" alt=""><figcaption></figcaption></figure>

### 1. Kubernetes Ortamında Log Toplama İhtiyacı

Kubernetes kümesinde çalışan her pod veya container, kendi loglarını tutar. Normalde, bu log’ları merkezi bir yerde depolamak ve ihtiyaç olduğunda sorgulayabilmek için bir log toplama aracına ihtiyaç duyarsınız. İşte **Loki** bu noktada devreye girer:

1. **Loki**: Log verisini saklayan ve sorgulamaya olanak tanıyan bileşen.
2. **Promtail**: Kubernetes node’larında çalışan ajan (agent) olarak log’ları alır ve Loki’ye iletir.
3. **Grafana**: Loki’ye kaydedilen log’ları görsel arayüzde (dashboard) izlemek ve sorgulamak için kullanılır.

### 2. Loki ve Grafana’yı Kubernetes’te Konumlandırma

* Loki, **Kubernetes kümesi içinde** veya **harici** bir sunucuda çalışabilir.
* Grafana da aynı şekilde Kubernetes içinde çalışabilir veya harici bir sunucuda çalıştırılabilir.

Ancak, zaten Kubernetes kullanıyorsanız çoğunlukla Loki ve Grafana’yı da küme içerisinde deploy etmek tercih edilir. Böylece bakım, ölçeklendirme ve konfigürasyon tek bir çatı (Kubernetes) altında yönetilir.

### 3. Promtail ve DaemonSet: Her Node’da Log Toplamak

Kubernetes’te birden fazla node bulunur ve üzerinde birçok container/pod çalışır. Log’ları tek tek almak yerine, **Promtail** bu işi üstlenir:

1. **Promtail**, node’larda çalışan pod’ların ürettiği log’ları okur (örneğin `/var/log/containers` veya benzeri klasörlerde).
2. **DaemonSet** olarak tanımlandığında, **her Kubernetes node’una** otomatik olarak bir Promtail pod’u dağıtılır. Yeni bir node eklendiğinde, orada da Promtail pod’u ayağa kalkar.
3. Promtail, topladığı log’ları **Loki’ye** push eder.

Böylece log verileri merkezîleştirilir ve istediğiniz zaman Grafana üzerinden görüntülenebilir.

### 4. Helm Chart ile Kolay Kurulum

Hem Loki hem Promtail hem de Grafana için konfigürasyon dosyalarını elle yazmak zaman alıcı olabilir. Bu noktada, Kubernetes dünyasındaki **Helm** aracı devreye girer:

* **Helm**: Kubernetes uygulamalarını paketleyip yönetmeyi kolaylaştıran bir araçtır.
* **Loki Helm Chart**: Loki, Promtail ve Grafana’yı tek komutla veya çok az konfigürasyonla Kubernetes’e kurmanıza imkân tanır.
  * Birkaç parametreyi (`values.yaml`) ayarlayarak, tüm bileşenleri ayağa kaldırabilirsiniz.
  * Helm Chart, arka planda Deployment/DaemonSet/Service gibi temel Kubernetes manifestlerini sizin yerinize oluşturur.

### 5. Özet

* **Loki in Kubernetes** yaklaşımı, uygulama log’larını merkezi bir yerde toplamak için idealdir.
* **Promtail** her node’daki log’ları **DaemonSet** üzerinden toplayıp, **Loki**’ye iletir.
* **Grafana** ile bu log’lar kullanıcı dostu bir arayüzde sorgulanır ve görselleştirilir.
* **Helm** kullanarak, Loki/Grafana/Promtail’i hızla ve kolayca **Kubernetes** ortamına kurabilirsiniz.
