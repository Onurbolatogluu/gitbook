---
icon: file-svg
---

# Scalability

* Mikro-VM Gibi Davranmak: Her Pod'un kendine ait bir IP'si vardır.
  * _Eski Dünya:_ Bir sunucuda 5 tane web sitesi çalıştırırsan, portları (80, 8080, 8081...) paylaştırmak zorundasın. Bu karmaşa büyümeyi engeller.
  * _K8s Dünyası:_ Her Pod'un kendi IP'si olduğu için hepsi standart 80 portunu kullanabilir.
* NAT'sız Otoban: Pod A, Pod B ile konuşurken arada NAT yapılmaz.
  * _Neden Ölçeklenir?_ Çünkü NAT işlemi CPU harcar. Binlerce Pod olduğunda bu işlemci yükü sistemi kilitlerdi. NAT olmadığı için sistem binlerce Pod'a rahatça çıkar.

**2. IP Address Management**

* CIDR Havuzları:
  * Pod'ların alacağı IP aralığı bellidir (Örn: 10.244.0.0/16).
  * Servislerin alacağı IP aralığı ayrıdır (Örn: 10.96.0.0/12).
  * Sunucuların IP'leri ayrıdır.
* Bu disiplin sayesinde, Cluster'a 500 yeni Node ve 5000 yeni Pod eklesen bile kimse kimsenin adresini işgal etmez.

**3. CNI Eklentileri**

* Siz sadece '100 Pod istiyorum' diyerek ölçeklendirme emrini verirsiniz. Node üzerindeki Kubelet, bu yeni Pod'ların ağa bağlanması ve IP alması için CNI (Container Network Interface) eklentisini tetikler.
* CNI, manuel müdahaleye gerek kalmadan saniyeler içinde yüzlerce Pod için gerekli sanal ağ arayüzlerini (veth) oluşturur, IP atamalarını yapar ve ağ trafiğini aktif hale getirir.
* Altyapı ihtiyacınıza göre uygun CNI eklentisini seçebilirsiniz. Örneğin, devasa ölçekli clusterlarda yönlendirme performansı için BGP kullanan Calico veya çekirdek seviyesinde (kernel) yüksek hız için eBPF tabanlı Cilium tercih edilebilir.
