---
icon: file-lines
---

# Create a Customization Specification for Windows in the vSphere Client

Kurumsal sanallaştırma ortamlarında, bir sanal makinenin klonlanması sadece disklerin kopyalanması işleminden ibaret değildir. Klonlama sonrasında ağda yaşanabilecek IP, SID ve Hostname çatışmalarını önlemek için işletim sisteminin kimliğinin sıfırlanması hayati bir zorunluluktur.

İşte vCenter'ın bu kimlik sıfırlama operasyonunu insan eli değmeden, tam otomatik bir şekilde yapabilmesi için bir "Cevap Anahtarına" ihtiyacı vardır. VMware literatüründe bu cevap anahtarına Customization Specification (Özelleştirme Şablonu) adı verilir.

Aşağıda, sıfırdan bir Windows Customization Specification dosyasının nasıl inşa edildiğini ve sihirbazdaki adımların kaputun altında hangi mimari kararlara karşılık geldiğini adım adım inceleyeceğiz.

**1. Şablonun Temelini Atmak: İşletim Sistemi Kilidi ve İsimlendirme**

Kütüphanenizde "Yeni Bir Spesifikasyon Yarat" (Create New Specification) dediğinizde, ilk karşılaşacağınız durum hedef işletim sisteminin (Target OS) kilitli olmasıdır.

* Mimari Kilit: Eğer klonlama işlemine bir Windows makinesi üzerinden başladıysanız, vCenter işletim sistemi türünü otomatik olarak "Windows" seçer ve bunu değiştirmenize izin vermez. Çünkü Linux için hazırlanan bir cevap anahtarı (örneğin bash scriptleri), Windows bir klona uygulanamaz.
* Şablon İsimlendirmesi: Hazırladığınız şablona vereceğiniz isim çok kritiktir. İleride ortamınızda onlarca şablon olacağı için, `SQL_2012_Standart_Kurulum` veya `HR_Database_Sablonu` gibi net ve açıklayıcı isimlendirmeler yapmak, doğru makineye doğru kimliği basmak açısından büyük önem taşır.

**2. Kurumsal Kimlik ve Hostname Stratejisi**

Şablonun en kritik aşaması, yeni doğacak olan sunucunun ağdaki adının (Computer Name/Hostname) nasıl belirleneceğine karar verilen adımdır. vCenter burada sistem yöneticilerine üç farklı strateji sunar:

1. Use the Virtual Machine Name (Sanal Makine Adını Kullan): En sık tercih edilen pratik yöntemdir. Klonlama sihirbazının en başında makineye vCenter arayüzünde hangi ismi verdiyseniz (Örn: `SQL-Srv-04`), Windows'un içine de o ismi Hostname olarak gömer.
2. Enter a name (Statik İsim Gir): Buraya tek bir isim yazarsanız, bu şablonu her kullandığınızda makinenin Hostname'i o olur. Ancak bu şablonu birden fazla makine için kullanmak isterseniz çatışma yaratacağı için çok önerilmez.
3. Enter a name in the Clone/Deploy Wizard (Sihirbazda Sor): Şablon mimarisinin en esnek ve en güçlü seçeneğidir. Eğer bir şablonu bir "altın standart" olarak hazırlayıp gelecekteki tüm Windows kurulumlarında kullanmak istiyorsanız, bu seçeneği işaretlersiniz. Bu sayede, şablonu her kullandığınızda vCenter klonlama ekranında size "Bu yeni makinenin Hostname'i ne olsun?" diye özel bir kutucuk çıkartır ve her dağıtımda farklı bir isim verebilirsiniz.

**3. Güvenlik ve Lisanslama Otomasyonu**

Windows makinelerin aktivasyonu ve yerel güvenliği de bu şablon üzerinden otomatize edilir:

* Product Key: Kurumunuza ait KMS (Key Management Service) veya Volume License anahtarını bu adıma bir kez girersiniz. Bu şablondan doğan binlerce makine, açılır açılmaz ağdaki lisans sunucunuzla konuşup kendini otomatik aktive eder.
* Administrator Password: Yeni makinenin yerel "Administrator" hesabı için karmaşık bir şifre belirlenir. İsteğe bağlı olarak, makine Sysprep sonrası ilk kez açıldığında otomatik olarak (Auto-logon) bu hesapla masaüstünü açması sağlanabilir.

{% hint style="info" %}
💡 Uzman İpucu: Klonlama Sonrası Parola Yönetimi ve Sysprep Davranışı

_"Klonladığım kaynak makinede zaten bildiğim bir parola var, neden şablon (Customization Spec) oluştururken tekrar parola girmek zorundayım?"_ diye düşünebilirsiniz.

Bunun mimari sebebi; Sysprep aracının işletim sistemini OOBE (Kutudan Yeni Çıkmış) durumuna getirmesidir. Bu güvenlik sıfırlaması sırasında, yerel Administrator hesabı da sıfırlanır. Eğer şablon üzerinde bir parola belirlemezseniz, yeni sanal makine ilk açılışında kurulum ekranında donup kalacak ve sizden manuel olarak klavyeyle bir parola girmenizi bekleyecektir. Bu durum, vCenter'ın "el değmeden tam otomatik kurulum" felsefesini bozar.

Çözüm çok basittir: Eski parolanızla çalışmaya devam etmek istiyorsanız, şablondaki parola alanına orijinal sunucudaki parolanızın birebir aynısını yazmanız yeterlidir. vCenter bu parolayı makine açılırken otomatik olarak işletim sistemine enjekte edecektir.

_(Not: Bu sıfırlanma durumu sadece Windows'un kendi içindeki Yerel (Local) Administrator hesabı için geçerlidir. Sunucu bir Active Directory Domain'ine katılıyorsa, Domain Admin hesaplarınızın yetkileri ve parolaları bu süreçten etkilenmez.)_
{% endhint %}

**4. Ufuk Açıcı Özellik: "Run Once" (Bir Kez Çalıştır) Scriptleri**

Sihirbazın sonlarına doğru karşınıza çıkan Run Once Commands ekranı, ileri seviye sistem yöneticilerinin en çok kullandığı otomasyon alanıdır.

Sysprep işlemi bitip makine yeni kimliğiyle ilk kez açıldığında, vCenter burada yazdığınız komutları makinenin komut satırında (CMD) sırayla, bir defaya mahsus olarak çalıştırır.

Bu alana neler yazılabilir?

* `netsh advfirewall set allprofiles state off` (Test ortamları için Firewall'u kapatma komutu).
* Özel bir Windows Servisini başlatma komutları.
* Antivirüs agent'ını tetikleme veya ortamdaki bir dosya sunucusundan otomatik konfigürasyon (config) dosyası çeken powershell betikleri.

Özetle; Customization Specification, sadece IP ve İsim değiştiren basit bir araç değildir. İşletim sisteminin kurulumundan lisanslanmasına, yerel şifre politikasından açılış scriptlerine kadar her şeyi tek bir pakette toplayan "Kurumsal Bir Fabrika Ayarıdır". Bu şablonu bir kez kusursuz hazırladığınızda, geriye kalan tek şey arkanıza yaslanıp vCenter'ın onlarca sunucuyu sıfır insan hatasıyla dakikalar içinde canlıya almasını izlemektir.
