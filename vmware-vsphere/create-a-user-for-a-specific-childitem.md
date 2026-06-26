---
icon: user-group
---

# Create A User for A Specific ChildItem

Sanallaştırma altyapılarında yönetim sadece sanal makineleri açıp kapatmaktan ibaret değildir. "Hangi ekibin, hangi kaynakları göreceği ve yöneteceği" sorusu, mimari güvenliğin temelini oluşturur. Örneğin; 50 farklı ESXi host barındıran devasa bir yapınız (vCenter) olduğunu varsayalım. Ekibinize yeni katılan bir uzman, sadece belirli bir departmanın veya müşterinin sunucularından (Örneğin sadece `Host-10` cihazından) sorumlu olacaksa, ona vCenter'ın en tepesinden (Root) yetki vermek tam bir güvenlik felaketidir.

Bu makalede, vCenter'ın "Granüler Yetki" mimarisini, "Child Item" konseptini ve İzin Kalıtımı (Propagation) mantığını inceleyeceğiz.

**1. En Üstten Aşağıya (Top-Down) Yetki Akışı: `Propagate to Children` Mimarisi**

Daha önceki makalemizde "Jack" isimli bir kullanıcıyı doğrudan vCenter Server nesnesi üzerinde yetkilendirmiştik. vCenter'ın katı hiyerarşik yapısı (Ağacı) şu şekildedir: vCenter > Datacenter > Cluster > Host > Virtual Machine.

Siz en tepedeki vCenter nesnesine bir kullanıcı atayıp "Propagate to Children" (Tüm alt öğelere uygulansın) kutucuğunu işaretlediğinizde, bu kullanıcının kimliği bir şelale gibi sistemin en alt klasörüne (sanal makinelere) kadar akar.

Jack kullanıcısı; `Host-10` sunucusunda da vardır, `Host-20` sunucusunda da vardır, veri depolarında da vardır. Çünkü Jack, şelalenin kaynağında (vCenter Root) yetkilendirilmiştir. Bu özellik sistem yöneticilerinin yüzlerce makineye tek tek yetki girmesini engeller, ancak yetkinin çapı çok geniştir.

**2. Nokta Atışı Yetkilendirme: Sadece Belirli Bir Host'a Görev Atamak**

Peki ya "Idris" isimli kullanıcının, tüm vCenter'ı değil, sadece `Host-10` cihazını ve o cihazın içindeki sanal makineleri (Child Items) yönetmesini istiyorsanız ne yapmalısınız?

İşte bu noktada yetkilendirmeyi en tepeden başlatmak yerine, doğrudan hedefin kendisine odaklanmalısınız:

1. vCenter envanter ağacında doğrudan `Host-10` cihazına tıklanır.
2. `Permissions` (İzinler) sekmesine girilir.
3. "Idris" kullanıcısı seçilir ve ona örneğin "Administrator" rolü atanır.
4. "Propagate to Children" kutusu işaretli bırakılır.

Sonuç (Mimari İzolasyon): Bu işlemin ardından Idris kullanıcısı, yalnızca kendi sorumluluk alanına hapsedilir. "Propagate" mantığı her zaman yukarıdan aşağıya (Top-Down) çalışır, aşağıdan yukarıya (Bottom-Up) ASLA çalışmaz.

Idris, `Host-10` sunucusuna tıkladığında tam yetkili bir yöneticidir. `Host-10`'un içindeki sanal makinelere (Child Items) de hükmedebilir. Ancak hiyerarşide bir üst kademeye (Datacenter veya vCenter Root) çıkmaya çalıştığında sistemi göremez. Aynı şekilde ağaçtaki diğer dallara (`Host-20` veya onun içindeki makinelere) tıkladığında, buralarda hiçbir yetkisi olmadığı için "Erişim Reddedildi" mesajı alır veya bu klasörler ona tamamen görünmez olur.

> 💡 İpucu: Multi-Tenant Yapıların ve Sorun Gideriminin Sırrı
>
> Bu özellik sadece basit bir güvenlik katmanı değil, Cloud Service Provider mimarisinin kalbidir. Aynı vCenter'ı kullanan farklı müşterileriniz (veya departmanlarınız) varsa, her müşteri için ayrı bir "Resource Pool" veya "Folder" oluşturur ve yetkiyi tam o klasör seviyesinde atarsınız. Bu sayede "A Müşterisi", "B Müşterisinin" sunucularını göremez bile.
>
> Sorun Giderme: `Permissions` ekranında gezinirken, bir kullanıcının yetkisinin hangi kaynaktan geldiğini anlamak her zaman kolay olmayabilir. `Defined In` (Tanımlandığı Yer) adlı sihirli sütuna dikkat edin. Bir sanal makineye tıkladığınızda; "Jack" kullanıcısının yanında `Defined in: vCenter` (Tepeden akıp gelmiş) yazdığını, ancak "Idris" kullanıcısının yanında `Defined in: Host-10` (Sadece o donanımdan akıp gelmiş) yazdığını görürsünüz. Mimari tasarımı okumak, tam olarak bu detaylarda gizlidir.
