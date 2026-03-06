---
icon: align-justify
---

# Troubleshooting Internal Networking

Kubernetes cluster'ınızda iki uygulama aniden birbiriyle konuşmayı kestiğinde, sorunun kaynağını bulmak samanlıkta iğne aramaya benzeyebilir. Sorun DNS'te mi? Güvenlik duvarında mı? Yoksa etiketlerde mi (labels) bir yazım hatası var?

Kaos içinde kaybolmamak için Kubernetes ağ mimarisini 4 mantıksal katmana ayırmalı ve sorunu aşağıdan yukarıya doğru (altyapıdan uygulamaya) izole etmeliyiz. İşte ağ sorunlarını çözerken izlemeniz gereken 4 adımlı standart prosedür.

### 1. Katman: CNI

Container Network Interface (CNI), Kubernetes'in fiziksel ağ kablosudur (Örn: Cilium, Calico, Flannel). Eğer CNI çalışmıyorsa, üstteki hiçbir şey çalışmaz.

Nasıl Kontrol Edilir?

Tüm CNI eklentileri arka planda `kube-system` namespace'i içinde birer Pod olarak çalışır. İlk iş olarak tesisatın çöküp çökmediğine bakmalıyız:

* `kubectl get pods -n kube-system` komutunu çalıştırın. CNI pod'larında sürekli bir yeniden başlama (CrashLoopBackOff) var mı?
* Eğer Cilium kullanıyorsanız, sunucuların (Node) ağ durumunu görmek için terminale doğrudan `cilium status` yazabilirsiniz.
* Node sağlığını kontrol edin. Alt yapıda Docker veya Containerd gibi runtime'ların ayakta olduğundan emin olun.

### 2. Katman: Network Policies

Tesisat sağlam ama trafik hala geçmiyor mu? Belki de görünmez bir güvenlik görevlisi kapıyı tutuyordur. Network Policies, Kubernetes'in iç güvenlik duvarıdır. Yanlış yazılmış bir kural, trafiği sessizce (hiçbir hata mesajı vermeden) engeller.

En Sık Yapılan Hatalar ve Çözümleri:

* Gizli Bloklar: Boş bırakılmış bir Network Policy, varsayılan olarak o namespace'e gelen veya giden tüm trafiği engeller. `kubectl get networkpolicies --all-namespaces` komutuyla devrede olan kuralları listeleyin.
* Etiket Uyuşmazlığı: `podSelector` veya `namespaceSelector` kısmındaki etiketleri kontrol edin. Kurallar doğru uygulamaları hedefliyor mu?
*   Manuel Test: İzin verildiğini düşündüğünüz iki Pod arasına girip fiziksel test yapın:

    `kubectl exec -it test-pod -- nc -zv <hedef-pod-ip> 80`

### 3. Katman: Service Discovery ve DNS

Güvenlik duvarında sorun yok ama uygulamanız "Böyle bir adres bulunamadı" hatası alıyor. Bu durum, içerideki telefon rehberinin (CoreDNS) çöktüğünü gösterir.

DNS Sorunlarını Nasıl Yakalarız?

* CoreDNS Sağlığı: Tıpkı CNI gibi, DNS de `kube-system` içinde çalışır. `kubectl get pods -n kube-system -l k8s-app=kube-dns` komutuyla DNS pod'larının `Running` durumunda olduğundan emin olun. Gerekirse `kubectl logs` ile loglarına bakın.
*   İçeriden Çözümleme Testi: Sorun yaşayan Pod'un içine girip bir DNS sorgusu atın:

    `nslookup kubernetes.default` veya `nslookup hedef-servis.namespace.svc.cluster.local`
* Yapılandırma Hatası: DNS sunucunuz yanıt vermiyorsa, yanlış bir ayar yapılmış olabilir. `kubectl get configmap coredns -n kube-system -o yaml` ile konfigürasyon dosyasını inceleyin.

### 4. Katman: Services, Endpoints ve Pods

Kablolar sağlam, güvenlik duvarı açık, DNS adresi çözümlüyor ama hala bağlantı yok! Sorun çok büyük ihtimalle YAML dosyanızdaki ufak bir insan hatasıdır.

Adım Adım Eşleştirme Kontrolü:

* Pod'lar Hayatta mı? Trafiğin gideceği hedef Pod'lar gerçekten ayakta mı? `CrashLoopBackOff` durumundalarsa, servis onlara trafik göndermez.
* Etiket (Label) Eşleşiyor mu? Servisinizin YAML dosyasındaki `spec.selector` ile Pod'unuzun etiketleri harfi harfine aynı olmak zorundadır. Aksi takdirde servisiniz "kör" olur.
* Endpoint Listesi Dolu mu? `kubectl get endpoints <servis-adi>` komutunu çalıştırın. Eğer karşısında IP adresleri yerine `<none>` yazıyorsa, servisiniz hiçbir Pod'u bulamamış demektir.
* Port Karmaşası: Servisin `port` değeri ile uygulamanın dinlediği `targetPort` değerinin birbirini tuttuğundan emin olun.

Altın İpucu (Port-Forward):

Sorunun Service nesnesinde olup olmadığını kesin olarak anlamak için servisi aradan çıkarın! `kubectl port-forward <pod-adi> 8080:80` komutuyla doğrudan Pod'a kendi bilgisayarınızdan bağlanın. Uygulama cevap veriyorsa sorun Pod'da değil, önündeki Service tanımındadır.

***

| **Şüphelenilen Katman** | **Odak Noktası**                | **Hayat Kurtaran Komutlar**                                   |
| ----------------------- | ------------------------------- | ------------------------------------------------------------- |
| CNI (Altyapı)           | Ağ eklentileri ayakta mı?       | `kubectl get pods -n kube-system`                             |
| Network Policies        | Trafik engelleniyor mu?         | `kubectl get networkpolicies`                                 |
| CoreDNS                 | İsimler IP'ye çevriliyor mu?    | `nslookup`, `kubectl logs -l k8s-app=kube-dns -n kube-system` |
| Service & Endpoints     | Trafik doğru Pod'a ulaşıyor mu? | `kubectl get endpoints`, `kubectl describe svc`               |
