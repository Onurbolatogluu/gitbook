# 🍵 Kubectl ve Versiyon

* Kubernetes semantik olarak versiyonlanır. x(major).y(minor).z(patch).
* Kubernetes sene de 3 minör versiyon yayınlar. Her minör versiyon yayınladığı tarihten itibaren 12 ay boyunca desteklenir. Kısacası 12 ay boyunca, her ay patch versiyonları yayınlanarak, hatalar ve güvenlik açıkları giderilir.
* Kubernetes yönetimi, 3 farklı şekilde yapılır. Rest API,GUI araçları,Kubectl.
* Minikube ve docker desktop 'a kubernetes cluster kurabiliriz.
* kubeadm ve kubespray gibi araçlarla kendi sanal sunucularımıza kubernetes cluster kurabiliriz.
* Amazon AWS,GCP,Azure gibi cloud servis sağlayıcılarının hazır kubernetes araçları ile kubernetes cluster kurabiliriz.

### Kubectl

* Kubectl aracı, bağlanacağı kubernetes cluster bilgilerine config dosyası arayıcılığıyla erişir.
* Config dosyası içerisinde, kubernetes cluster bağlantı bilgilerini ve oraya bağlanırken kullanmak istediğimiz kullanıcıları belirtiriz.
* Daha sonra bu bağlantı bilgileri ve kullanıcıları ve ek olarak namespace bilgilerini de, oluşturularak context'ler yaratırız.
* Kubectl varsayılan olarak $home/.kube/ altındaki config isimli dosyaya bakar. Ama bunu KUBECONFIG environment variable değerini değiştirerek güncelleyebiliriz.
* Kubectl config get-context ile mevcut çalışılan cluster bilgisini öğrenebiliriz.
* Kubectl config use-context {context name} ile, geçmek istediğimiz context 'i yazarak, farklı bir cluster 'a geçiş yapabiliriz.



```
kubectl cluster-info
# cluster hakkında bilgi verir.

kubectl --help
# komutlar ve kullanımları ile alakalı bilgi almak için kullanırız.

kubectl get pods -n kube-system
# komutun çalışmasını istediğimiz cluster namespace bilgisi belirtebiliriz.

kubectl get pods -A -o wide
# Komutların sonuna -0 wide yazarsak, daha detaylı bir çıktı alabiliriz.
# wide yerine, yaml ve json yazabiliriz.

kubectl explain pod
# Pod objesi hakkında bilgi alabiliriz. 
```

{% embed url="https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands" %}
