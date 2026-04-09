---
icon: align-justify
---

# VMware vSphere Components and Features

Sanallaştırma dünyasına adım attığımızda, donanımı nasıl parçalayıp sanal makinelere dağıttığımızı öğrendik. Doğrudan donanım üzerine kurduğumuz Bare Metal (Type 1) hypervisor'ümüz olan VMware ESXi, altımızdaki sunucuyu ( host ) sanal bir fabrikaya dönüştürdü.

Peki ama bu fabrikayı nasıl yöneteceğiz? Hele ki elimizde sadece 1 değil, 50 tane böyle fabrika (host) varsa?

Bu yazıda, VMware mimarisinin yönetim katmanlarını ve sistem yöneticilerini kâbuslardan kurtaran o büyük icadı, vCenter Server'ı inceliyoruz.

#### 🧱 1. Geleneksel Yönetim: Standalone ESXi

ESXi kurduğunuz bir fiziksel sunucunun başına geçip, klavye ve monitör bağlayarak yapabileceğiniz ayarlar son derece kısıtlıdır (sadece ağ ayarları ve basit log okumaları yapabilirsiniz).

Sanal makineler (VM) yaratmak, onlara CPU veya RAM atamak için uzak bir cihazdan (örneğin laptop'ınızdan) bir web tarayıcısı (Chrome, Firefox vb.) açmanız gerekir. Tarayıcıya ESXi sunucusunun IP adresini yazar, kullanıcı adı ve şifrenizi girerek web tabanlı vSphere Client arayüzüne ulaşırsınız.

Sorun Nerede Başlıyor?

Diyelim ki şirketiniz büyüdü ve veri merkezinizde 20, 50 veya 100 adet fiziksel host'unuz var.

Bir VM'in hangi host'ta olduğunu bulmak için, tarayıcıda tek tek bu 50 sunucunun IP adresini girmek, her biri için ayrı ayrı şifre yazmak ve 50 farklı sekmeyle boğuşmak zorunda kalırsınız. Bu mimari, operasyonel olarak yönetilemez bir kaostur.

```
[ Geleneksel ve Kısıtlı Yönetim Topolojisi ]

    💻 Senin Laptop'ın (vSphere Client)
       /       |       |       \
      /        |       |        \  (Her biri için ayrı IP, ayrı Login!)
     /         |       |         \
[Host 1]   [Host 2]  [Host 3]   [Host 50]
 (ESXi)     (ESXi)    (ESXi)     (ESXi)
```

#### 🧠 2. Oyunu Değiştiren Aktör: vCenter Server

İşte tam bu noktada, VMware'in kalbi devreye girer: vCenter Server.

vCenter Server, ağınıza kurduğunuz merkezi bir yönetim uygulamasıdır. Artık her bir host'a tek tek IP yazıp bağlanmazsınız. Tüm ESXi host'larınızı (kullanıcı adı ve şifreleriyle birlikte) bir kereye mahsus olmak üzere vCenter'a eklersiniz.

Artık sabah işe geldiğinizde tarayıcınızı açar, _sadece vCenter'ın IP adresine_ girer ve tek bir şifreyle giriş yaparsınız. Karşınıza çıkan o devasa ekranda, veri merkezinizdeki 50 host'u ve içlerinde koşan 1000'lerce sanal makineyi tek bir ağaç hiyerarşisi altında görürsünüz.

```
[ vCenter Server Merkezi Yönetim Topolojisi ]

           💻 Senin Laptop'ın (vSphere Client)
                       |
               (Tek bir IP, Tek Login)
                       |
             +-------------------+
             |  vCenter Server   |  <--- Tüm Veri Merkezinin Beyni
             +-------------------+
            /          |          \
           /           |           \
     [Host 1]      [Host 2]      [Host 50]  <--- vCenter tarafından yönetilir
      (ESXi)        (ESXi)        (ESXi)
```

#### 🌟 vCenter'ın Gizli Gücü: O Bahsedilen "%80'lik" Kısım Ne?

vCenter olmadan yapamayacağımız, kurumsal firmaların milyonlarca dolar ödediği bu yetenekler nelerdir?

* vMotion (Canlı Göç): Host 1'in RAM'i bozulmak üzere ve sunucuyu acil kapatmanız gerekiyor. vCenter sayesinde, Host 1 üzerinde çalışan bir sanal makineyi (içindeki işletim sistemi çalışmaya devam ederken ve kullanıcının ruhu bile duymazken) saniyeler içinde canlı olarak Host 2'ye taşıyabilirsiniz.&#x20;
* High Availability (HA): Eğer Host 1 aniden alev alır ve kapanırsa, vCenter bunu saniyeler içinde fark eder. Host 1'in içinde ölen sanal makineleri alır ve otomatik olarak Host 2 veya Host 3 üzerinde yeniden başlatır. Sistemin kesinti süresi sadece dakikalarla sınırlı kalır.
* DRS (Distributed Resource Scheduler): vCenter akıllı bir sistemdir. Eğer Host 1 üzerindeki VM'ler çok fazla CPU kullanmaya başlar ve sunucu boğulursa, vCenter araya girer ve bazı VM'leri otomatik olarak daha boş olan Host 2'ye taşır.
* Centralized Datastore: Tüm depolama alanlarınızı (SAN/NAS mimarilerini) tek bir merkezden tüm host'lara aynı anda tanıtabilirsiniz.

#### 🎯 Sonuç

Eğer sadece kendi bilgisayarınızda bir Type 2 hypervisor (Workstation vb.) ile laboratuvar ortamı kuruyorsanız, her şeyi tek tek yönetmek sorun değildir. Ancak gerçek dünya senaryolarında, birden fazla ESXi host'un bulunduğu bir yapıda vCenter Server lüks değil, mutlak bir zorunluluktur. Altyapıyı bir cluster mantığıyla tek bir devasa bilgisayarmış gibi yönetmenizi sağlayan güç tamamen vCenter'da yatar.
