# 🐐 Kube Sheets

<figure><img src="../.gitbook/assets/electronics-kubernetes-ukraine-l-viv.jpeg" alt=""><figcaption></figcaption></figure>

#### Kubernetes'in (`kubectl`) yaygın kullanılan komutlarından bazıları şunlardır:

| **Komut Adı**                    | **Açıklama**                                                | **Kullanım Örneği**                                                                                   |
| -------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `kubectl get`                    | Kaynakları listeler.                                        | `kubectl get pods`                                                                                    |
| `kubectl describe`               | Kaynak detaylarını gösterir.                                | `kubectl describe pod <pod-name>`                                                                     |
| `kubectl create`                 | Yeni bir kaynak oluşturur.                                  | `kubectl create -f <file.yaml>`                                                                       |
| `kubectl delete`                 | Bir kaynağı siler.                                          | `kubectl delete pod <pod-name>`                                                                       |
| `kubectl logs`                   | Bir pod'daki konteynerlerin loglarını gösterir.             | `kubectl logs <pod-name>`                                                                             |
| `kubectl exec`                   | Bir pod'daki konteynerde komut çalıştırır.                  | `kubectl exec <pod-name> -- ls /`                                                                     |
| `kubectl apply`                  | Kaynak değişikliklerini uygular.                            | `kubectl apply -f <file.yaml>`                                                                        |
| `kubectl port-forward`           | Bir pod'a lokal port üzerinden erişim sağlar.               | `kubectl port-forward <pod-name> 8080:80`                                                             |
| `kubectl run`                    | Belirtilen imajı kullanarak pod çalıştırır.                 | `kubectl run nginx --image=nginx`                                                                     |
| `kubectl expose`                 | Pod'u ağ üzerinde erişilebilir yapar.                       | `kubectl expose pod nginx --port=80`                                                                  |
| `kubectl scale`                  | Replica sayısını değiştirir.                                | `kubectl scale deployment nginx --replicas=3`                                                         |
| `kubectl rollout`                | Deployment güncellemelerini yönetir.                        | `kubectl rollout status deployment/nginx`                                                             |
| `kubectl set`                    | Kaynaklarda özellikleri günceller.                          | `kubectl set image deploy/nginx nginx=nginx:1.9.1`                                                    |
| `kubectl label`                  | Kaynaklara etiket ekler veya günceller.                     | `kubectl label pods <pod-name> env=prod`                                                              |
| `kubectl annotate`               | Kaynaklara not ekler veya günceller.                        | `kubectl annotate pod <pod-name> note="my note"`                                                      |
| `kubectl config`                 | Kubeconfig ayarlarıyla oynar.                               | `kubectl config view`                                                                                 |
| `kubectl cluster-info`           | Cluster hakkında temel bilgileri verir.                     | `kubectl cluster-info`                                                                                |
| `kubectl top`                    | Kaynakların kaynak kullanımını gösterir.                    | `kubectl top nodes`                                                                                   |
| `kubectl cordon`                 | Bir node'u planlama dışı bırakır.                           | `kubectl cordon <node-name>`                                                                          |
| `kubectl uncordon`               | Bir node'un planlama dışı olma durumunu kaldırır.           | `kubectl uncordon <node-name>`                                                                        |
| `kubectl drain`                  | Bir node'da çalışan pod'ları boşaltır.                      | `kubectl drain <node-name>`                                                                           |
| `kubectl taint`                  | Bir node'a taint ekler veya kaldırır.                       | `kubectl taint nodes <node-name> key=value:NoSchedule`                                                |
| `kubectl cp`                     | Pod ile yerel sistem arasında dosya kopyalar.               | `kubectl cp <pod-name>:/path/to/remote /path/to/local`                                                |
| `kubectl auth`                   | Yetkilendirme ile ilgili işlemler yapar.                    | `kubectl auth can-i list pods`                                                                        |
| `kubectl version`                | Klient ve sunucu sürüm bilgisi verir.                       | `kubectl version`                                                                                     |
| `kubectl proxy`                  | Kubernetes API sunucusuna bir proxy oluşturur.              | `kubectl proxy`                                                                                       |
| `kubectl diff`                   | Canlı obje ve bir dosyadaki obje arasındaki farkı gösterir. | `kubectl diff -f <file.yaml>`                                                                         |
| `kubectl patch`                  | Kaynaklarda değişiklik yapar.                               | `kubectl patch pod <pod-name> -p '{"spec": {"containers":[{"name":"nginx","image":"nginx:1.9.1"}]}}'` |
| `kubectl replace`                | Kaynakları değiştirir.                                      | `kubectl replace -f <file.yaml>`                                                                      |
| `kubectl wait`                   | Kaynakların belirli bir duruma gelmesini bekler.            | `kubectl wait --for=condition=complete job/myjob`                                                     |
| `kubectl attach`                 | Bir konteynere bağlanır.                                    | `kubectl attach <pod-name> -i`                                                                        |
| `kubectl edit`                   | Kaynakları düzenler.                                        | `kubectl edit pod <pod-name>`                                                                         |
| `kubectl api-resources`          | Kullanılabilir API kaynaklarını listeler.                   | `kubectl api-resources`                                                                               |
| `kubectl api-versions`           | Desteklenen API sürümlerini listeler.                       | `kubectl api-versions`                                                                                |
| `kubectl completion`             | Otomatik tamamlama için script üretir.                      | `source <(kubectl completion bash)`                                                                   |
| `kubectl config get-contexts`    | Mevcut konteksleri listeler.                                | `kubectl config get-contexts`                                                                         |
| `kubectl config use-context`     | Belirtilen konteksi kullanır.                               | `kubectl config use-context my-cluster`                                                               |
| `kubectl config set-context`     | Bir konteksi ayarlar veya günceller.                        | `kubectl config set-context my-cluster --namespace=myns`                                              |
| `kubectl config current-context` | Aktif konteksi gösterir.                                    | `kubectl config current-context`                                                                      |
| `kubectl plugin`                 | Kubectl eklentileriyle çalışır.                             | `kubectl plugin list`                                                                                 |
| `kubectl certificate`            | Sertifika talepleriyle çalışır.                             | `kubectl certificate approve mycert`                                                                  |

Bu, Kubernetes'de (`kubectl` ile) kullanılan yaygın komutlardan sadece bir bölümüdür. Kubernetes çok geniş kapsamlı bir platform olduğu için bu listeyi genişletebilir veya belirli bir kullanım senaryosu için daha spesifik komutları araştırabilirsiniz.

