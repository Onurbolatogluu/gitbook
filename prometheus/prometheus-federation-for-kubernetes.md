---
description: Harici Prometheus ile kubernetes cluster'ı monitör etmek.
---

# 🤖 Prometheus Federation For Kubernetes

#### Kubernetes cluster'a prometheus kurulumu:

1. Kube-Prometheus-Stack kullanma:
   * Kube-Prometheus-Stack, Kubernetes kümenizde tam bir Prometheus yığını kurmanın en basit yoludur.
   * Prometheus, Kubernetes ortamınızdan ve uygulamalarından metrikleri toplayan, depolayan ve ortaya çıkaran bir zaman serisi veritabanıdır.
   * Node-Exporter, Kubernetes kümenizdeki düğümlerden kaynak kullanımı verilerini toplayan bir Prometheus dışa aktarıcısıdır.
   * Kube-State-Metrics, Kubernetes kümenizdeki API nesneleri hakkında (Pod'lar, konteynerler vb.) bilgi sağlayan başka bir dışa aktarıcıdır.
   * Grafana, Prometheus veritabanları dahil olmak üzere birçok veri kaynağıyla çalışan bir gözlem platformudur.
   * Alertmanager, metrikler değiştikçe bildirimler sağlayan bağımsız bir Prometheus bileşenidir.
2. Kube-Prometheus-Stack'in kurulumu:
   *   Helm istemcinizde kılavuza aşağıdaki komutu kullanarak kube-prometheus-stack deposunu kaydedin:

       ```bash
       helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
       ```
   *   Ardından, kılavuzunuzu keşfetmek için repo listelerinizi güncelleyin:

       ```
       helm repo update
       ```
   *   Şimdi, aşağıdaki komutu kullanarak grafikleri bir yeni ad alanında kümenize dağıtmak için aşağıdaki komutu çalıştırabilirsiniz:

       ```bash
       helm install kube-prometheus-stack-1   --create-namespace   --namespace kube-prometheus-stack   prometheus-community/kube-prometheus-stack
       ```
3. Kurulumun kontrol edilmesi:
   *   Aşağıdaki komutu kullanarak bileşenlerin durumunu kontrol edin:

       ```bash
       kubectl --namespace kube-prometheus-stack get pods
       ```
   * Tüm Pod'lar "Running" olarak görüntülendikten sonra, izleme yığını kullanıma hazırdır.
4.  Prometheus Sorgusu Çalıştırma:

    * Prometheus, verilerinizi sorgulamak için kullanabileceğiniz bir web arayüzü içerir.
    *   Arayüze erişmek için aşağıdaki komutu kullanarak yerel trafiği kümenizdeki hizmete yönlendirmek için Kubectl port forwarding kullanabilirsiniz:

        ```bash
        kubectl port-forward -n kube-prometheus-stack svc/kube-prometheus-stack-1-prometheus 9090:9090 --address=0.0.0.0 &
        ```
    * Tarayıcınızda nodeip:9090 adresini ziyaret ederek Prometheus UI'sine erişebilirsiniz.



<figure><img src="../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

#### Prometheus Federation Yapılandırma:

\
Prometheus Federation, birden fazla Prometheus sunucusunu bir araya getirerek merkezi bir görünürlük ve ölçeklenebilirlik sağlayan bir mekanizmadır. Birçok Prometheus sunucusu, ayrı ayrı topladıkları metrikleri federasyon kullanarak merkezi bir sunucuya gönderebilir. Bu sayede, birden fazla küme veya ortamda bulunan Prometheus sunucuları, tek bir yerden izlenebilir ve yönetilebilir.

Federasyon, Prometheus'un sunduğu metrik verilerini toplamak, depolamak ve sorgulamak için kullanılan HTTP tabanlı bir protokolü kullanır. Bu protokol aracılığıyla, kaynak Prometheus sunucuları birbirleriyle etkileşimde bulunurken hedef Prometheus sunucusu da verileri toplayarak tek bir görünüm sağlar.



{% code title="/etc/prometheus/prometheus.yml" %}
```yaml
  - job_name: 'advanced-federation'
    scrape_interval: 20s
    scrape_timeout: 20s
    scheme: http
    metrics_path: /federate
    honor_labels: true
    metric_relabel_configs:
      - source_labels: [id]
        regex: '^static-agent$'
        action: drop
    params:
      match[]:
        - '{__name__=~"kube_.*|node_.*|container_.*"}'
    static_configs:
      - targets: ['10.90.0.170:9090'] # External kubernetes cluster

```
{% endcode %}

1. `scrape_interval` ve `scrape_timeout`: Hedeften veri toplama sıklığını ve zaman aşımını belirtir. Bu örnekte, her 20 saniyede bir scrape işlemi gerçekleştirilecek ve 20 saniye içinde tamamlanması gerektiği belirtilmiştir.
2. `scheme` ve `metrics_path`: Toplama işlemi için kullanılacak protokol ve URL yolu. Bu örnekte, HTTP protokolü kullanılır ve `/federate` yolu belirtilmiştir.
3. `honor_labels`: Kaynak hedefin etiketlerini korumak için kullanılır. Bu örnekte, etiketlerin korunması istenmektedir.
4. `metric_relabel_configs`: Toplanan metriklerin etiketlerini yeniden düzenlemek için kullanılır. Bu örnekte, "id" etiketi "static-agent" olan metriklerin atılması belirtilmiştir.
5. `params`: Toplama işlemi için ek parametrelerin belirtilmesine olanak sağlar. Bu örnekte, `{__name__=~"kube_.*|node_.*|container_.*"}` ifadesiyle eşleşen metriklerin toplanacağı belirtilmiştir. Burada, "kube\_", "node\_" veya "container\_" ile başlayan metrikler toplanacaktır.
6. `static_configs`: Sabit hedeflerin belirtilmesini sağlar. Bu örnekte, "10.90.0.170:9090" IP adresine ve 9090 portuna sahip bir hedef belirtilmiştir. Bu hedeften metrikler toplanacaktır.

Bu Prometheus yapılandırması, "advanced-federation" olarak adlandırılan bir iş tanımı için scrape (toplama) işlemi yapılandırmalarını belirtir.

<figure><img src="../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

<br>



