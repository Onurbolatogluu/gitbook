# SSH Kullanırken Hız, Güvenlik ve Kararlılık Sağlayan Ayarlar

Eğer her gün SSH ile **deploy**, **debugging** veya **dosya senkronizasyonu** yapıyorsan, mutlaka yavaş ya da gecikmeli bağlantılarla karşılaşmışsındır. Güzel haber şu: birkaç basit SSH ayarıyla bağlantılarını **daha hızlı, kararlı ve güvenli** hale getirebilirsin. Bu ayarları `~/.ssh/config` veya `sshd_config` dosyana ekleyebilirsin.

### 1. 🔁 Connection Multiplexing (Bağlantı Paylaşımı) Aktif Et

Her SSH bağlantısında (scp, rsync dahil), SSH tekrar oturum açar, kimlik doğrular ve anahtar değişimi yapar. Bu işlemler **ekstra gecikmeye** neden olur. **Multiplexing** ile aynı TCP bağlantısını birden fazla SSH oturumu için tekrar kullanabilirsin.

`~/.ssh/config` dosyana ekle:

```bash
Host *
  ControlMaster auto
  ControlPath ~/.ssh/cm-%r@%h:%p
  ControlPersist 5m

```

🧩 Açıklama:

* `ControlMaster auto`: Aynı bağlantıyı birden fazla oturumla paylaşır.
* `ControlPath`: Socket dosyasının nerede tutulacağını belirtir.
* `ControlPersist 5m`: Son oturum kapandıktan sonra 5 dakika boyunca bağlantıyı açık tutar.

Bu ayar CI/CD işlemlerinde veya birden fazla SSH çağrısı yapan scriptlerde **anında hız artışı sağlar.**

### 2. ⚡ Daha Hızlı Şifreleme (Cipher) Algoritmaları Kullan

Bazı şifreleme yöntemleri (örneğin `aes256-cbc`) CPU’yu çok zorlar. Yeni nesil CPU’lar `chacha20-poly1305@openssh.com` gibi modern algoritmalar için **donanımsal hızlandırma** sağlar.

`~/.ssh/config` dosyana ekle:

```
Host *
  Ciphers chacha20-poly1305@openssh.com,aes128-gcm@openssh.com
```

Ya da bağlantı anında test et:

```bash
ssh -o Ciphers=chacha20-poly1305@openssh.com user@host
```

💡 Özellikle ARM tabanlı cihazlarda veya düşük güçteki sistemlerde **belirgin hız farkı** hissedilir.

### 3. 🔄 Bağlantının Kopmasını Engelle (KeepAlive)

VPN düşmesi veya ağ kesintisi yaşandığında SSH bağlantısı kopabilir. Bunu **KeepAlive** paketleriyle önleyebilirsin.

**İstemci tarafında (`~/.ssh/config`):**

```bash
Host *
  ServerAliveInterval 60
  ServerAliveCountMax 3
```

**Sunucu tarafında (`/etc/ssh/sshd_config`):**

```bash
ClientAliveInterval 60
ClientAliveCountMax 3
```

💡 Bu ayar, 60 saniyede bir “yaşıyor musun?” paketi gönderir. 3 kere cevap alamazsa oturumu düzgünce kapatır. Sonuç: **donmuş SSH oturumlarına elveda.**

### 4. 🚫 Reverse DNS Sorgularını Kapat (Sunucu Tarafı)

SSH, bağlantı geldiğinde IP’nin DNS’ini çözmeye çalışır. DNS yavaşsa veya hatalıysa bağlantı birkaç saniye geç açılır.

Sunucuda kapat:

```bash
# /etc/ssh/sshd_config
UseDNS no
```

SSH servisini yeniden yükle:

```bash
sudo systemctl reload sshd
```

⚡ Bağlantı süresi gözle görülür şekilde hızlanır.

### 5. 🧠 Bilinen Hostları Önceden Ekle

CI/CD pipeline’larında SSH bazen bilinmeyen host için onay ister. Bu da otomasyonun durmasına neden olabilir.

Güvendiğin sunucular için host anahtarlarını önceden ekle:

```bash
ssh-keyscan -H server.example.com >> ~/.ssh/known_hosts
```

Birden fazla sunucu için:

```bash
for host in server1 server2 server3; do
  ssh-keyscan -H $host >> ~/.ssh/known_hosts
done
```

⚠️ **Dikkat:** Bu işlemi sadece **güvendiğin sunucular** için yap. Yanlış kullanılırsa MITM (man-in-the-middle) saldırılarına açık hale gelebilirsin.

### 6. 🧩 GSSAPI Authentication'ı Kapat (Kullanılmıyorsa)

{% hint style="info" %}
**GSSAPI (Generic Security Services Application Program Interface)**, SSH’nin **kurumsal kimlik doğrulama** sistemleriyle (örneğin **Kerberos** veya **LDAP**) entegre olmasını sağlar.

Yani bu özellik, senin SSH ile bağlanırken:

* Parola veya anahtar yerine,
* **Kurumsal domain hesabın** (örneğin `onur@company.local`) üzerinden\
  otomatik olarak oturum açmanı sağlar.
{% endhint %}

Ubuntu ve RHEL gibi bazı dağıtımlar, varsayılan olarak **GSSAPIAuthentication**’ı aktif eder.\
Bu da her bağlantıda **1–3 saniyelik gecikme** yaratır.

Kullanmıyorsan kapat:

**İstemci tarafı:**

```bash
Host *
  GSSAPIAuthentication no
```

**Sunucu tarafı:**

```bash
# /etc/ssh/sshd_config
GSSAPIAuthentication no
```

💨 SSH bağlantıları birkaç saniye daha hızlı açılır.

### 7. 🔑 Daha Hızlı Anahtar Tipi Kullan (ed25519)

Yeni nesil `ed25519` anahtarları hem **daha güvenli** hem de **daha hızlı**.

Yeni anahtar oluştur:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Sonra `~/.ssh/config`’e ekle:

```bash
Host *
  IdentityFile ~/.ssh/id_ed25519
```

⚙️ Özellikle sık sık SSH bağlantısı kuran CI/CD süreçlerinde fark yaratır.

### 8. 🚚 scp Yerine rsync Kullan

Bu doğrudan bir konfig ayarı değil ama **dosya transferinde devrim** yaratır.

Örnek:

```bash
rsync -avz -e "ssh -T -o Compression=yes -x" ./app/ user@host:/opt/app/
```

Açıklama:

* `-a`: Dosya izinlerini korur
* `-v`: Ayrıntılı çıktı verir
* `-z`: Transfer sırasında sıkıştırır
* `-T`: Pseudo-TTY’yi kapatır
* `-x`: X11 forwarding’i devre dışı bırakır

📈 Büyük ve tekrar eden transferlerde `scp`’ye göre **10 kata kadar daha hızlı** olabilir.

### 🎯 Özet: SSH’yi Optimize Etmek İçin Temel Ayar

Aşağıdaki ayarları `~/.ssh/config` dosyana ekleyerek genel bir hız ve güvenlik artışı elde edebilirsin:

```bash
Host *
  ControlMaster auto
  ControlPath ~/.ssh/cm-%r@%h:%p
  ControlPersist 5m
  Compression yes
  ServerAliveInterval 60
  ServerAliveCountMax 3
  Ciphers chacha20-poly1305@openssh.com,aes128-gcm@openssh.com
  GSSAPIAuthentication no
  TCPKeepAlive yes
  IdentityFile ~/.ssh/id_ed25519
```

***

### 💬 Sonuç

SSH çoğu zaman farkında olmadan kullandığımız bir araç,\
ama birkaç küçük dokunuşla:

* Deploy sürelerini saniyeler kısaltabilir,
* Kopmaları önleyebilir,
* Dosya transferlerini hızlandırabilir,
* Otomasyonlarını daha güvenilir hale getirebilirsin.

