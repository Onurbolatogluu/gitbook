---
icon: arrow-right-to-bracket
---

# Log in With a Custom User Roles

Önceki makalelerimizde, ESXi sunucusuna doğrudan kök (root) erişimi vermenin tehlikelerini tartışmış ve 'En Az Yetki Prensibi'ne (Least Privilege) dayanarak bir senaryo kurgulamıştık. Bu senaryoda, sadece sanal makineleri ve veri depolarını (Datastore) yönetebilecek özel bir rol (Custom Role) oluşturmuş ve bu kısıtlı yetkileri Mike isimli kullanıcı profiline atamıştık.

Teoride kurduğumuz bu "Terzi İşi" yetki şablonunun, gerçek dünya operasyonlarında (sahada) nasıl davrandığını görmek, bir sistem yöneticisinin kurguladığı güvenlik mimarisine güvenmesini sağlayan en önemli adımdır. Bu makalede, oluşturduğumuz özel kullanıcının arayüzdeki hareket alanını test edecek ve VMware izin mimarisinin görünmez duvarlarına nasıl çarptığını inceleyeceğiz.

**1. "Sanal Makine Yönetmek" Ne Demektir?**

Mike kullanıcısıyla ESXi arayüzüne giriş yaptığınızda, ilk bakışta her şey normal görünür. Kullanıcının sanal makine yetkisi (Virtual Machine Privilege) olduğu için, içgüdüsel olarak yeni bir makine kurabileceğini düşünebilir.

Tıpkı Read-Only yetkisinde olduğu gibi, Mike "Create/Register VM" sihirbazını başlatabilir, isim verebilir, donanım seçebilir. Ancak son adımda "Finish" (Bitir) dediği an sistem şu hatayı fırlatır: _"VM configuration was rejected."_

Neden? Çünkü biz Mike'a "Sanal Makine Seçeneklerini (Options)" yönetme yetkisi verdik, "Sistemden yeni disk ve kaynak yaratma (Provisioning)" yetkisi vermedik. Yeni bir sanal makine kurmak, sadece bir klasör açmak değil; depolama alanından (Datastore) kalıcı olarak yer ayırmak ve Host'un CPU/RAM limitlerini kalıcı olarak rezerve etmektir. Mikro yetkilendirme mimarisi, bu ince çizgiyi kusursuzca ayırt eder.

**2. Mike Neleri Yapabilir?**

Yeni makine kuramayan bu kullanıcının asıl operasyonel gücü nerede yatar? Zaten mevcut olan ve kendisine delege edilmiş makineler üzerinde:

* Güç Operasyonları (Power Ops): Kapalı bir makineyi açabilir (Power On), açık bir makineyi kapatabilir.
* Donanım Modifikasyonu (Edit Settings): Mevcut bir makinenin RAM'ini artırabilir, CPU ekleyebilir veya ağ kartını değiştirebilir.
* Yedekleme ve Geri Dönüş (Snapshots): Kritik bir yama (Patch) geçişi öncesinde makinenin anlık görüntüsünü (Snapshot) alabilir, işlemi tamamladığında silebilir.
* Veri Deposu Erişimi (Datastore): "Datastore" yetkisini de verdiğimiz için, Datastore'ların içine girebilir (Browse), log dosyalarını okuyabilir veya ISO dosyalarını yükleyebilir.

**3. Güvenlik Duvarlarına Çarpmak**

Sistemin en güçlü savunma mekanizması, Mike'ın "Manage" (Yönetim) sekmesindeki çaresizliğidir.

Kendi yetkilerinden fazlasını merak eden bu kullanıcı, `Security & users` sekmesine girip yeni bir kullanıcı (Örn: bir arka kapı hesabı) açmaya çalıştığında veya kendine yeni bir rol (Örn: Administrator rolü) tanımlamaya çalıştığında sistem bu isteği anında bloke eder: _"The permission to perform this operation was denied."_

Daha da önemlisi, Mike `Permissions` sekmesine tıklasa bile sistemde `root` kullanıcısının veya diğer yöneticilerin varlığını dahi göremez. Kendi dünyasına hapsedilmiş, sadece kendisine verilen oyuncaklarla (Sanal makineler ve Datastore) oynayabilen güvenli bir operatör haline gelmiştir.

> Kendi rollerinizi yaratmak (Custom Roles) güvenlik açısından harikadır, ancak operasyonel bir körlüğe (Troubleshooting kabusuna) dönüşme riski taşır.
>
> Örneğin; bir kullanıcı size _"Makinenin Snapshot'ını alamıyorum"_ diye geldiğinde, sorunun diskte yer kalmamasından mı, dosya kilitlenmesinden (File Lock) mi yoksa kullanıcının rolünde ilgili kutucuğun işaretlenmemiş olmasından mı kaynaklandığını anlamak saatler sürebilir.
>
> Bu nedenle kurumsal pratiklerde her yeni rol (Custom Role) canlıya alınmadan önce mutlaka "UAT" (User Acceptance Testing) sürecinden geçirilmelidir. İlgili ekibin günlük hayatta yaptığı 5 temel işlem (Açma, kapama, snapshot alma, konsola bağlanma, donanım ekleme) test ortamında tek tek denenmeli, "Access Denied" hataları yakalanıp rol revize edildikten sonra Production ortamına uygulanmalıdır.&#x20;
