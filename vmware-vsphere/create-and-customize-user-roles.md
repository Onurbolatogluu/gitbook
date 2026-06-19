---
icon: user-question
---

# Create and Customize User Roles

Daha önceki yazılarımızda, ESXi sunucularına herkesin `root` parolasıyla girmesinin ne kadar tehlikeli olduğundan ve kullanıcılara Read Only gibi kısıtlı roller atamanın öneminden bahsetmiştik. Ancak gerçek dünya operasyonlarında yetki yönetimi siyah ve beyazdan (Tam Yetki vs. Hiçbir Yetki) ibaret değildir.

Ortamınıza yeni katılan bir sanallaştırma uzmanına sadece "Sanal Makineleri ve Veri Depolama (Datastore) alanlarını yönetme" görevi vermek istediğinizi varsayalım. ESXi'ın varsayılan rolleri size sadece üç seçenek sunar: Administrator (Tüm sistemi silip süpürebilir), Read Only (Sadece izleyebilir) veya No Access (Sisteme giremez).

İşte bu noktada VMware mimarisinin en esnek güvenlik katmanı olan Özel Roller (Custom Roles) devreye girer. Bu makalede, sistemdeki varsayılan rollerin kısıtlamalarından kurtulup, kendi "Terzi İşi" yetki şablonlarımızı nasıl oluşturacağımızı ve bu izinlerin sistemde nasıl dağıldığını (Propagation) inceleyeceğiz.

**1. Mikro Yetkilendirme: İhtiyaca Özel Rol Yaratmak**

Kullanıcılara doğrudan "Yönetici" pelerini giydirmek yerine, onlara sadece ihtiyaç duydukları araçları içeren özel bir alet çantası hazırlamamız gerekir. ESXi arayüzünde `Manage > Security & users > Roles` sekmesi, kendi alet çantamızı (Rolümüzü) yaratacağımız yerdir.

Yeni bir rol eklerken (Add Role), karşınıza sanallaştırma ortamındaki tüm ince ayarların listelendiği devasa bir Ayrıcalıklar (Privileges) ağacı çıkar.

Örneğin; sadece makineleri ve depolama alanını yönetecek bir uzman için bu listeden sadece Virtual Machine ve Datastore seçeneklerini işaretleriz. Ağa (Network) veya Güvenlik Sertifikalarına (Certificates) dokunma yetkisi vermeyiz.

İsimlendirme Standardı: Oluşturulan bu özel role `VM_Datastore_Admin` gibi ne işe yaradığını anında belli eden açıklayıcı bir isim vermek, gelecekteki yönetim karmaşasını önlemenin en temel kuralıdır.

**2. Görünmez Bağımlılıklar: Sistem (System) Yetkileri Neden Kendi Kendine Eklenir?**

Kendi oluşturduğunuz role (Örneğin sadece VM ve Datastore yetkileri seçili) bir kullanıcıyı atarken çok ilginç bir mimari davranışla karşılaşırsınız: Seçmediğiniz halde "System" ayrıcalıklarının bir kısmının varsayılan olarak (otomatik) işaretlendiğini görürsünüz.

Bu bir hata değil, VMware'in kendi iç mimarisini koruma yöntemidir. Bir kullanıcının sanal makine oluşturabilmesi (Virtual Machine privilege), arka planda ESXi'ın sistem kaynaklarına dokunmasını, donanım kimlikleri üretmesini ve bellek (RAM) ayırmasını gerektirir. Sistem, ana eylemin gerçekleşebilmesi için mecburi olan bu "Altyapısal Sistem Görevlerini" asıl yetkinin yanına zorunlu olarak ekler. Sistem yöneticisi olarak bu otomatik eklenen yetkileri görüp endişelenmemeniz, bunun mimarinin doğal bir gereksinimi olduğunu bilmeniz önemlidir.

**3. Mimari Bir Kavram: Kalıtım (Propagate to all children)**

Bir kullanıcıya "VM\_Datastore\_Admin" rolünü verdiğiniz ekranda (Permissions bölümü) her zaman işaretli gelen kritik bir kutucuk vardır: Propagate to all children (Tüm alt öğelere uygula).

Bu özellik, VMware ortamlarındaki nesne hiyerarşisinin (Ağaç Yapısının) temel taşıdır.

* İşaretli Olursa: Bir ESXi Host üzerinde kullanıcıya yetki verdiğinizde, bu yetki Host'un şemsiyesi altındaki tüm "çocuk nesnelere" (içindeki tüm mevcut ve gelecekte kurulacak sanal makinelere, ağ kartlarına vb.) şelale gibi akar ve uygulanır.
* İşaret Kaldırılırsa: Kullanıcı sadece tanımladığınız o en üst kabukta (sadece Host'un arayüz ayarlarında) yetkili olur, ancak Host'un içindeki hiçbir sanal makineye dokunamaz.

Bu özellik özellikle vCenter gibi devasa yapılara (Datacenter > Cluster > Host > VM hiyerarşisi) geçildiğinde hayat kurtarır. En tepeden verdiğiniz bir yetkinin tüm alt klasörlere otomatik yayılmasını sağlayan mekanizma budur.

> 💡 Uzman İpucu: Rol Tanımlama ve "En Az Yetki" (Least Privilege) Prensibi
>
> Siber güvenlik mimarisinde altın kural şudur: _Bir kullanıcıya, görevini yapabilmesi için gereken asgari yetkiden bir fazlası dahi verilmemelidir._ >
>
> Yeni bir rol oluştururken "Bu ayrıcalık da ileride lazım olur" diyerek fazladan kutucuk işaretlemek, o hesabın ele geçirilmesi durumunda oluşacak hasarın büyümesine yol açar. Ağ yönetimi ayrı bir rol (`Network_Admin`), sanal makine operasyonları ayrı bir rol (`VM_Operator`) olarak parçalara bölünmeli ve kullanıcılara mikro roller atanmalıdır. Bu sayede hem güvenlik maksimuma çıkarılır hem de log (olay günlüğü) takibinde "Kimin tam olarak ne yapmaya yetkisi vardı?" sorusunun cevabı netleşir.
