---
icon: ethereum
---

# The VMkernel, The Port Group and The Physical Adapters

Fiziksel network adaptörleri (uplinks), sanal makinelere hizmet veren **VM Port Group**'lar ve hostun kendi yönetimsel trafiği için kullandığı **VMkernel port**'lar. Makalenin merkezinde şu soru var: _"Bir ESXi hostunda birden fazla fiziksel NIC varken, farklı trafik türlerini (yönetim, vMotion, storage, VM trafiği) nasıl ayırmalı ve neden ayırmalıyız?"_

### Standart Switch'in Temel Yapısı

ESXi bir host olarak kurulduğunda, hypervisor varsayılan olarak bir **vSwitch0** oluşturur ve bu switch'i hem fiziksel dünyaya hem sanal dünyaya bağlar:

```
                          ┌─────────────────────────────────────┐
                          │           ESXi Host                 │
                          │                                     │
   VM1 ──┐                │   ┌─────────────────────────┐       │
   VM2 ──┼── VM Port ─────┼──▶│                          │       │
   VM3 ──┘    Group       │   │      vSwitch (Standard)  │       │
                          │   │                          │       │
   Management ────────────┼──▶│                          │◀──────┼── vmnic0 (fiziksel NIC)
   VMkernel Port           │   │                          │       │
                          │   └─────────────────────────┘       │
                          │              │                       │
                          │         vmnic1, vmnic2...             │
                          └─────────────────────────────────────┘
```

Gerçek prod ortamlarında bir fiziksel host neredeyse her zaman birden fazla network adaptörüne (NIC) sahiptir ve bunların hepsi bu sanal switch'e **uplink** olarak bağlanabilir.

### VM Port Group ile VMkernel Port Arasındaki Fark

**VM Port Group:**

* Sanal makineleri switch'e bağlamak için kullanılır.
* Kendi başına bir işletim sistemine veya IP konfigürasyonuna ihtiyaç duymaz — çünkü IP adresini, subnet bilgisini vs. bağlı olan **sanal makinenin kendi işletim sistemi** (örneğin Windows veya Linux guest OS) DHCP ile ya da statik olarak yönetir.
* Port group, trafiğin sadece "geçtiği" bir kapı gibi düşünülebilir.

**VMkernel Port:**

* ESXi hostunun **kendisinin** kullanması gereken servisler için var: host yönetimi (Management), **vMotion**, **Fault Tolerance (FT)**, IP tabanlı harici storage bağlantıları (iSCSI, NFS) gibi.
* Burada bir guest OS olmadığı için (host'un kendisi konuşuyor), VMkernel adaptörü **kendi TCP/IP stack'ine sahip** ve doğrudan IP adresi, subnet mask, gateway gibi bilgilerle konfigüre edilebilir hale gelir.
* Kısacası VMkernel, hypervisor'a "bir işletim sistemi olmadan da ağ üzerinde IP ile konuşabilme" yeteneği kazandırır.

Bu ayrımı özetlersek:

| Özellik           | VM Port Group            | VMkernel Port                      |
| ----------------- | ------------------------ | ---------------------------------- |
| Kimin trafiği     | Guest VM                 | ESXi host'un kendisi               |
| IP konfigürasyonu | Guest OS tarafından      | Doğrudan VMkernel arayüzünde       |
| Örnek kullanım    | Web sunucu VM'i, DB VM'i | Management, vMotion, iSCSI/NFS, FT |
| OS bağımlılığı    | Var (guest OS gerekir)   | Yok                                |

### Neden Fiziksel Adaptörleri Ayırmalısın?

**Tek NIC senaryosu (önerilmez):** Tüm VM trafiği + Management + vMotion + Storage trafiği **aynı fiziksel NIC** üzerinden geçiyorsa, bu adaptör üzerinde ciddi bir yük birikir. Sonuç: veri transferinde yavaşlama, storage I/O gecikmesi, vMotion sırasında kesintiler ve genel ağ hataları.

**Çoklu NIC senaryosu (önerilen):**

```
vmnic0  ─────▶  VM Port Group      (Sanal makine trafiği)
vmnic1  ─────▶  VMkernel Port      (Management + Storage)
vmnic2  ─────▶  VMkernel Port      (vMotion / FT — ayrı, izole trafik)
```

### İzole Switch'ler ve L2 Bağlantı Mantığı

Birden fazla vSwitch oluşturduğunda (örneğin vSwitch0 ve vSwitch1), bunlar **birbirinden tamamen izole**dir — tıpkı iki ayrı fiziksel switch gibi.

* vSwitch0'a bağlı VM'ler, vSwitch1'e bağlı VM'lerle **doğrudan konuşamaz.**
* Bu izolasyonu kırmanın tek yolu, her iki switch'in uplink'lerini (fiziksel NIC'lerini) **aynı fiziksel switch/aynı L2 segment** üzerinde buluşturmaktır. Yani vmnic0 ve vmnic1'i fiziksel olarak aynı switch'e, aynı VLAN'a bağlarsan, iki sanal switch arasında dolaylı bir yol açılmış olur.

### Production'da Nelere Dikkat Etmeli

* **NIC Teaming / Load Balancing Policy:** Bir port group'a birden fazla uplink atadığında, ESXi hangi trafiğin hangi NIC'ten çıkacağına "Route based on originating virtual port", "Route based on IP hash" gibi politikalarla karar verir. Bu, tek NIC'e yük binmesini engellemenin bir diğer katmanıdır.
* **VLAN Tagging (802.1Q):** Port group ve VMkernel port'lara VLAN ID atayarak, aynı fiziksel switch üzerinde bile trafik türlerini mantıksal olarak ayırabilirsin — bu, "aynı switch'e bağlarsan izolasyon biter" kuralına ince bir istisna sağlar.
* **Standard Switch → Distributed Switch (vDS) geçişi:** Anlatılan mimari Standard Switch (host-bazlı) seviyesinde. Çok node'lu ortamlarda vDS'e geçmek, tüm host'lar için merkezi ve tutarlı port group/VMkernel konfigürasyonu sağlar.

***

**ESXi'de ağ mimarisi kurarken, "kimin" trafik ürettiğini (guest VM mi, yoksa host'un kendisi mi) net ayırt etmen ve bu iki farklı trafik sınıfını mümkünse farklı fiziksel adaptörlere dağıtman gerekiyor.** VM Port Group ile VMkernel Port arasındaki ayrım kavramsal olarak basit görünse de, production'da performans, izolasyon ve güvenilirlik açısından doğrudan sonuç doğuran bir tasarım kararı. Ve son olarak: sanal switch mantığında kafan karıştığında, onu fiziksel bir switch gibi düşünmek her zaman işe yarayan bir zihinsel model.
