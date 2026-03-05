---
icon: building
---

# Efficient Operation

&#x31;**. Teknoloji Desteği (Cilium ve eBPF)**

* eBPF Nedir? Linux çekirdeğine "yama yapmadan" özellik eklemeni sağlayan çok güçlü bir teknolojidir.
* Cilium: Bu teknolojiyi kullanan bir CNI (Ağ eklentisi).
* Standart ağ trafiği, şehir içi trafiğinde ışıklara takılarak gitmektir. Cilium ve eBPF kullanmak ise, altına özel bir tünel kazıp (Kernel seviyesinde işlem yapıp) trafiği oradan akıtmaktır. Çok daha hızlıdır.

**2. Düzenli Planlama**

Karmaşa her zaman yavaşlık getirir.

* CIDR (IP Blokları): Pod'ların, Service'lerin ve Node'ların IP aralıkları baştan bellidir.
* _Neden Verimli?_ Sistem, "Acaba bu IP kime aitti?" diye düşünmez veya yanlış yere paket gönderip vakit kaybetmez. Her şeyin yeri bellidir.
