# 🧊 kube cadvisor with external prometheus

Pre Kubernetes Cluster at Master Node:

* install cadvisor

```bash
kubectl apply -k https://github.com/google/cadvisor/deploy/kubernetes/base
```

* assing env pod name

```bash
pod=`kubectl -n cadvisor get pod -l app=cadvisor -o jsonpath="{.items[0].metadata.name}"`
```

* port-forward cadvisor

```bash
kubectl -n cadvisor port-forward pod/"${pod}" 8080:8080 --address='0.0.0.0' &
```

\-----------------------------------------------------------------------------------------------

\-----------------------------------------------------------------------------------------------

\-----------------------------------------------------------------------------------------------

* İlk adım service account oluşturun.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: testsa
  namespace: default
```

`kubectl apply -f service-account-testsa.yml`

* Service account'u bir clusterrolebinding oluşturup, clusterrole'e atayarak, yetki tanımlamış olalım.

```bash
kubectl create clusterrolebinding cluster_role_binding_name --clusterrole=cluster-admin --serviceaccount=namespace:service_account_name

```

> Example

```bash
kubectl create clusterrolebinding cluster_role_binding_testsa --clusterrole=cluster-admin --serviceaccount=default:testsa
```

* Service account'ına ait, token ve ca.crt dosyalarını almak için, service account'ı bir pod'a tanımlayalım.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: testpod
  namespace: default
spec:
  serviceAccountName: testsa
  containers:
  - name: testcontainer
    image: nginx
    ports:
    - containerPort: 80
```

* Oluşturduğumuz Pod'a shell'den bağlanıp, iglili dizine gidelim.

```bash
kubectl exec -it testpod -- bash
```

> Dizine geçelim.

```bash
cd /var/run/secrets/kubernetes.io/serviceaccount/
```

> Token ve ca.crt dosyalarının içeriklerini alalım.

\-----------------------------------------------------------------------------------------------

\-----------------------------------------------------------------------------------------------

\-----------------------------------------------------------------------------------------------



Prometheus Sunucuya Geçelim;



* ca.crt dosyasını /tmp altına kopyalayalım.
* Prometheus yapılandırma dosyasını güncelleyelim.

```yaml
  - job_name: 'kubernetes-cadvisor'
    scheme: https
    tls_config:
      ca_file: /tmp/ca.crt
    bearer_token: 'eyJhbGciOiJSUzI1NiIsImtpZCI6InZHOHFKTk9WR3FYVEVVYVVaZE8xSl85Wjk3OVJZTnZnZjlaY1JSWHlILUkifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzE2NjI5NzM1LCJpYXQiOjE2ODUwOTM3MzUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJ0ZXN0cG9kIiwidWlkIjoiMDU3ODI0MWYtMTQ0Zi00ZmNhLWJkNGYtZmJmN2U1MmY5MzM0In0sInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJ0ZXN0c2EiLCJ1aWQiOiIwMWU3MWUxOS0zMmQ3LTRhODEtYjM3Yy01NTM4ZmE4M2IwZWEifSwid2FybmFmdGVyIjoxNjg1MDk3MzQyfSwibmJmIjoxNjg1MDkzNzM1LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDp0ZXN0c2EifQ.oCCR5qdnL2bJirN-u8pqt6HKW3vfj-yuUn-zen7h8k1MJDwbFDLkTqtqyPBOae1loLEnxOjTBmjAReiv3xJ0NPJ8sSQ4pMIPVScKgAGEmq9hs5ZD60rEajKFh-THWdZSopdCiGFkZ9r5xPz2dZgoAFEDgOS1FzXBWekyioBG63OnVUsPZ6jJfSagw5cWYoCY8WcQCAu1NSfS2yGO_GRv4zUAIhT4ENtngicPnBw5a3MmHR4Zp7foROQmdW5oGom-2rCsk3IaKuwmUWGh26JOMl93Bx2fH3h8zHkc-KCzJZ9DAeoWxkIXRNtT5yCEN52ix1eoG_N7GRsnaaUvxzv9Tg'

    kubernetes_sd_configs:
      - api_server: 'https://10.90.0.170:6443'
        role: node
        tls_config:
          ca_file: /tmp/ca.crt
        bearer_token: 'eyJhbGciOiJSUzI1NiIsImtpZCI6InZHOHFKTk9WR3FYVEVVYVVaZE8xSl85Wjk3OVJZTnZnZjlaY1JSWHlILUkifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzE2NjI5NzM1LCJpYXQiOjE2ODUwOTM3MzUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJ0ZXN0cG9kIiwidWlkIjoiMDU3ODI0MWYtMTQ0Zi00ZmNhLWJkNGYtZmJmN2U1MmY5MzM0In0sInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJ0ZXN0c2EiLCJ1aWQiOiIwMWU3MWUxOS0zMmQ3LTRhODEtYjM3Yy01NTM4ZmE4M2IwZWEifSwid2FybmFmdGVyIjoxNjg1MDk3MzQyfSwibmJmIjoxNjg1MDkzNzM1LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDp0ZXN0c2EifQ.oCCR5qdnL2bJirN-u8pqt6HKW3vfj-yuUn-zen7h8k1MJDwbFDLkTqtqyPBOae1loLEnxOjTBmjAReiv3xJ0NPJ8sSQ4pMIPVScKgAGEmq9hs5ZD60rEajKFh-THWdZSopdCiGFkZ9r5xPz2dZgoAFEDgOS1FzXBWekyioBG63OnVUsPZ6jJfSagw5cWYoCY8WcQCAu1NSfS2yGO_GRv4zUAIhT4ENtngicPnBw5a3MmHR4Zp7foROQmdW5oGom-2rCsk3IaKuwmUWGh26JOMl93Bx2fH3h8zHkc-KCzJZ9DAeoWxkIXRNtT5yCEN52ix1eoG_N7GRsnaaUvxzv9Tg'
    relabel_configs:
      - separator: ;
        regex: __meta_kubernetes_node_label_(.+)
        replacement: $1
        action: labelmap
      - separator: ;
        regex: (.*)
        target_label: __address__
        replacement: '10.90.0.170:6443' # <URL to you k8s API>
        action: replace
      - source_labels: [__meta_kubernetes_node_name]
        separator: ;
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
        action: replace
```

> token, ca.crt, api server tanımları yapıldı.

* Prometheus servisi restart edelim.

`systemctl restart prometheus`

<figure><img src="../.gitbook/assets/image (147).png" alt=""><figcaption><p>Veriler gelmeye başladı.</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>



{% embed url="https://documentation.commvault.com/11.24/essential/129223_create_service_account_for_kubernetes.html" %}

{% embed url="https://uzimihsr.github.io/post/2022-11-28-kubernetes-prometheus-kube-state-metrics-cadvisor/" %}
