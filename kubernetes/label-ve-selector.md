# 🏷 Label ve Selector

Etiketler, yani tag'ler Kubernetes'de her türlü objeye atayabildiğimiz, anahtar-değer eşlenikleridir. Etiketler sayesinde, bizler oluşturduğumuz objelere atabildiğimiz ve bizlerin anlayacağı ve gruplama yaparken kullanabileceğimiz bilgiler eklemiş oluruz. Bu sayede kubernetes tarafından bizlere core özellik olarak sunulmayan objelere belirli bir aidiyet ataması gerçekleştirebiliriz. Obje oluştururken, atanabileceği gibi, obje oluşturulduktan sonra da etiketler atanabilir. Etiketler key-value şeklinde oluşturulur.

```
example.com/tier:frontend
  prefix     Key   Value
  
Prefix kısmı opsiyoneldir.
```

Kubernetes 'de objeler arası bağlantı da etiketler sayesinde kurulur. Servisler ve deployment objeleri, hangi pod'ların kendilerine ait olduğunu ve hangi pod'lar ile ilişki kuracaklarını etiketler sayesinde belirler.&#x20;

Örneğin, bir servis oluşturur ve bu servise şunu deriz; Git ve tier : frontend etiketli podları bul, ve trafiği bunlara yönlendir. vb.

Özetle, Label(etiketler) objeleri gruplama imkanı vermenin yanı sıra, Kubernetes 'de objeler arası bağ kurmak için kullandığı en temel mekanizmadır.&#x20;

{% hint style="info" %}
Bir yml dosyasında, birden çok pod tanımı yapabiliriz. Aralarında (---) olmak şartıyla.
{% endhint %}

<mark style="color:red;">Metadata kısmında, etiket tanımı yaparız. ve istediğimiz kadar bu etiket sayısını arttırabiliriz.</mark>

```
kubectl get pods -L "app"
# app etiketine sahip podları getirir. --show-label parametresini eklersek, mevcut
etiketleride gösterir.

kubectl get pods -L "app:firstapp" --show-labels
# App etiketine sahip ve firstapp değerine sahip podları getirir.

# Daha fazla etikete göre aramak için tırnak arasına , işareti koyup, 2.istediğimiz
etiketide arayabiliriz.
"app:firstapp,tier:frontend" 

"app:firstapp,tier!=frontend"
# Tier etiketi frontend değeri olmayan,değeri bul ve o podları getir.

kubectl get pods -L "app in (firstapp)" --show-label
# App anahtarına firstapp değeri olan podları getir.

"app in (firstapp,secondapp)"
# APP değeri firstapp veya secondapp olan podları listele.

"app notin (firstapp)"
# App anahtarının değeri firstapp olmayan podları listele.

"app in (firstapp),tier notin(frontend)"
# app anahtarının değeri firstapp olacak, tier etiketinin değeri frontend olmayan
podları getir.


kubectl label pods pod9 app=thirdapp
# Pod9 poduna etiket ekledik.
# Etiket güncellemelerini yml dosyasını güncelleyerek yapabiliriz.

kubectl label pods pod9 app -
# Label'i sildik.

kubectl label --overwrite pods pod9 team=teams3
# etkiketi güncelledik.

kubectl label pods -all foo:bar
# Tüm namespace altında bulunan podlara etiket eklemek için.
```

YML dosyasına gireceğimiz node selector parametresi ile bu pod'un çalışmasını istediğimiz, nodeları belirtebiliyoruz. Bunu da label arayıcılığıyla yapıyoruz.

Örneğin,

```
nodeselector:
  hddtype:ssd
```

Kube scheduler 'e şunu diyoruz, bu podu oluştururken,  hddtype:ssd değerine sahip olan bir node bul ve bu podu orada çalıştır. Bu pod objesi ile node objesini eşleştirdik. Labeller bize objeler arası ilişki kurmaya yarar dediğimiz kısım budur.

```
kubectl label nodes worker-2 hddtype:ssd
# node 'a etiket atadık

kubectl delete -f podlabel.yml
# podlabel.yml dosyasından oluşturduğumuz tüm podları siler.
```

