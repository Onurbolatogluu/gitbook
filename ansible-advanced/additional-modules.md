---
icon: hammer-brush
---

# Additional Modules

### 📦 Paket Modülü (`package`)

Bilgisayarına bir program kurmak istediğinde ne yaparsın? Muhtemelen bir kurulum dosyası indirir ve çalıştırırsın. Ansible'da da sunucularına yazılım kurmak için **paket modülünü** kullanırız.

* Farklı Linux dağıtımları (CentOS, Ubuntu gibi) farklı paket yöneticileri kullanır. Mesela CentOS **YUM** kullanırken, Ubuntu **APT** kullanır.
* Neyse ki Ansible 2.0 ile birlikte gelen `package` adında sihirli bir modül var. Bu modül, sunucunun hangi işletim sistemini kullandığını otomatik olarak anlar ve doğru paket yöneticisini (YUM, APT vb.) kendisi seçer. Sana sadece hangi paketi kurmak istediğini söylemek kalır.
* **Dikkat Edilmesi Gereken Bir Nokta:** Paket isimleri işletim sistemlerine göre değişebilir. Örneğin, web sunucusu kurmak için CentOS'ta `httpd` paketini, Ubuntu'da ise `apache2` paketini kurman gerekir. `package` modülü sihirli olsa da, bu isim farkını otomatik olarak çözemez. Bu gibi durumlar için ileride öğreneceğin değişkenler veya koşullu ifadeler gibi daha gelişmiş teknikler kullanman gerekecek.

**Örnek:**

CentOS'a web sunucusu (httpd) kurmak:

```
- name: Install web server on CentOS
  yum:
    name: httpd
    state: installed
```

Ubuntu'ya web sunucusu (apache2) kurmak:

```
- name: Install web server on Ubuntu
  apt:
    name: apache2
    state: installed
```

Herhangi bir sisteme `package` modülü ile (paket adının `httpd` olduğunu varsayarak) web sunucusu kurmak:

```
- name: Install web server on any host using package module
  package:
    name: httpd
    state: installed
```

Burada `state: installed` parametresi, paketin kurulu olmasını istediğimizi belirtir. Eğer `state: absent` deseydik, paketi kaldırmak istediğimizi belirtmiş olurduk.

***

### ⚙️ Servis Modülü (`service`)

Programları kurduktan sonra bazen onları çalıştırmamız, durdurmamız veya yeniden başlatmamız gerekir. İşte **servis modülü** tam da bu işe yarar! Ayrıca, bir servisin bilgisayar her açıldığında otomatik olarak başlamasını da bu modülle sağlayabilirsin.

* **start:** Servisi başlatır.
* **stop:** Servisi durdurur.
* **restart:** Servisi yeniden başlatır.
* **enabled: yes:** Servisin sistem açılışında otomatik başlamasını sağlar.

**Örnek:** `httpd` (web sunucusu) servisini çalıştırmak ve açılışta otomatik başlamasını sağlamak:

```
- name: Ensure httpd service is running and enabled
  hosts: all
  tasks:
    - service:
        name: httpd
        state: started
        enabled: yes
```

`state: started` servisin çalışır durumda olmasını, `enabled: yes` ise sistem başlangıcında otomatik olarak çalışmasını sağlar.

***

### 🔥 Firewalld Modülü (`firewalld`)

Sunucularımızın güvenliği için güvenlik duvarı (firewall) kullanırız. **Firewalld modülü**, özellikle CentOS veya Red Hat tabanlı sistemlerde güvenlik duvarı kurallarını yönetmeni sağlar. Hangi portlardan veya servislerden trafik kabul edileceğini veya engelleneceğini bu modülle belirlersin.

* **permanent: yes**: Kuralın kalıcı olmasını sağlar (yeniden başlatma sonrası da geçerli olur).
* **immediate: yes**: Kuralın hemen uygulanmasını sağlar.
* **Dikkat!** Eğer `permanent: yes` yaparsan, kural hemen devreye girmez, sadece sistem yeniden başladığında aktif olur. Hem kalıcı olsun hem de hemen devreye girsin istiyorsan `immediate: yes` seçeneğini de eklemelisin. Eğer `permanent` seçeneği kullanılmazsa, `immediate` varsayılan olarak `yes` kabul edilir ve kural hemen uygulanır ama kalıcı olmaz.

**Örnek:** 8080 portundan HTTP trafiğine izin vermek:

```
- name: Add firewalld rule to allow HTTP traffic on port 8080
  hosts: all
  tasks:
    - firewalld:
        port: 8080/tcp   # 8080 portu ve TCP protokolü
        service: http    # HTTP servisi için
        source: 192.0.0.0/24 # Belirli bir IP bloğundan gelen trafik için
        zone: public     # 'public' zone için kural
        state: enabled   # Kural aktif
        permanent: yes   # Kural kalıcı
        immediate: yes   # Kural hemen uygulansın
```

***

### 💾 LVM Modülleri (`lvg`, `lvol`)

LVM (Logical Volume Management - Mantıksal Birim Yönetimi), disk alanını daha esnek kullanmanı sağlayan bir teknolojidir. Sanki legolarla oynar gibi disklerini birleştirip, istediğin boyutlarda yeni sanal diskler (mantıksal birimler) oluşturabilirsin.

* **`lvg` modülü:** "Volume Group" (Birim Grubu) oluşturmak için kullanılır. Bu, fiziksel disklerini veya disk bölümlerini bir araya getirdiğin bir havuz gibidir.
* **`lvol` modülü:** Oluşturduğun birim grubundan "Logical Volume" (Mantıksal Birim) yani sanal diskler oluşturmak için kullanılır.

**Örnek:** `vg1` adında bir birim grubu ve içinden `lvol1` adında 2GB'lık bir mantıksal birim oluşturmak:

```
- hosts: all
  tasks:
    - name: Create LVM Volume Group "vg1"
      lvg:
        vg: vg1                     # Birim grubunun adı
        pvs: /dev/sdb1,/dev/sdb2   # Kullanılacak fiziksel diskler/bölümler

    - name: Create Logical Volume "lvol1" with 2GB size
      lvol:
        vg: vg1                     # Hangi birim grubunu kullanacağı
        lv: lvol1                   # Mantıksal birimin adı
        size: 2g                    # Boyutu (2 Gigabyte)
```

***

### 📂 Dosya Sistemi (`filesystem`) ve Dosya (`file`) Modülleri

Disklerini veya mantıksal birimlerini kullanabilmek için önce onları biçimlendirmen (formatlaman) yani bir dosya sistemi oluşturman gerekir. Sonra da bu biçimlendirilmiş alanı bir klasöre bağlaman (mount etmen) gerekir. Dosya ve klasör oluşturmak, izinlerini ayarlamak gibi işlemler de sıkça yapılır.

* **`filesystem` modülü:** Bir disk bölümünde (örneğin `/dev/sdb1` veya oluşturduğun LVM biriminde) dosya sistemi (ext4, xfs gibi) oluşturur.
  * `opts` parametresi ile komut satırında kullanabileceğin ek seçenekleri (`mkfs.ext4 -L etiket` gibi) belirtebilirsin.
* **`mount` modülü:** Bir dosya sistemini belirli bir klasöre bağlar (mount eder).
* **`file` modülü:** Dosya ve klasörleri yönetir.
  * `state: directory` ile klasör oluşturursun.
  * `state: touch` ile boş bir dosya oluşturursun (dosya varsa sadece erişim zamanını günceller).
  * `state: link` ile sembolik link oluşturursun.
  * `owner`, `group`, `mode` parametreleriyle dosya/klasör sahibi, grubu ve erişim izinlerini ayarlayabilirsin.

**Örnek:** `/opt/app/web` klasörünü oluşturmak ve içine `index.html` adında boş bir dosya eklemek:

```
- hosts: all
  tasks:
    - name: Create application directory
      file:
        path: /opt/app/web        # Klasörün yolu
        state: directory          # Bunun bir klasör olduğunu belirtir

    - name: Create index.html file with proper ownership and mode
      file:
        path: /opt/app/web/index.html # Dosyanın yolu
        state: touch                  # Boş bir dosya oluştur (veya var olanın zamanını güncelle)
        owner: app-owner              # Sahibi
        group: app-owner              # Grubu
        mode: '0644'                  # Erişim izinleri (sahip okuma/yazma, grup okuma, diğerleri okuma)
```

***

### 🗜️ Arşiv (`archive`) ve Arşivden Çıkarma (`unarchive`) Modülleri

Bazen dosyaları veya klasörleri sıkıştırıp yedeklemek veya başka bir yere taşımak isteyebilirsin. Ya da sıkıştırılmış bir dosyayı açman gerekebilir.

* **`archive` modülü:** Dosyaları veya klasörleri sıkıştırır.
  * Varsayılan sıkıştırma formatı `.gz` dir.
  * `format` parametresi ile `zip`, `tar`, `bz2` gibi farklı formatlar belirtebilirsin.
* **`unarchive` modülü:** Sıkıştırılmış dosyaları açar.
  * **Çok Önemli Bir Detay:** `unarchive` modülü, varsayılan olarak sıkıştırılmış dosyanın Ansible'ı çalıştırdığın **kontrol sunucusunda** (senin bilgisayarın gibi) olduğunu varsayar ve onu hedef sunuculara kopyalayıp orada açar.
  * Eğer sıkıştırılmış dosya zaten hedef sunucunun kendisindeyse ve sen sadece onu açmak istiyorsan, `remote_src: yes` parametresini kullanmalısın.

**Örnek:** `/opt/app/web` klasörünü `/tmp/web.gz` olarak sıkıştırmak ve sonra bu sıkıştırılmış dosyayı (hedef sunucuda zaten var olduğunu varsayarak) `/opt/app/web` içine geri açmak:

```
- hosts: all
  tasks:
    - name: Compress the /opt/app/web folder
      archive:
        path: /opt/app/web     # Sıkıştırılacak klasör
        dest: /tmp/web.gz      # Sıkıştırılmış dosyanın kaydedileceği yer ve adı
        format: gz             # Sıkıştırma formatı

    - name: Uncompress the archive on the remote host
      unarchive:
        src: /tmp/web.gz       # Açılacak sıkıştırılmış dosyanın yolu
        dest: /opt/app/web     # Dosyaların açılacağı hedef klasör
        remote_src: yes        # Kaynak dosya hedef sunucuda
```

***

### ⏰ Cron Modülü (`cron`)

Belirli görevlerin düzenli aralıklarla (her gün, her saat, her ayın 15'inde vb.) otomatik olarak çalışmasını isteyebilirsin. İşte bu noktada **cron modülü** devreye girer. Linux sistemlerdeki `crontab` işlemini Ansible ile yönetmeni sağlar.

* `name`: Cron işinin adı (açıklayıcı bir isim).
* `job`: Çalıştırılacak komut veya script.
* `minute`, `hour`, `day`, `month`, `weekday`: İşin ne zaman çalışacağını belirten zamanlama parametreleri.
  * `*` (yıldız): "Herhangi bir" anlamına gelir. Örneğin, `minute: "*"` her dakika demek.
  * `*/2`: "Her 2'de bir" anlamına gelir. Örneğin, `minute: "*/2"` her iki dakikada bir demek (adım değeri).
  * `weekday`: Haftanın günü (0 Pazar, 1 Pazartesi, ..., 6 Cumartesi).

**Örnekler:**

1.  Her gün saat 08:10'da `/opt/scripts/health.sh` script'ini çalıştırmak:

    ```
    - name: Create a scheduled daily task at 08:10
      cron:
        name: Run daily health report
        job: sh /opt/scripts/health.sh
        month: "*"  # Her ay
        day: "*"    # Her gün
        hour: "8"   # Saat 8
        minute: "10" # Dakika 10
        weekday: "*" # Herhangi bir gün
    ```

    Bu, crontab'da `10 8 * * *` satırına denk gelir.
2.  Her iki dakikada bir `/opt/scripts/health.sh` script'ini çalıştırmak:

    ```
    - name: Create a scheduled task to run every two minutes
      cron:
        name: Run daily health report
        job: sh /opt/scripts/health.sh
        month: "*"
        day: "*"
        hour: "*"
        minute: "*/2" # Her iki dakikada bir
        weekday: "*"
    ```

    Bu, crontab'da `*/2 * * * *` satırına denk gelir.

***

### 👥 Kullanıcı (`user`), Grup (`group`) ve Yetkili Anahtar (`authorized_key`) Modülleri

Sunucularında kullanıcı hesapları oluşturmak, mevcut kullanıcıları yönetmek veya kullanıcı grupları tanımlamak isteyebilirsin. Ayrıca, şifresiz SSH bağlantısı için genel anahtarları (public keys) sunuculara eklemen gerekebilir.

* **`user` modülü:** Kullanıcı hesaplarını yönetir. Yeni kullanıcı oluşturabilir, kullanıcı ID'si (UID), grubu, varsayılan kabuğu (shell) gibi özelliklerini ayarlayabilirsin.
* **`group` modülü:** Kullanıcı gruplarını yönetir. Yeni grup oluşturabilirsin.
* **`authorized_key` modülü:** Belirli bir kullanıcının `~/.ssh/authorized_keys` dosyasına SSH genel anahtarlarını ekler. Bu, o kullanıcının o sunucuya şifresiz erişebilmesini sağlar.

**Örnek:** "maria" adında bir kullanıcı ve "developers" adında bir grup oluşturmak, sonra da "maria" kullanıcısı için bir SSH public anahtarı eklemek:

```
- hosts: all
  tasks:
    - name: Create the "developers" group
      group:
        name: developers
        state: present # Grubun var olmasını sağla

    - name: Create a new user "maria"
      user:
        name: maria
        uid: 1001                   # Kullanıcı ID'si
        group: developers           # Birincil grubu
        shell: /bin/bash            # Varsayılan komut satırı kabuğu
        state: present # Kullanıcının var olmasını sağla

    - name: Configure SSH keys for user "maria"
      authorized_key:
        user: maria
        state: present # Anahtarın var olmasını sağla
        key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABQC4WKn4K2G3iWg9HdCGo34gh+......root@97a1b9c3a" # Kullanıcının genel SSH anahtarı
```

`key` parametresine doğrudan anahtarı yazabileceğin gibi, `lookup('file', '/yol/dosya.pub')` gibi yöntemlerle bir dosyadan da okutabilirsin.&#x20;

