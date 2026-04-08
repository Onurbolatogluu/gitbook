---
icon: sparkle
---

# How To Build A Virtual Environment

IT dünyasında "sanallaştırma" (virtualization) kavramı 13 yılı aşkın süredir hayatımızda olsa da, modern veri merkezlerinin ve cloud mimarilerinin hala en büyük itici gücüdür. Peki ama neden bu kadar önemli?

Sıfırdan başlayan biri için en temel tanımıyla sanallaştırma; tek bir fiziksel sunucu (donanım) üzerinde, sanki birden fazla bilgisayar varmış gibi birbirinden bağımsız çalışan birden fazla işletim sistemini (virtual machine) aynı anda ve sorunsuz bir şekilde çalıştırma teknolojisidir.

#### 💡 Neden Sanallaştırmaya İhtiyacımız Var?

Bir sunucunun %20 kapasiteyle çalışırken harcadığı elektrik ile %80 kapasiteyle çalışırken harcadığı elektrik neredeyse aynıdır. Geleneksel sistemlerde her bir servis için ayrı bir fiziksel makine alınırdı ve bu devasa bir kaynak israfına yol açardı.

Diyelim ki üretim ve satış ağı giderek genişleyen, birden fazla şubesi olan bir yapı ve inşaat malzemeleri şirketiniz var. Altyapınızda şu sunuculara ihtiyacınız var:

1. Active Directory Cluster: Kullanıcı yetkilendirmeleri için (Kapasitenin %70'ini tüketiyor)
2. Mail Server: Şirket içi iletişim için (%12 tüketiyor)
3. Application Server: Üretim takibi veya ERP yazılımınız için (%25 tüketiyor)
4. Accounting Server: Muhasebe departmanı için (%5 tüketiyor)

Eğer sanallaştırma kullanmazsanız, muhasebe yazılımınız için koca bir fiziksel sunucu alıp onun %95'lik kapasitesinin boşta yatmasını (ve elektrik/soğutma masrafı yaratmasını) izlersiniz.

Çözüm: Çok yorulan _Active Directory_ sunucularını kendi halinde bir cluster olarak bırakıp; Mail, Application ve Accounting sunucularını tek bir güçlü fiziksel sunucu ( host ) içine virtual machine (VM) olarak kurmaktır.

#### 🌟 Sanallaştırmanın Bize Kazandırdıkları

* Maliyet Optimizasyonu: Daha az donanım, daha az kablo, daha az elektrik ve soğutma masrafı.
* İşletim Sistemi Çeşitliliği: Aynı fiziksel host üzerinde Linux (örneğin bir web sunucusu için), Windows (muhasebe için) ve macOS tabanlı sistemleri yan yana çalıştırabilirsiniz.
* Rapid Deployment & Cloning: Yeni bir sunucuya mı ihtiyacınız var? Donanım sipariş edip günlerce beklemek yerine, mevcut bir sistemin imajını kopyalayarak (cloning) dakikalar içinde yeni bir node ayağa kaldırabilirsiniz.
* Resource Allocation & Load Balancing: Kaynakları (CPU, RAM) sanal makineler arasında dinamik olarak dağıtabilir, yük dengelemesi yapabilirsiniz.

> ⚠️ Eğer bir fiziksel sunucunuz zaten %90-%100 kapasiteyle, çok ağır bir veritabanı işlemi yapıyorsa, onu sanallaştırmak mantıklı olmayabilir. Sanallaştırma katmanı kendi içinde ekstra bir yük (overhead) yaratacağı için performans kaybı yaşayabilirsiniz.

***

#### 🧠 Sanallaştırmanın Kalbi: Hypervisor Nedir?

Sanallaştırmanın gerçekleşmesini sağlayan, donanım ile sanal makineler arasında trafik polisliği yapan o sihirli yazılıma Hypervisor diyoruz. Sektörde mimari yapısına göre iki tip Hypervisor bulunur:

**1. Type 1 Hypervisor (Bare-Metal)**

Bu mimaride arada Windows veya macOS gibi bir işletim sistemi yoktur. Hypervisor doğrudan donanımın (bare-metal) üzerine kurulur. Kendi minimal bir çekirdeği (kernel) vardır ve donanıma (CPU, RAM, Disk) doğrudan eriştiği için inanılmaz performanslıdır.

* Kullanım Alanı: Gerçek dünya production ortamları, veri merkezleri.
* Örnekler: VMware vSphere (ESXi), Microsoft Hyper-V, KVM, Xen.

```
[Type 1 - Bare-Metal Topolojisi]

+--------------------+ +--------------------+
|  Virtual Machine 1 | |  Virtual Machine 2 |  <-- (Linux, Windows vb.)
|    (Guest OS)      | |    (Guest OS)      |
+--------------------+ +--------------------+
|                                           |
|        HYPERVISOR (Örn: VMware ESXi)      |  <-- Donanımla direkt konuşur!
|                                           |
+-------------------------------------------+
|         FİZİKSEL DONANIM (Hardware)       |  <-- (CPU, RAM, Disk, Network)
+-------------------------------------------+
```

**2. Type 2 Hypervisor (Hosted)**

Bu modelde sistem, halihazırda var olan bir işletim sisteminin (örneğin günlük kullandığınız MacBook'unuzdaki macOS veya bir PC'deki Windows) üzerine sıradan bir uygulama gibi kurulur. Donanıma ulaşmak için altındaki asıl işletim sisteminden izin alması gerektiği için performansı Type 1'e göre daha düşüktür.

* Kullanım Alanı: Bireysel testler, yazılım geliştirme, eğitim ve Lab ortamları.
* Örnekler: VMware Workstation, VMware Fusion, Oracle VirtualBox.

```
[Type 2 - Hosted Topolojisi]

+--------------------+ +--------------------+
|  Virtual Machine 1 | |  Virtual Machine 2 |  <-- (Test ortamı sistemleri)
+--------------------+ +--------------------+
|                                           |
|  HYPERVISOR (Örn: VMware Workstation)     |  <-- Bir program gibi çalışır
|                                           |
+-------------------------------------------+
|      HOST OS (Örn: Windows 11 / macOS)    |  <-- Ana işletim sisteminiz
+-------------------------------------------+
|         FİZİKSEL DONANIM (Hardware)       |
+-------------------------------------------+
```

{% hint style="info" %}
Endüstri standartlarında her zaman Type 1 (Bare-Metal) kullanılır.&#x20;
{% endhint %}

