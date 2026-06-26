---
icon: user-hair
---

# User with Different Roles in Different Items

Sanallaştırma altyapılarında yetkilendirme (RBAC) kurgulanırken, sistem yöneticilerinin karşısına sıklıkla zorlu bir senaryo çıkar: Ekibinizdeki bir uzman, envanterin bir bölümünde tam yetkili bir yönetici iken, bir başka bölümünde sadece izleyici olmak zorundaysa ne yapacaksınız? Bu kişi için iki farklı hesap (Örn: `idris_admin` ve `idris_readonly`) açmak hem yönetim karmaşasına yol açar hem de log takibini kabusa çevirir.

VMware vCenter mimarisi, bu sorunu çözmek için son derece esnek bir "Multi-Role" yeteneği sunar. Bu makalede, tek bir kimliğin (Kullanıcının) vCenter hiyerarşisinde farklı nesnelere (Items) atandığında arayüzün buna nasıl tepki verdiğini ve bu esnekliğin saha operasyonlarındaki gücünü inceleyeceğiz.

**1. Görünmezlik Kalkanı: vCenter ve Datacenter Seviyesinde İzolasyon**

Sadece tek bir host üzerinde (Örneğin `Host-10`) yetkilendirdiğiniz bir kullanıcı sisteme giriş yaptığında, vCenter mimarisinin sıkı güvenlik duvarlarıyla anında yüzleşir.

Kullanıcı, envanter ağacında en üstteki vCenter veya Datacenter nesnelerine tıkladığında şu meşhur uyarıyı alır: _"You do not have permission to view this object or this object does not exist."_

Bu bir hata değil, mimarinin tam olarak tasarlanma şeklidir. Kullanıcının yetkisi en üstten (Root) Propagate edilmediği için, sistem ona yetkisi olmayan alanları hiç var olmamış gibi gösterir. Datastore menüsüne tıkladığında listeyi boş görür, ortamda `Host-20` adında başka bir sunucu olsa bile bunu kesinlikle göremez. Kullanıcının tüm dünyası, kendisine atanan o tek Host'tan ibarettir.

**2. Tam Yetki: `Host-10` Senaryosu**

Kullanıcı, yetkili olduğu `Host-10` donanımına tıkladığında arayüzdeki kilitler aniden açılır. Eğer bu kullanıcıya bu seviyede Administrator rolü atanmışsa, sadece bu Host sınırları içerisinde tam yetkili bir "Tanrı"ya dönüşür.

* Yeni bir Virtual Machine yaratabilir.
* Kendi hostu üzerinde Resource Pool oluşturabilir.
* Sunucuyu Maintenance Mode durumuna alıp yeniden başlatabilir (Reboot).
* Altında çalışan makinelerin Snapshot yönetimini yapabilir.

Buradaki mimari deha şudur: Kullanıcı `Host-10` seviyesinde bir yönetici olduğu için, bu Host'a yeni bir Sanal Makine kurarken vCenter'ın "Deploy Template" gibi merkezi yeteneklerini kullanabilir. Ancak sistem, bu şablonun sadece ve sadece `Host-10` içine kurulmasına izin verecektir.

**3.  Farklı Nesnelerde Farklı Roller (Multi-Role)**

İşte VMware yetkilendirme mimarisinin asıl esnekliği burada başlar. Aynı kullanıcıya, altyapıdaki farklı bir donanım veya klasör için tamamen farklı bir rol atayabilirsiniz.

Ana yönetici oturumuna geri dönüp, aynı kullanıcıyı bu kez `Host-20` üzerine eklediğinizi ve ona Read Only rolü atadığınızı varsayalım.

Kullanıcı kendi hesabıyla tekrar giriş yaptığında envanter ağacında artık iki farklı sunucu görecektir. Arayüzün dinamik tepkisi tam olarak şu şekilde çalışır:

* Kullanıcı `Host-10` üzerine tıkladığında; "Add Permission", "New Virtual Machine" veya "Power On/Off" gibi tüm operasyonel butonlar aktiftir.
* Aynı kullanıcı `Host-20` üzerine tıkladığında; ortamdaki makineleri görebilir, donanım kaynaklarını izleyebilir ancak tüm müdahale butonları (Snapshot alma, Power Off yapma vb.) pasif (Gri) duruma geçer. Çünkü `Host-20` üzerinde Propagate edilen yetkisi sadece okuma izninden ibarettir.

**4.  Kurumsal Faydaları**

Örnek Senaryo: Bir Veritabanı Yöneticisi (DBA), SQL sunucularının bulunduğu klasörde "Power User" rolüne sahip olabilir (Makineleri yeniden başlatabilir, Snapshot alabilir). Ancak aynı DBA, şirket içi web uygulamalarının bulunduğu klasörde sadece "Read Only" rolünde kalabilir. Böylece web sunucularının CPU ve RAM durumlarını inceleyebilir ancak onlara müdahale edemez.

Özetle; vCenter'ın RBAC yapısı, sistem yöneticilerine "Bir kullanıcıya sadece bir kimlik verilir ve o kimliğin tek bir yetkisi olur" gibi sığ bir kural dayatmaz. Ortamdaki her bir donanım parçası, klasör veya sanal makine için o hesaba yepyeni bir rol giydirebilirsiniz. Bu özellik, gereksiz hesap açılışlarını (Hesap enflasyonunu) engeller ve güvenlik denetimlerinde (Audit) işlemlerin tek bir hesaba kadar hatasızca izlenebilmesini sağlar.

***

### BONUS

Bir kullanıcıya sadece Host seviyesinde Administrator rolü vermek, Datastore (Veri Depolama) mimarisine bağlı olarak onu yarı yolda bırakabilir. Bu sorunun cevabı, vCenter hiyerarşisinde Datastore'un nerede durduğuna (Fiziksel mi, Paylaşımlı mı olduğuna) göre ikiye ayrılır:

#### 1. Senaryo: Local Datastore

Eğer `Host-10` sunucusunun üzerinde fiziksel olarak takılı kendi diskleri (Local Storage) varsa ve bu alan `datastore1` olarak formatlanmışsa, vCenter hiyerarşisinde bu Local Datastore doğrudan Host'un bir "Child Item"ı (Alt Öğesi) olarak kabul edilir.

Siz kullanıcıya Host seviyesinde Administrator verip `Propagate to Children` (Alt öğelere uygula) dediğinizde, bu tam yetki doğrudan Host'un içindeki Local Datastore'a da akar. Bu sayede kullanıcı, kendi hostunun yerel disklerinde sorunsuzca yepyeni bir Virtual Machine yaratabilir.

#### 2. Senaryo: Shared Datastore (Paylaşımlı Depolama - SAN/NAS)

Kurumsal ortamlarda makineler yerel disklere değil, dışarıdan bağlanan (iSCSI, FC, NFS) ortak depolama alanlarına (Shared Datastore) kurulur. Shared Datastore nesneleri, vCenter hiyerarşisinde Host'un altında YAŞAMAZLAR. Doğrudan vCenter veya Datacenter seviyesinde, ayrı bir nesne olarak dururlar. Host sadece onlara bir kabloyla bağlıdır.

Bu durumda ne olur?

1. Kullanıcı `Host-10` üzerinde tam yetkilidir. Yeni Virtual Machine sihirbazını açar.
2. İşlemci, RAM gibi donanımları (Compute) sorunsuzca yapılandırır.
3. Ancak disk oluşturma (Storage) adımına gelip, kurulum yeri olarak ortak bir SAN Datastore seçtiğinde ve Finish'e bastığında tahmin ettiğimiz şey olur: Sistem işlemi iptal eder ve _"Datastore Allocate Space - Permission Denied"_ hatasını fırlatır.

Çünkü `Host-10` üzerinden aşağıya doğru akan (Propagate) yetki, yandaki bağımsız Shared Datastore nesnesine ulaşamaz.

#### Çözüm Nasıl Kurgulanır?

Sistem yöneticileri bu "Multi-Role" mimarisini gerçek sahada kurgularken, tam olarak bu mantık hatasını önlemek için iki aşamalı yetki atarlar:

1. Host Seviyesinde Yetki: Kullanıcı `Host-10` üzerine eklenir ve Administrator rolü verilir. (Böylece donanımı yönetir, Resource Pool açar, Snapshot alır).
2. Datastore Seviyesinde Yetki: Aynı kullanıcı, vCenter'ın Datastore sekmesine gidilerek ilgili ortak depolama alanının (Örn: `LUN_01`) üzerine de eklenir. Burada ona Administrator değil, sadece diske yazabilmesi için Datastore Consumer (veya sadece Allocate Space yetkisi barındıran Custom Role) atanır.

Özetle; Sadece Host üzerinde tam yetkili olmak, dışarıdaki bir Datastore'dan disk alanı işgal etmeye yetmez. Sistemin kusursuz çalışması için Compute (Host) ve Storage (Datastore) yetkilerinin, hiyerarşide bulundukları doğru seviyelerden ayrı ayrı delege edilmesi zorunludur.
