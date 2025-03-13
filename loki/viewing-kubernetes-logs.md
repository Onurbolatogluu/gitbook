---
icon: binoculars
---

# Viewing Kubernetes logs

**Viewing Kubernetes Logs** başlığı altında, Kubernetes üzerinde Loki ve Promtail kurulumunu tamamladıktan sonra log’ları **Grafana** üzerinden nasıl görüntüleyebileceğinizi anlatacağım.

### 1. Grafana Explore Ekranı ile Log Sorgulama

1. **Explore’a Erişim**
   * Grafana arayüzünde sol menüdeki **“Explore”** sekmesine giderek veri kaynağını (Loki) seçin.
2. **Label Filtreleri**
   * `pod`, `app`, `namespace`, `node_name` gibi etiketlere göre log sorgusu yapabilirsiniz.
   * Örneğin, `pod="etcd-controlplane"` etiketiyle filtreleyerek ilgili pod’un ürettiği log’ları listeleyebilirsiniz.
3. **Log’ların İncelenmesi**
   * Zaman dilimini “Last 1 hour” veya farklı bir aralığa getirerek anlık veya geçmiş log’ları görüntüleyebilirsiniz.
   * **“Run Query”** butonuna tıkladığınızda, etiketlerle eşleşen log satırları aşağıda listelenir.

<figure><img src="../.gitbook/assets/Screenshot 2025-03-14 at 00.15.15.png" alt=""><figcaption></figcaption></figure>

Bu sayede Kubernetes kümesinde çalışan herhangi bir pod’un ya da uygulamanın log’larına hızla ulaşabilirsiniz.

### 2. Varsayılan Promtail Konfigürasyonu

Helm chart (örneğin `grafana/loki-stack`) kullanarak **Promtail** kurduğunuzda, Promtail **DaemonSet** olarak atanır ve her Kubernetes node’unda çalışıp log toplar. Bu varsayılan kurulum:

1. **Kubernetes Service Discovery**
   * `promtail.yaml` içerisinde yer alan `kubernetes_sd_configs` bölümü, tüm pod’ları otomatik olarak keşfeder.
2. **Varsayılan Etiketleme (Relabeling)**
   * `relabel_configs` sayesinde pod ismi, namespace, container adı gibi bilgileri **label** olarak loglara ekler.
3. **Varsayılan Dizinler**
   * Kubernetes’in genellikle `/var/log/pods` veya `/var/log/containers` altında tuttuğu container loglarını otomatik yakalar.

Özellikle `scrape_configs` bölümü, Promtail’in hangi yolları tarayacağı ve hangi etiketleri ekleyeceğini belirler.

### 3. Promtail Konfigürasyonunu Görüntüleme

<figure><img src="../.gitbook/assets/Screenshot 2025-03-14 at 00.59.18.png" alt=""><figcaption></figcaption></figure>

1. **Pod Ayrıntıları**
   * `kubectl describe pod <promtail-pod-ismi>` komutuyla Promtail pod’unun kullandığı volume ve mount noktalarını görebilirsiniz (örn. `/etc/promtail`).
   * Bu volumelerin büyük kısmı, Kubernetes node’undaki log dizinlerini, konfigürasyon dosyalarını vs. barındırır.
2. **Secret İçerisine Gömülü Config**
   * Helm chart, Promtail’in `promtail.yaml` dosyasını bir **Secret** (örn. `loki-promtail`) içinde base64 olarak saklar.
   * `kubectl get secret loki-promtail -o jsonpath="{.data.promtail\.yaml}" | base64 --decode`komutuyla bu dosyanın içeriğini düz metin olarak görebilirsiniz.

Bu `promtail.yaml` içeriğinde, `scrape_configs`’in **kubernetes-pods** job’uyla otomatik discovery yapıldığını ve relabeling kurallarının tanımlı olduğunu göreceksiniz.

***

`loki-stack` chart’ını kurarken `values.yaml` dosyasında sadece `promtail.enabled: true` ayarını yapıp promtail ile alakalı geri kalan her şeyi varsayılan (default) bırakırsanız, **helm chart’ın kendi içindeki** (yani paketlenmiş varsayılan) `promtail.yaml`devreye girer. Bu varsayılan konfigürasyon:

* Kubernetes node’larında **DaemonSet** olarak Promtail’i çalıştırır,
* **kubernetes\_sd\_configs** ile tüm pod’ların loglarını otomatik keşfeder,
* Log’ları etiketlendirmek (relabel) ve Loki’ye göndermek için temel ayarları içerir.



