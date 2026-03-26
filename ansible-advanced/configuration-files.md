---
icon: sliders-up
---

# Configuration files

Ansible’ın **yapılandırma dosyaları (configuration files)**, Ansible’ın hangi ayarlarla çalışacağını belirleyen dosyalardır. Ansible’ı kurduğunuzda, genellikle **`/etc/ansible/ansible.cfg`** konumunda varsayılan bir yapılandırma dosyası oluşturulur. İçinde, Ansible’ın nasıl davranacağını etkileyen pek çok ayar ve bunlara ait değerler bulunur. Gelin bunları adım adım inceleyelim:

### 1) Varsayılan Yapılandırma Dosyası

* **Konum**: `/etc/ansible/ansible.cfg`
* **İçeriği**: `[defaults]`, `[inventory]`, `[privilege_escalation]`, `[ssh_connection]` gibi farklı bölümler (sections).
* **Örnek Ayarlar**:
  * `inventory`: Ansible hangi envanter (hosts listesi) dosyasını kullanacak?
  * `log_path`: Log dosyalarının nereye yazılacağı
  * `gathering`: Varsayılan fact toplama davranışı (implicit, explicit, vs.)
  * `timeout`: SSH bağlantısında zaman aşımı süresi (ör. 10 saniye)
  * `forks`: Aynı anda kaç host’la paralel çalışılacağı

Dosyanın içeriğini incelediğinizde, her bölümde farklı ayarların yer aldığını göreceksiniz.

### 2) Farklı Yerlerde Farklı `ansible.cfg` Dosyaları Kullanma

Bazen aynı kontrol makinesi (control node) üzerinde **farklı proje klasörleri** olabilir:

* Web sunucuları için bir playbook dizini
* Veritabanları için başka bir dizin
* Ağ ayarları (network) için başka bir dizin

Her birinde farklı yapılandırma ihtiyaçlarınız olabilir. Örneğin:

* Web playbook’larında fact toplama (gather\_facts) kapalı olsun.
* Veritabanı playbook’larında fact toplama açık olsun, ancak renkli çıktı (colored output) kapalı olsun.
* Ağ (network) playbook’larında SSH timeout süresi 20 saniye olsun.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Bu farklılıkları yönetmenin en kolay yollarından biri, **her dizine özel bir `ansible.cfg` dosyası koymaktır**. Bu şekilde, **Ansible bir playbook’u çalıştırırken** öncelikle _o_ dizindeki `ansible.cfg` dosyasına bakar. Bulamazsa bir üst öncelikli dosyaya (örneğin `/etc/ansible/ansible.cfg`) geri döner.

### 3) Yapılandırma Dosyaları İçin Öncelik Sırası

Eğer birden fazla yerde `ansible.cfg` veya ilgili ayarlar varsa, Ansible şu öncelik sırasına göre dosyalardaki ayarları uygular (en yüksek öncelik en başta):

1. **`ANSIBLE_CONFIG` ortam değişkeni** ile belirtilen dosya
   * Örnek: `ANSIBLE_CONFIG=/opt/ansible-web.cfg ansible-playbook site.yml`
   * Bu, tüm dosyaları **bastırır**.
2. **Ansible’ı çalıştırdığınız dizindeki** `ansible.cfg`
3. **Kullanıcının ev dizinindeki** `.ansible.cfg`
4. **Sistem varsayılanı**: `/etc/ansible/ansible.cfg`

Aynı ayar farklı dosyalarda tanımlanmışsa, en yüksek öncelikli dosya kazanır. **Diğerlerinde olmayan bir ayar ise sonraki dosyadan devralınır.**

### 4) Ortam Değişkenleri (Environment Variables) ile Ayar Yapma

Bazı durumlarda, yapılandırma dosyası kopyalamak istemeyebilirsiniz. Sadece hızlıca tek bir parametreyi değiştirmek yeterlidir. İşte bu nokta da **ortam değişkenleri** devreye girer.

*   **Örnek**: `gathering` adındaki ayarı `explicit` yapmak istiyorsanız, şu şekilde yapabilirsiniz:

    ```bash
    export ANSIBLE_GATHERING=explicit
    ansible-playbook playbook.yml
    ```

    Bu komuttan sonra Ansible, `gathering` parametresi için `explicit` değerini kullanır. Shell oturumunu kapatana kadar bu değişken aktif kalır.
*   **Tek Seferlik Kullanım**:

    ```bash
    ANSIBLE_GATHERING=explicit ansible-playbook playbook.yml
    ```

    Sadece bu komut için geçerli olur, komut bittiğinde ayar sıfırlanır.
* **Neden ANSIBLE\_GATHERING?**\
  Çoğu konfigürasyon parametresi, `ansible.cfg` içindeki ismin **büyük harf versiyonuna** ve **başına `ANSIBLE_` eklenmiş** haline dönüştürülerek ortam değişkeni haline getirilebilir. (Örneğin `timeout` → `ANSIBLE_TIMEOUT`, `forks` → `ANSIBLE_FORKS` gibi.)

Ortam değişkenleri **dosyalardaki** ayarlardan bile **daha yüksek önceliklidir**. Yani aynı parametreye hem dosyada hem de ortam değişkeninde farklı değer verilirseniz, ortam değişkeni kazanır.

### 5) `ansible-config` Komutları (List, View, Dump)

**Hangi ayarların mevcut olduğunu, varsayılan değerlerin ne olduğunu** ve şu anda **hangi konfigürasyon dosyasını Ansible’ın kullandığını** öğrenmek için kullanışlı araçlar:

1. **`ansible-config list`**
   * Tüm konfigürasyon seçeneklerini, varsayılan değerlerini ve hangi ortam değişkeniyle ilişkilendirilebileceğini gösterir.
2. **`ansible-config view`**
   * **Şu anda aktif** olan yapılandırma dosyasını gösterir. Eğer yerel dizinde bir `ansible.cfg` varsa onu, yoksa sıradaki dosyayı vb. gösterir.
3. **`ansible-config dump`**
   * Ansible’ın **geçerli oturumda** hangi ayarları kullandığını ve bu ayarları **nereden** çektiğini listeler. Örneğin `ansible-config dump | grep GATHERING` diyerek `gathering` ayarının nereden geldiğini görebilirsiniz.

### 6) Özet

1. **Varsayılan Dosya**: `/etc/ansible/ansible.cfg`’dir. Her şeyi orada tutabilir veya farklı projeler için yerel kopyalarını oluşturabilirsiniz.
2. **Yerel `ansible.cfg`**: Bulunduğunuz dizinde varsa, sistem varsayılanını bastırır. Bu, farklı projeler için özel ayarlara sahip olmanızı kolaylaştırır.
3. **Ortam Değişkeni**: Belirli bir parametreyi hızlıca değiştirmek istediğinizde kullanın. **En yüksek** önceliğe sahiptir.
4. **Öncelik Sırası**:
   1. `ANSIBLE_CONFIG` ortam değişkeni
   2. Yerel dizindeki `ansible.cfg`
   3. Kullanıcı ev dizinindeki `.ansible.cfg`
   4. `/etc/ansible/ansible.cfg`
5. **Konfigürasyon Komutları**: `ansible-config list`, `ansible-config view`, `ansible-config dump` → Hangisi geçerli, nereden geliyor, varsayılanı ne? Tüm detayları görmenizi sağlar.

