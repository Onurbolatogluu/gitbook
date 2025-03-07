---
icon: hands-asl-interpreting
---

# Configure Self Hosted Runners for Enterprise

**GitHub Actions Self-Hosted Runners** özelliği, kendi altyapınız (sunucu, VM veya container) üzerinde GitHub Actions Runner çalıştırmanıza olanak sağlar. Özellikle **GitHub Enterprise** ortamlarında güvenlik, ağ konfigürasyonu, proxy, IP kısıtlamaları (allowlist) vb. ek gereksinimler olabildiği için ek konfigürasyonlar gerekebiliyor.

### 1. Self-Hosted Runner Nasıl Çalışır?

* **Runner**: GitHub Actions’ın iş akışlarını (workflow) çalıştıracak, GitHub’dan gelen komutları dinleyen ve  arka plan da çalışan işletim sistemidir. Normalde GitHub size _Hosted Runner_’lar sunar (Ubuntu, Windows, macOS), ancak “Self-Hosted” seçeneğiyle kendi makinenizde bu yazılımı kurabilir ve iş akışlarınızı orada koşturabilirsiniz.
* **Enterprise Avantajı**: Kendi veri merkezinize, bulutunuza veya şirket içi ağına (on-prem) Runner kurarak **daha fazla güvenlik, esneklik ve özelleştirme** elde edersiniz.

### 2. Proxy Sunucular (Proxy Servers)

Kurumsal ağlarda çoğu zaman **HTTP/HTTPS proxy** üzerinden internete çıkma zorunluluğu bulunur. GitHub Actions Runner’ın GitHub ile haberleşmesi için şu ortam değişkenleri (environment variables) kullanılabilir:

1. `http_proxy`
2. `https_proxy`
3. `no_proxy`

**Örnek Kullanım**

```bash
# Örnek Linux terminalinde
export http_proxy="http://proxy.example.com:8080"
export https_proxy="http://proxy.example.com:8080"
# Birden fazla adresi virgülle ayırabilirsiniz
export no_proxy="localhost,127.0.0.1,.example.org"
```

> **no\_proxy**: Proxy’yi devre dışı bırakmak istediğiniz alan adları veya IP’leri belirtirsiniz. Örneğin şirket içindeki `intranet.example.com` için proxy’ye gitmeden direkt erişim sağlanabilir.

Bu ortam değişkenlerini runner’ı başlatmadan önce tanımlarsanız, runner kurulum sırasında ve sonrasında doğru proxy üzerinden GitHub ile iletişime geçecektir.

***

### 3. IP Allowlist (IP İzin Listesi)

**IP allowlist**, GitHub Enterprise ortamında belirli IP adresleri veya IP bloklarının GitHub ile iletişimine izin verilmesi anlamına gelir. Şirket içi güvenlik politikaları gereği genellikle dış dünyaya giden istekler sıkı bir şekilde kısıtlanabilir.

* **Neden Gerekli?** Self-hosted runner’ın çalıştığı makine, GitHub Actions servislerine (ve bazen Container Registry, Packages vb. ek hizmetlere) erişmek zorunda. Eğer bir _Firewall_ veya _Reverse Proxy_ arkasındaysanız, bu sunucudan gelen istekleri engellememek için IP adresini/kaynağını güvenilir listede (allowlist) tanımlamanız gerekir.
* **Ne Yapmak Gerekir?**
  1. Self-hosted runnerınızın kullanacağı **kaynak IP adresini** (veya IP aralığını) bulun.
  2. GitHub Enterprise yönetim panelinde veya firewall ayarlarında, bu IP’yi “Allowlist”e ekleyin.
  3. GitHub Actions’ın kullandığı etki alanlarına (örn. `github.com`, `api.github.com`, `githubusercontent.com`vb.) ve portlara (genelde 443) izin verdiğinizden emin olun.

Bazı kurumsal yapılarda, GitHub Actions Runner’ın kendisi de _dışarıdan gelecek bağlantıları_ kabul etmek yerine genellikle dışarıya doğru bir webhook benzeri iletişime ihtiyaç duyar. Bu durumlarda, inbound yerine outbound ağ trafiğini ayarlamak önceliklidir.



### 4. Runner’ın Konfigürasyonu ve Çalıştırılması

#### A. Runner Kaynak Dosyaları

1. GitHub/Enterprise hesabı veya organizasyon seviyesinde “Actions > Runners” sayfasına gidin.
2. “New self-hosted runner” butonunu tıklayın, işletim sisteminize uygun komut setini kopyalayın.
3.  Sunucunuzda indirdiğiniz `actions-runner` klasörünün içindeki `config.sh` (Linux/macOS) veya `config.cmd`(Windows) komutlarını kullanarak runner’ı ayarlayın. Örnek:

    ```bash
    ./config.sh --url https://github.com/YourEnterprise/YourRepo \
                --token <TOKEN> \
                --name "MyEnterpriseRunner" \
                --labels "linux,x64,enterprise"
    ```
4. Ardından `./run.sh` (veya Windows’ta `run.cmd`) ile runner’ı arka planda çalıştırabilirsiniz.

#### B. Proxy Değişkenlerinin Tanımlanması

* Config/Run komutunu çalıştırmadan önce proxy ortam değişkenlerinin tanımlı olduğundan emin olun.
* Proxy ile ilgili ek parametreler varsa (örneğin kimlik doğrulama gerektiren proxy), ilgili kullanıcı:şifre bilgilerini environment variable’a ekleyebilir veya sistemde tanımlı bir secrets yönetimi kullanabilirsiniz.

#### C. Servis Olarak Çalıştırma (Opsiyonel)

* Linux/Windows’ta runner’ı sistem servisi olarak kurup sürekli çalışır halde tutabilirsiniz.
* Bunun için `sudo ./svc.sh install` ve `sudo ./svc.sh start` (Linux) komutları kullanılabilir.

### 5. Gelişmiş Konular

#### A. Ephemeral Runners

* **Ephemeral** modda runner çalıştırarak her workflow’dan sonra temiz bir ortamda runner’ın otomatik sonlandırılmasını sağlayabilirsiniz. Bu özellik, container tabanlı veya kısa ömürlü runner senaryolarında tercih edilir.

#### B. İleride Güncelleme

* GitHub Actions Runner düzenli olarak güncellenir. Bu nedenle self-hosted runner’ların da periyodik olarak güncellenmesi gerekir. Komut satırında `./config.sh remove` ve ardından en güncel sürümü indirip yeniden kurmak şeklinde bir akış izleyebilirsiniz (veya `./run.sh --once` gibi seçenekleri inceleyebilirsiniz).

#### C. Güvenlik Kapsamı

* Self-hosted runner, çalıştığı ağ içerisinde yer alan kaynaklara (veritabanı, dosya sistemi, vs.) erişebilir. Bu nedenle hangi repo’ların, hangi branşların bu runner’ı kullanmasına izin vereceğinize dikkat edin.
* Gerekli token’ları, parolaları veya sertifikaları GitHub Secrets vasıtasıyla ya da kurumsal gizli bilgileri tutan bir kasa (vault) üzerinden yönetmeye özen gösterin.

### 6. Özet

1. Şirket içi ağ koşullarında GitHub ile iletişim için `http_proxy`, `https_proxy` ve `no_proxy` tanımlamak gerekebilir.
2. Self-hosted runner’ın IP’sini GitHub Enterprise güvenlik ayarlarında veya şirket firewall/IPS/IDS ayarlarında allowlist’e eklemelisiniz.
3. **Kurulum Adımları**:
   * GitHub üzerinde “Self-Hosted Runner” oluşturma talimatlarını kopyalayın.
   * Sunucunuzda bu talimatlara uyarak runner’ı konfigüre edin.
   * Runner’ı (gerekirse servis olarak) başlatın ve GitHub Actions üzerinden test edin.
4. Ephemeral runner, otomatik güncellemeler, secret yönetimi gibi daha ileri düzey konulara da dikkat edin.

