---
icon: wrench
---

# Connectivity

Geleneksel ağ mimarilerinde trafik; güvenlik duvarı ve NAT gibi ara katmanlardan geçerken ek işlem yükü (overhead) oluşur, bu da gecikmeye (latency) neden olurdu.

* NAT'sız Hayat: Kubernetes diyor ki; "Pod A, Pod B'ye giderken arada, NAT olmasın."
* Şehirler arası yolda (Network) hiç gişe olmadığını ve hız sınırının olmadığını düşün. Veri paketleri Latency (gecikme) yaşamadan akar.

**2. Herkesin Bir Kimliği Var**

Sistem büyüdüğünde "Sen kimsin?" kargaşası yaşanmasın diye IP-per-Pod modeli kullanılır.

* Her Pod, sanki küçük bir sanal makineymiş gibi kendi IP adresine sahiptir.
* Pod'lar, Servisler ve Node'lar için IP aralıkları baştan belirlenmiştir. Böylece 1000. Pod da kurulsa, adresi hazırdır. Kimse kimsenin adresini işgal etmez.

**3. Tesisatçı ve Otomasyon (Etkili Yönetim)**

* CNI: Kubernetes'in "Tesisatçısıdır".
  * Bir Pod oluşturulduğunda Kubelet (Ustabaşı), CNI'ya seslenir: _"Yeni daire yapıldı, hemen internet kablosunu çek ve IP ver."_
  * Pod silindiğinde de CNI gelir, kabloyu toplar ve o IP'yi boşa çıkarır. Her şey otomatiktir.
* Pod'un evi yıkılıp yeniden yapılsa (IP değişse) bile, kapı numarası (Service IP) sabit kalır. Bağlantı kopmaz.

**4. Güvenlik ve Dış Kapı**

* Network Policies: Bu, binanın güvenlik görevlisidir. Yollar açıktır ama kimin hangi odaya girebileceğini Network Policy belirler. _"Database Pod'una sadece Backend Pod'u girebilir, Frontend giremez"_ diyebilirsin.
* NodePort / LoadBalancer: Bunlar da sitenin ana giriş kapılarıdır. Dışarıdaki (Internet) kullanıcıları içeri güvenle alırlar.
