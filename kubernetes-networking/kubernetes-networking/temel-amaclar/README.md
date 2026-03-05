---
icon: basketball
---

# Temel Amaçlar

Sistem dünyasında "hız" sadece işlemci hızı değildir, ağdaki gecikme (latency) de çok önemlidir.

* Önceki bölümde konuştuğumuz gibi, Pod'lar birbirleriyle konuşurken arada bir NAT olmadığı için vakit kaybetmezler. Doğrudan iletişim kurarlar.
* Aynı Pod içindeki Container'lar, internete çıkmadan, sanki aynı bilgisayarın içindeymiş gibi (`localhost`) konuşurlar. Bu, veri transferini ışık hızına çıkarır.

**2. Ölçeklenebilirlik**

Bir sistemi 5 kullanıcı için kurmak kolaydır, ama 5 milyon kullanıcıya çıktığında işler karışır. Kubernetes bunu baştan çözer.

* IP-per-Pod: Her Pod'un kendine ait bir kimliği (IP) olması, sistemi sanal makineler (VM) gibi yönetmeyi sağlar. 10 Pod da olsa, 10.000 Pod da olsa adresleme mantığı değişmez.
* IP Havuzları (CIDR): Pod'lara, Service'lere ve Node'lara verilen IP aralıkları baştan bellidir. Böylece sistem büyürken "Eyvah, IP çakışması oldu!" (IP conflict) derdi yaşanmaz.

**3. Etkili Yönetim**

Yüzlerce mikro servisin olduğu bir ortamı elle yönetemezsin. Otomasyon şarttır.

* CNI Standardı: Kubernetes, ağ işini "Plug-and-Play" mantığına oturtmuştur. İhtiyacına göre; "Bana güvenlik lazım" dersen Calico, "Hız lazım" dersen Cilium eklentisini kullanırsın. Ağ yönetimini standartlaştırır.
* Pod'lar geçici (ephemeral) yapıda oldukları için her yeniden başlatıldıklarında IP adresleri değişir. Service katmanı ise sabit bir erişim noktası (ClusterIP) sağlayarak, arka plandaki bu IP değişiminden etkilenmeden trafiğin doğru Pod'lara iletilmesini garanti eder.
  * Bir şirketi aradığında "Ahmet beyi" değil, "Muhasebe departmanını" (Service) ararsın. Ahmet işten ayrılsa bile (Pod ölse bile), o numara (Service IP) hep cevap verir.

**4. Bağlanabilirlik**

Sistemin çalışması için trafiğin 4 ana yolda sorunsuz akması gerekir (bunu önceki makale de görmüştük):

1. Container-to-Container: (Ev içi fısıldaşma)
2. Pod-to-Pod: (Komşular arası konuşma)
3. Pod-to-Service: (Departmana ulaşma)
4. External-to-Service: (Müşterinin dükkana girmesi)

***

Özetle: Kubernetes ağ yapısı rastgele kurulmamıştır. Performans (NAT'sız yapı), Ölçeklenebilirlik (Her Pod'a IP) ve Yönetilebilirlik (CNI ve Service'ler) üzerine kurgulanmıştır.
