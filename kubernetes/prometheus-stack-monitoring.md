# 📽 Prometheus Stack - Monitoring

<div>

<figure><img src="../.gitbook/assets/Ekran görüntüsü 2022-10-06 175945.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../.gitbook/assets/Ekran görüntüsü 2022-10-06 175931.png" alt=""><figcaption></figcaption></figure>

</div>

Production ortamlarda, kubernetes cluster durumunu ve benzeri şeyleri kontrol etmemiz ve olası sorunlarda alertler oluşturmamız önemli bir konudur. Monitoring konusunu aşağıdaki maddelere ayırabiliriz;

\=> Kubernetes Cluster'ın durumunu gözlemlememiz gerekiyor. Objelerin durumu, deploy edilen obje miktarını vb. ( Bunu promethues ile monitor edeceğiz )

\=> Objelerin mevcut durumu nedir? İstenilen şekilde çalışıyor mu? Podlar ne kadarlık bir kaynak tüketiyor gibi soruların cevaplarını öğrenmemiz gerekiyor. ( Bunu prometheus ile monitor edeceğiz )

\=> Worker, Manager nodeları izlememiz ve anlık kaynak kullanımlarını gözlemlememiz gerekiyor. ( Bunu prometheus ile monitor edeceğiz )

\=> Pod içerisinde çalışan uygulamalarımıza ait logları incelememiz gerekiyor. ( Prometheus ile yapabiliriz fakat, bunu daha efektif bir şekilde arama yapabileceğimiz EFK stack ile monitor edeceğiz, bir sonraki bölümde )

Prometheus 'a geçmeden, kubectl ile bilgi toplama;

```bash
kubectl get pods 
# çalışan podların listesini getirir.

kubectl get All -A 
# Sistemde bulunan tüm objelerin listesini getirir.

kubectl describe pods pod1 
# Pod1 hakkında detaylı bilgi alırız.

kubectl get events
# Cluster da oluşan tüm logları gösterir.

kubectl top node node1 
# Node1 isimli node hakkında kaynak tüketim miktarını getirir.

kubectl top pods pod1
# pod1 isimli podun kaynak tüketim miktarını getirir.

kubectl logs pod1
# pod1 isimli podun içerisinde çalışan uygulamanın loglarını getirir.
```

Yukarıdaki komutlar ile gerekli tüm bilgilere/loglara ulaşabiliriz fakat, production ortamlarda bu komutlar ile tek, tek toplamak uğraştırıcı ve zaman kaybı olacaktır ve yönetilemez hale gelecektir.

Misal çok fazla objenin bulunduğu Kubernetes cluster ortamlarında, bu şekilde kubectl ile manuel olarak log toplamaya çalışmak tavsiye etmeyeceğimiz bir davranış olacaktır.

Bunun yerine merkezi bir loglama sunucusu veya merkezi bir metric sunucusuna, kubernetes cluster'dan edindiğimiz loglar ve metricleri göndermemiz daha mantıklı bir çözüm olacaktır.  Böylelikle, edindiğimiz loglardan ve metric'leri görselleştirip, alertler oluşturabiliriz. Misal, node1 down olursa, mail gönder vb. gibi alertler oluşturabiliriz.

&#x20;Kısacası, Kubernetes üzerinde monitoring altyapısı sağlayabilecek bir servise ihtiyacımız var. Bu ihtiyacımızı Prometeus stack ile çözeceğiz. Prometheus stack ile uygulamala logları hariç, geriye kalan tüm metricleri alabileceğiz. Metricleri prometheus ile alıp, Grafana ile prometheus'u konuşturup, aldığımız metricleri Grafana ile görselleştirebiliriz.

Prometheus stack kurulumu;

```bash
kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kubeprostack --namespace monitoring prometheus-community/kube-prometheus-stack
kubectl --namespace monitoring get pods -l "release=kubeprostack"
```

Prometheus, Grafana, Alertmanager 'a dışarıdan erişmek için nodePort dosyası.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
spec:
  selector:
    app.kubernetes.io/name: prometheus
    prometheus: kubeprostack-kube-promethe-prometheus
  type: NodePort
  ports:
  - nodePort: 32079
    port: 9090
    protocol: TCP
    targetPort: 9090
    
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus-grafana-service
  namespace: monitoring
spec:
  selector:
    app.kubernetes.io/instance: kubeprostack
    app.kubernetes.io/name: grafana
  type: NodePort
  ports:
  - nodePort: 32078
    port: 80
    protocol: TCP
    targetPort: 3000
    
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus-alertmanager-service
  namespace: monitoring
spec:
  selector:
    app.kubernetes.io/name: alertmanager
  type: NodePort
  ports:
  - nodePort: 32084
    port: 9093
    protocol: TCP
    targetPort: 9093
```

{% embed url="https://github.com/aytitech/k8sfundamentals/blob/main/monitoring/kube-prometheus-stack.md" %}

Prometheus ile grafana'yı konuşturmak 'dan bahsettik yukarıda, bunu nasıl yapıyoruz? Bunu biz değil, bizim yerimize bu stack'i yöneten kubernetes operator yapıyor.

Kubernetes üzerine prometeus'u kurduk ve kurduktan sonra bir çok ayarın yapılması gerekiyor. Misal, ruleların oluşturulması; gerekli tanımların yapılması vb. Özetle prometheus stack 'i kurup,yapılandırıp,yönetmemiz gerekiyor.  Kubernetes 'de operator dediğimiz bir yöntem mevcut; Bizler operatorler oluşturabiliyoruz ve kubernetes de operatorler oluşturduğumuz zaman, bizim admin olarak yapacağımız tüm manuel ve uzun sürecek işlemleri otomatik hale getirilmesini sağlıyoruz operator ile.&#x20;

Prometheus'u kurarken, yani helm chart'ı sisteme release olarak yükledikten sonra, operator prometheus 'un çalışması için gereken tüm ayarlamaları yapıyor. Tıpkı bir admin gibi, bu stack'in yönetiminden sorumlu oluyor.
