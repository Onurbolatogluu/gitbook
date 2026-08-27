---
icon: folder-arrow-right
---

# Export and Restore Distributed Switch Configuration

Distributed Switch geçişini tamamladığınızda elinizde değerli bir şey vardır: tüm cluster'ın ağ mimarisini tanımlayan merkezi bir yapılandırma. Port group'lar, VLAN'lar, güvenlik politikaları, teaming ve failover kuralları, uplink tanımları — hepsi tek bir nesnede toplanmıştır.

Bu merkezileşmenin bir de diğer yüzü vardır: yanlış bir değişiklik de artık tek noktadan tüm ortama yayılır. Bir port group'un VLAN'ını yanlış girmek ya da yanlışlıkla bir nesneyi silmek, onlarca host'taki yüzlerce iş yükünü aynı anda etkileyebilir.

vDS bu riske karşı yerleşik bir güvence sunar: **yapılandırmayı dosyaya aktarma (export) ve geri yükleme (restore)**. Bu makalede bu mekanizmanın nasıl çalıştığını, ne zaman kullanılacağını ve production ortamlarında nasıl bir disipline bağlanması gerektiğini ele alıyoruz.

### Export Neyi Kaydeder?

Export işlemi, vDS'in **yapılandırmasını** bir ikili (binary) dosyaya yazar. Bu dosya şunları içerir:

* Distributed switch'in kendi ayarları: adı, sürümü, MTU, uplink sayısı, discovery protokolü
* Tüm **distributed port group** tanımları: isimler, VLAN yapılandırması, port binding ve allocation ayarları
* **Politikalar:** güvenlik (Promiscuous Mode, MAC Address Changes, Forged Transmits), teaming ve failover, traffic shaping, monitoring
* Network I/O Control ayarları

Neyi **içermediğini** bilmek de aynı derecede önemlidir:

* Host üyelikleri ve fiziksel adaptör (vmnic → uplink) eşlemeleri kaydedilmez.
* VM'lerin hangi port group'a bağlı olduğu bilgisi tutulmaz.
* VMkernel portlarının IP yapılandırması dosyada yer almaz.

Yani export, ağın **şablonunu** korur; host'a özgü fiziksel eşlemeleri ve iş yükü bağlantılarını değil. Bu ayrımı bilmek, restore sonrası neyin otomatik geleceğini neyin elle yeniden kurulacağını doğru planlamanızı sağlar.

### Yapılandırmayı Export Etme

İşlem birkaç adımdan oluşur:

1. vCenter envanterinde vDS'e sağ tıklayın.
2. **Settings → Export Configuration** seçin.
3. Kapsamı belirleyin:
   * **Distributed switch and all port groups** — önerilen seçenek
   * **Distributed switch only** — yalnızca switch nesnesi, port group'lar hariç
4. Anlamlı bir açıklama girin.
5. Dosyayı indirip güvenli bir konuma kaydedin.

Kapsam seçiminde kural nettir: **her zaman port group'lar dahil export alın.** Port group'lar olmadan alınan bir yedek, ağın asıl işlevsel tanımını (VLAN'lar, politikalar, hangi trafiğin nerede aktığı) içermez ve kurtarma senaryosunda işinize yaramaz. "Switch only" seçeneği yalnızca çok özel durumlar içindir.

Açıklama alanını da boş geçmeyin. Aylar sonra elinizde birkaç dosya olduğunda "hangisi neydi" sorusunun cevabı burada olacaktır — tarih, değişikliğin sebebi ve ortam bilgisi yazmak iyi bir alışkanlıktır.

### Restore ve Import: İki Farklı İşlem

Geri yükleme tarafında birbirine benzeyen ama farklı sonuçlar doğuran iki seçenek vardır. Karıştırılması sık yaşanan bir hatadır:

#### Restore Configuration

Mevcut bir vDS'in yapılandırmasını, dosyadaki haline **geri döndürür**. Switch nesnesi yerinde kalır; ayarları dosyadan gelen değerlerle değiştirilir.

Kullanım senaryosu: yapılandırmada hatalı bir değişiklik yapıldı ve önceki duruma dönmek isteniyor.

İki kapsam seçeneği vardır:

* **Restore distributed switch and all port groups:** Switch ve port group'ların tamamı dosyadaki haline döner. Dosyada bulunmayan, sonradan oluşturulmuş port group'lar **silinir** — bu nedenle export tarihinden sonra yapılan meşru eklemeleri kaybetmemek için dosyanın güncelliğini kontrol edin.
* **Restore distributed switch only:** Yalnızca switch seviyesindeki ayarlar döner, port group'lara dokunulmaz.

#### Import Configuration

Dosyadaki tanımdan **yeni bir vDS oluşturur**. Mevcut switch'e dokunmaz.

Kullanım senaryoları:

* vDS tamamen silindi ve sıfırdan yeniden oluşturulması gerekiyor
* Aynı ağ mimarisi başka bir datacenter veya vCenter'da kurulacak
* Test/DR ortamında production yapılandırmasının birebir kopyası isteniyor

Import sırasında **"Preserve original distributed switch and port group identifiers"** seçeneği karşınıza çıkar. Bu kritik bir ayardır:

* **İşaretlerseniz:** Nesneler orijinal kimlikleriyle (UUID) oluşturulur. VM'ler ve host'lar eski bağlantılarını tanır — silinen bir switch'i yerine koymak için doğru seçim budur.
* **İşaretlemezseniz:** Yeni kimliklerle bir switch oluşur; bağımsız bir kopya elde edersiniz. Farklı bir ortama şablon taşırken bu tercih edilir.

Silinen bir vDS'i geri getirirken bu kutuyu işaretlemeyi atlamak, yapılandırması doğru ama VM'lerin bağlanamadığı bir switch'le sonuçlanır.

### Restore Sonrası Ne Yapmak Gerekir?

Export'un neyi kaydetmediğini yukarıda belirtmiştik; kurtarma sonrası tabloyu tamamlamak için şunlar elle yapılır:

1. **Host'ları vDS'e ekleyin** ve fiziksel adaptörleri uplink slotlarına eşleyin (Add and Manage Hosts).
2. **VMkernel portlarını** ilgili distributed port group'lara taşıyın veya yeniden oluşturun; IP yapılandırmalarını girin.
3. **VM ağ bağlantılarını** doğrulayın; gerekirse toplu migrasyon aracıyla port group'lara geri bağlayın.
4. **Fonksiyonel doğrulama yapın:** `vmkping -I` ile servis ağlarını test edin, bir test VM'i taşıyarak vMotion'ı doğrulayın, storage erişimini kontrol edin.

Bu adımların varlığı, export'u değersiz kılmaz — aksine, en zaman alıcı ve hataya açık kısmı (onlarca port group'un VLAN ve politika tanımlarını elle yeniden girmek) ortadan kaldırır. Elle kalan iş, host'a özgü ve zaten doğrulanması gereken fiziksel eşlemelerdir.

### Bir Disipline Bağlamak: Ne Zaman Export Alınmalı?

Export'un değeri, ne sıklıkta ve ne zaman alındığına bağlıdır. Tek seferlik bir yedek, altı ay sonra gerçeği yansıtmaz.

**Her yapılandırma değişikliğinden önce ve sonra export alın:**

* Yeni port group eklemeden veya silmeden önce
* VLAN, güvenlik politikası veya teaming ayarı değiştirmeden önce
* vDS sürüm yükseltmesi öncesinde
* Yeni host'lar cluster'a dahil edilmeden önce

Bu, değişiklik yönetiminin standart bir parçası olmalıdır: **değişiklikten önceki export, geri dönüş noktanızdır.**

**Dosyaları doğru saklayın:**

* vCenter'ın kendisinden **bağımsız** bir konumda tutun. vCenter'ın çöktüğü bir senaryoda, yedek dosyasının vCenter üzerindeki bir datastore'da olması işinize yaramaz.
* Sürüm kontrolü mantığıyla, tarihli ve açıklamalı olarak arşivleyin.
* Kurumsal yedekleme politikanızın kapsamına dahil edin.

**Düzenli olarak test edin:** Test edilmemiş bir yedek, yedek sayılmaz. Lab ortamında dosyayı import ederek yapılandırmanın beklendiği gibi geldiğini periyodik olarak doğrulayın.

### Export'un Diğer Kullanım Alanları

Bu mekanizma yalnızca felaket kurtarma için değildir:

* **Ortamlar arası standardizasyon:** Production'da olgunlaşmış bir ağ mimarisini test veya DR ortamına birebir taşımak, elle yeniden kurmaktan hem hızlı hem de hatasızdır.
* **Değişiklik dokümantasyonu:** Export dosyaları, ağ yapılandırmasının belirli zamanlardaki halinin kaydıdır; denetim ve sorun teşhisi sırasında referans oluşturur.
* **Yeni datacenter kurulumu:** Yeni bir lokasyonda aynı mimariyi kurarken şablon olarak kullanılabilir.

Bir tamamlayıcı not: export/restore, vDS'in kendi yedekleme mekanizmasıdır ve **PowerCLI ile alınan yapılandırma çıktılarının yerine geçmez, onları tamamlar.** Script tabanlı bir dokümantasyon (port group listeleri, VLAN eşlemeleri, uplink atamaları) insan tarafından okunabilir bir kayıt sağlarken, export dosyası doğrudan geri yüklenebilir bir artefakttır. İkisini birlikte tutmak en sağlam yaklaşımdır.

### Sonuç

Distributed Switch yapılandırmasının export ve restore edilmesi, merkezi ağ yönetiminin doğal tamamlayıcısıdır. Özetle:

* Export, switch ve port group tanımlarını (VLAN'lar, politikalar, binding ayarları) kaydeder; **host üyeliklerini, vmnic eşlemelerini ve VM bağlantılarını kaydetmez.**
* Her zaman **"switch and all port groups"** kapsamıyla export alın ve açıklama alanını anlamlı doldurun.
* **Restore**, mevcut switch'i eski haline döndürür; **Import**, dosyadan yeni bir switch oluşturur. Silinen bir switch'i geri getirirken **orijinal tanımlayıcıları koruma** seçeneğini işaretlemeyi unutmayın.
* Restore sonrası host ekleme, uplink eşleme ve VMkernel yapılandırması elle tamamlanır; fonksiyonel doğrulama şarttır.
* Export'u tek seferlik bir işlem değil, **değişiklik yönetiminin parçası** haline getirin; dosyaları vCenter'dan bağımsız bir yerde saklayın ve periyodik olarak test edin.

Merkezi yönetimin gücü, aynı zamanda merkezi bir hatanın da tüm ortama yayılabilmesi anlamına gelir. Düzenli alınan ve test edilmiş bir yapılandırma yedeği, bu gücü risk almadan kullanmanın en basit yoludur — ve ağ mimarinizin kurumsal hafızasını kişilerden bağımsız bir artefakta bağlar.
