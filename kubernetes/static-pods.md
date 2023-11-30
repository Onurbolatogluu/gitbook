# 📖 Static Pods

Bizler sürekli "kubectl" arayıcılığıyla, kubernetes api server'a komutlar vererek, pod oluşturma işlemlerini yaptık. Fakat kubernetes üstünde, bir tane daha pod oluşturma yöntemimiz daha var. Buna da "static pods" diyoruz.&#x20;

Bu özelliği bize sağlayan "kubelet" komponentidir. Kubelet kendi üzerinde, yani çalıştığı node üzerinde, bir klasör belirleyebiliyoruz. Bu klasörün içerisinde, biz yaml dosyaları şeklinde pod tanımımızı koyarsak, kubelet bu dosyayı ( podu ), bulunduğu node üzerinde ayağa kaldırır.

Yani direkt olarak, kubernetes api 'ya gidip, yeni bir pod talebi oluşturmak yerine, kubelet ile direkt haberleşip, yani kubelet 'in belirlediğimiz dizinine veya default dizinine oluşturulmasını istediğimiz pod tanımını yaptığımız "yaml" dosyasını koyarak kubelet'in podlar yaratmasını söyleyebiliyoruz. Bu duruma da Kubernetes ortamında "static pods" diyoruz.

Kubernetes cluster'ın, kendisi de static podlar ile ayağa kalkar ve cluster haline gelir. Kubelet master sunucu olacak node'un, /etc/kubernetes/manifest dizinine bakar ve kubernetes cluster için gerekli olan yaml dosyalarını okur ve oluşturur.

<figure><img src="../.gitbook/assets/image (169).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

Özetle, kubelet varsayılan olarak kendi üzerinde bulunduğu host 'da bir tane dizine bakar. Varsayılan olarak bu path "/etc/kubernetes/manifest/" dizinidir. Bu dizin içerisinde, yaml dosyası şeklinde pod tanımlarımızı koyarsak, kubelet yaml dosyasında tanımını yaptığımız podu bulunduğu node üzerinde ayağa kaldırır ve daha sonrasında bu podu, api server üzerinde "kubectl get pods" şeklinde komut çalıştırarak görüntüleyebiliriz. Bu static pod üzerinde değişiklik yapmak istediğimizde, ilgili node'un "manifest" dizinine gidip, yaml dosyası üzerinde istediğimiz değişikliği yapabiliriz. Bu podu api server üzerinden değiştiremeyiz. Ek olarak, Pod'u silmek istediğimizde, ilgili dosyayı dizinden sildiğimizde static pod silinir.

{% embed url="https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/" %}

