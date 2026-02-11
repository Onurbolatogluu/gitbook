---
icon: message-medical
---

# Advanced

**1. Ağ Güvenliği**

* Network Policies: Bu yapı, cluster içindeki güvenlik duvarı kurallarıdır. CNI eklentileri (özellikle Calico ve Cilium) aracılığıyla uygulanır. Varsayılan olarak tüm trafik serbesttir; ancak Network Policy ile "Sadece Frontend Pod'ları, Backend Pod'larına 8080 portundan erişebilir" gibi whitelist mantığı kurulur.
* Şifreleme (Encryption & mTLS): Verinin ağ üzerinde taşınırken güvenli olması esastır. mTLS (Mutual TLS) ile servisler birbirini karşılıklı doğrular ve şifreli konuşur. Cert-Manager gibi araçlar sertifika yönetimini otomatikleştirir.

**2. Gelişmiş Trafik Yönetimi**

Standart "Service" (ClusterIP/NodePort) yapıları, karmaşık yönlendirme ihtiyaçları için yetersiz kalabilir.

* Ingress ve Ingress Controllers: Dış dünyadan gelen HTTP/HTTPS trafiğini yönetmek için kullanılır. Traefik veya Nginx gibi Ingress Controller'lar, gelen trafiği URL yoluna (path-based) veya alan adına (host-based) göre ilgili servislere yönlendirir (L7 Routing).
* Service Mesh: Cluster içindeki servisler arası iletişimi yöneten özel bir katmandır. Trafik yönetimi, hata toleransı ve izleme yeteneklerini uygulama kodundan soyutlayarak altyapıya indirger.
* Multi-Cluster: Birden fazla Kubernetes kümesinin tek bir ağ gibi çalışmasını sağlayan mimarilerdir.

**3.Observability**

Ağda bir sorun olduğunda "Paket nerede kayboldu?" sorusuna yanıt verebilmek için izleme araçları gereklidir.

* Cilium Hubble gibi araçlar, ağ akışlarını görselleştirir. Hangi servisin hangisine ne zaman eriştiğini, hangi paketlerin engellendiğini ve latency analiz etmeyi sağlar.

**4. Dış DNS Yönetimi**

Pod IP'leri geçicidir, ancak dış dünyadan erişim sağlayan Load Balancer adresleri sabittir. External DNS; Kubernetes servislerinin dış IP'lerini otomatik olarak algılar ve DNS sağlayıcına (AWS Route53 vb.) kaydederek, `app.sirketim.com` gibi alan adlarının her zaman doğru erişim noktasına yönlenmesini sağlar.
