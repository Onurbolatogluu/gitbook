---
icon: '4'
---

# Dört Temel İletişim Türü

#### 🚦 Kubernetes Trafiğinin 4 Ana Yolu

**1. Ev Arkadaşları (Container-to-Container)**

Aynı Pod içinde yaşayan Container'ların iletişimidir.

* Durum: Bir stüdyo dairede (Pod) yaşayan iki arkadaş (Container) düşün.
* Nasıl Konuşurlar?: Telefon etmelerine gerek yoktur, aynı odadalar. Birbirlerine `localhost` üzerinden seslenirler.
* Teknik Karşılığı: Aynı Network Namespace'i paylaşırlar. Yani IP adresleri ve MAC adresleri aynıdır. En hızlı iletişim budur.

**2. Komşuluk İlişkileri (Pod-to-Pod)**

Mahalledeki (Cluster) farklı evlerin (Pod'ların) birbirleriyle konuşmasıdır.

* Durum: A Blok'taki Pod, B Blok'taki Pod'a veri göndermek istiyor.
* Nasıl Konuşurlar?: Herkesin kapı numarası (IP) bellidir. Arada güvenlik kulübesi veya NAT yoktur. Dümdüz yürüyüp kapıyı çalarlar.
* Teknik Karşılığı: IP-per-Pod modeli sayesinde, Pod'lar hangi Node üzerinde olursa olsun, doğrudan IP ile iletişim kurarlar.

**3. Sabit Danışma Hattı (Pod-to-Service)**

Burası çok kritiktir. Pod'lar ölümlüdür, bugün var yarın yoklar. IP'leri sürekli değişir.

* Sorun: Sürekli taşınan bir arkadaşını nasıl bulursun?
* Çözüm: Onu cepten değil, sabit şirket hattından (Service) ararsın.
* Nasıl Konuşurlar?: Sen Service'e (sanal bir IP adresi olan ClusterIP) paketi gönderirsin. Service, o an nöbetçi olan, çalışan Pod'a paketi iletir.
* Teknik Karşılığı: Service Discovery ve Load Balancing. Pod ölse de Service IP'si sabit kalır, iletişim kopmaz.

**4. Müşteri Kapısı (External-to-Service)**

* Durum: Web sitene bir kullanıcı girmek istiyor.
* Nasıl Konuşurlar?: Sitenin dış kapısını (NodePort) veya özel VIP girişini (LoadBalancer) kullanırlar. Bu kapıdan giren trafik, içerideki ilgili Service'e, oradan da Pod'a yönlendirilir.
* Teknik Karşılığı: Ingress trafiğidir. Güvenli ve kontrollü bir giriş noktasıdır.

***

1. Aynı evdeysen (Pod içi) -> Localhost kullan.
2. Komşuya gideceksen (Pod'dan Pod'a) -> Doğrudan IP ile git.
3. İş yapacaksan (Uygulamaya erişim) -> Service üzerinden git.
4. Dışarıdan geleceksen -> NodePort/LoadBalancer kapısını çal.
