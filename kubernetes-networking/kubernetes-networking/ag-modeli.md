---
icon: wifi-fair
---

# Ağ Modeli

Eskiden (Docker'ı tek başına kullandığımızda) ağ işleri tam bir kabustu. Port çakışmaları, karmaşık yönlendirmeler...

**1. IP-per-Pod**

Bu kural en önemlisidir.

* Eski Usül: Bir sunucuda 3 uygulama çalıştırırsan, dışarıdan erişmek için port atamak zorundaydın (App1: Port 80, App2: Port 8080...).
* K8s Kanunu: Her Pod, sanki kendi başına küçük bir bilgisayarmış (Mikro-VM) gibi kendi IP adresini alır.
* Kimse kimsenin portunu işgal etmez. Herkesin kendi kimliği vardır.&#x20;

**2.NAT'sız İletişim**

Kubernetes, Cluster içini "Schengen Bölgesi" gibi görür. Sınırlar ve kontroller kaldırılmıştır.

* Kural: Bir Pod, başka bir Node üzerindeki Pod ile konuşurken arada NAT (Network Address Translation) yapılmaz.
* Nasıl ki evindeki bir odadan diğerine giderken pasaport göstermiyorsan, Pod'lar da birbirine giderken NAT yapmaz.
* Sonuç: Çeviri yoksa gecikme (Latency) yoktur. İşlemci yorulmaz (Overhead azalır), sistem hızlı çalışır.

***

#### 🏠 Pod İçi Yaşam

* Aynı Pod içindeki Container'lar (örneğin ana uygulama ve log toplayıcı), aynı evi paylaşan öğrenciler gibidir.
* Tek IP: Evin kapı numarası tektir (Pod IP).
* Localhost: Ev içindeki öğrenciler birbirine seslenirken telefon kullanmaz, `localhost` üzerinden konuşurlar.
