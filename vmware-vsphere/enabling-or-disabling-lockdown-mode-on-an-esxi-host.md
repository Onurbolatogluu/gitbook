---
icon: lock-open
---

# Enabling or Disabling Lockdown Mode on an ESXi Host

Önceki makalelerimizde ESXi altyapısının Security Profile (Güvenlik Profili) üzerinden port yönetimini (Firewall) ve servislerin (DCUI, ESXi Shell) nasıl durdurulup başlatılacağını detaylıca incelemiştik. Ancak siber güvenlikte "Defense in Depth" felsefesi gereği, sadece kapıları kapatmak yetmez; yönetim trafiğini tamamen tek bir merkeze kanalize etmek gerekir.

İşte tam bu noktada VMware mimarisinin en agresif ve en etkili güvenlik katmanı olan Lockdown Mode devreye girer. Bu makalede, Lockdown Mode seviyelerini, vCenter bağımlılığını ve canlı bir sistemde aktif edildiğinde arayüzlerin bu duruma nasıl anlık tepki verdiğini analiz edeceğiz.

**1. Lockdown Mode Seviyeleri: "Normal" ve "Strict" Ayrımı**

Lockdown Mode'un temel amacı, ESXi Host'lara dışarıdan (doğrudan IP adresi yazılarak) yapılan yönetim bağlantılarını kesmek ve tüm kontrolü vCenter Server üzerine almaktır. Bu özellik varsayılan olarak Disabled durumdadır. Ancak aktif edildiğinde sistem yöneticisine iki farklı yol sunar:

* Normal Lockdown Mode: Bu seviye aktif edildiğinde, Host'a web tarayıcısı üzerinden doğrudan (IP adresiyle) erişim tamamen engellenir. Yönetim işlemleri sadece vCenter üzerinden veya sunucunun başındaki fiziksel konsoldan (DCUI) yapılabilir.
* Strict Lockdown Mode: Güvenliğin en üst noktasıdır. Bu seviyede vCenter haricindeki tüm kapılar mühürlenir. Önceki makalede bahsettiğimiz DCUI servisi sistem tarafından durdurulur. Sunucunun başına gidip klavyeden F2'ye basan yetkili bir sistem yöneticisi bile _"You do not have authorization to access this host"_ hatasıyla karşılaşır.

**2. Neden vCenter Şart?**

Lockdown Mode, sadece vCenter tarafından yönetilebilen bir özelliktir.

Eğer bir ESXi Host'u vCenter'dan koparır (Disconnect) ve bağımsız (Standalone) hale getirirseniz, Security Profile altındaki Lockdown Mode menüsü anında pasif (Gri) duruma geçer. Sistem bu özelliği kendi başına yönetmenize izin vermez.

Eğer sunucunun başındaysanız ve DCUI (Fiziksel Konsol) üzerinden Lockdown Mode'u açmak isterseniz, sistem size sadece Normal seçeneğini sunar. Strict seçeneği fiziksel konsol menülerinde bilerek gizlenmiştir.

Bunun sebebi basit bir mantık hatasını önlemektir: Eğer fiziksel konsol üzerinden "Strict" modunu seçebilseydiniz, bu işlem anında DCUI servisini durduracağı için kendi bindiğiniz dalı kesmiş olurdunuz ve anında sistemden atılırdınız. VMware mühendisleri bu operasyonel kazayı önlemek için Strict modunu sadece vCenter web arayüzünden tetiklenebilir hale getirmiştir.

**3. Anlık Uygulama ve "Kick-Out" (Sistemden Atılma) Etkisi**

Lockdown Mode'un en çarpıcı özelliklerinden biri, konfigürasyonun uygulanma hızıdır. Sistem hiçbir gecikme yaşatmaz.

* Erişim Reddi: Özellik aktif edildiği an, tarayıcıdan Host'un IP adresine giden bir kişi en yetkili root şifresini bilse dahi _"Permission to perform this operation was denied"_ hatasını alır.
* Açık Oturumların Düşmesi: Eğer bir yönetici halihazırda Host'un web arayüzüne (Host Client) bağlıyken, başka bir yönetici vCenter üzerinden Host'u Lockdown Mode'a alırsa; Host Client üzerindeki aktif kullanıcının ekranında anında bir _"Unhandled Exception"_ belirir. Kullanıcı bir sayfayı yenilediği veya bir butona bastığı anda oturumu zorla kapatılır ve sistemden atılır.

**4. Neden Lockdown Mode Kullanmalıyız?**

Kurumsal veri merkezlerinde, yönetim trafikleri her zaman izlenebilir olmak zorundadır. Ortamınızda vCenter varken yöneticilerin Host'lara doğrudan IP adresleriyle bağlanması; logların dağılmasına, RBAC kurgusunun delinmesine ve en önemlisi Host'un root hesabının "Brute Force" şifre kırma saldırılarına açık hale gelmesine neden olur.

Lockdown Mode'u aktif etmek; saldırganların doğrudan Host'lara saldırmasını engeller, tüm yönetimsel gücü, şifre politikalarının ve alarm mekanizmalarının çok daha güçlü olduğu vCenter kalesine hapseder. Profesyonel bir sanallaştırma mimarisinin vazgeçilmez standardı budur.

