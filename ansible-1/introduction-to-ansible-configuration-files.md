---
icon: square-check
---

# Introduction to Ansible Configuration Files

### 1. Varsayılan Ansible Konfigürasyon Dosyası

* **Kurulum Sonrası Varsayılan Dosya**\
  Ansible kurulunca, genellikle `/etc/ansible/ansible.cfg` konumunda bir varsayılan konfigürasyon dosyası oluşturulur.
* **Bölümler (Sections)**\
  Bu dosya çeşitli bölümlere sahiptir (örn: `[defaults]`, `[inventory]`, `[privilege_escalation]`, `[ssh_connection]`, vb.).
  * En çok ayar genellikle `[defaults]` bölümünde yer alır.
  * Örnek ayarlar:
    * Varsayılan envanter dosyası (inventory) konumu
    * Log dosyası konumu
    * Modül veya rol dizinleri
    * SSH bağlantı zaman aşımı (timeout) süresi
    * Ansible’ın “facts” (sistem bilgileri) toplayıp toplamayacağı
    * Aynı anda kaç host’u paralel işleyebileceği (forks)

Bu varsayılan dosya, herhangi bir ek ayar belirtilmezse Ansible tarafından her zaman okunur.

### 2. Konfigürasyon Dosyası Öncelik Sırası

Ansible, **birden fazla konfigürasyon dosyası** bulunursa belli bir öncelik sırasına göre ayarları okur.  Yani, Ansible, ayarları (konfigürasyonları) **birden fazla yerden** okuyabilir. Ama **aynı ayar** farklı dosyalarda farklı değerlerle tanımlanırsa, **en yüksek öncelikli** olan kazanır. İşte öncelik sırası (en yüksek → en düşük):

1. **Environment Variable**
   * `ANSIBLE_CONFIG` adında bir değişken, konfigürasyon dosyasının tam yolunu gösterir.
   *   Örneğin,

       ```bash
       export ANSIBLE_CONFIG=/opt/ansible-web.cfg
       ```
   * Bu şekilde ayarlanırsa, **ilk** olarak buradaki ayarlar kullanılır.
2. **Mevcut Çalışma Dizini (Current Directory)**
   * Ansible komutunu çalıştırdığın dizinde bir `ansible.cfg` dosyası varsa, **ikinci** olarak buradaki ayarlar geçerli olur.
3. **Kullanıcının Ev Dizini (Home Directory) İçindeki `.ansible.cfg`**
   * Örneğin, `/home/onur/.ansible.cfg` dosyası.
   * Eğer çevre değişkeni veya çalışma dizininde bir dosya yoksa, **üçüncü** sırada bu dosyayı kontrol eder.
4. **Varsayılan Sistem Dosyası**
   * `/etc/ansible/ansible.cfg`
   * **Hiçbir üstteki konfigürasyon yoksa** veya ilgili ayar oralarda belirtilmemişse, en son bu dosyanın değerleri kullanılır.

**Nasıl Düşünmeliyiz?**

* Eğer tek bir ayarı herkesten önce zorla değiştirmek istiyorsan → **Environment Variable** ile `ANSIBLE_CONFIG`kullan.
* Bulunduğun dizinde özel ayarlar istiyorsan → **`ansible.cfg`** dosyasını oraya koy.
* Tüm kullanıcılar için geçerli varsayılan bir ayar yapmak istiyorsan → **`/etc/ansible/ansible.cfg`** dosyasını düzenle.

**Örnek Senaryo:**

* `/etc/ansible/ansible.cfg` içinde `timeout=10` yazıyor.
* Kendi klasöründeki `ansible.cfg` içinde `timeout=20` yazarsan, senin klasöründe Ansible çalıştırıldığında **20** geçerli olur.
* Ayrıca `export ANSIBLE_CONFIG=/home/onur/diger_ayarlar.cfg` yapıp o dosyada `timeout=30` yazarsan, bu kez **30** geçerli olur.

Bu şekilde, Ansible hangi dosyanın ayarını seçeceğini belirli bir öncelik sırasına göre karar verir.

### 3. Konfigürasyon Dosyalarını ve Değerlerini Değiştirme Yöntemleri

**Ansible Konfigürasyonunu Değiştirmek İçin Üç Ana Yol Var:**

1. **Klasöre (Dizine) `ansible.cfg` Dosyası Kopyalamak**
   * Eğer `/etc/ansible/ansible.cfg` dosyası var ve bu dosyadaki ayarları biraz değiştirmek istiyorsan:
     1. Projende, örneğin `web-playbooks` klasörüne geç.
     2. Orada `ansible.cfg` adında **yeni** bir dosya oluştur (veya `/etc/ansible/ansible.cfg`’den kopyala).
     3. İçindeki sadece değiştirmek istediğin satırları düzenle (örn. `timeout=20`).
   * Artık o klasörde `ansible-playbook ...` komutunu çalıştırdığında, **o klasördeki `ansible.cfg`** ayarları geçerli olur.
2. **Environment Variable (Değişken) Kullanmak**
   *   Eğer tek bir ayarı **hızlıca** değiştirmek istiyorsan, `ansible.cfg` dosyasıyla uğraşmadan terminalde şu şekilde yazabilirsin:

       ```bash
       ANSIBLE_GATHERING=explicit ansible-playbook myplaybook.yml
       ```

       Bu komut çalışırken `ANSIBLE_GATHERING` değeri **geçici** olarak `explicit` olur ve diğer tüm dosyalardaki `gathering` ayarlarını yok sayar.
   *   Ya da önce değişkeni kalıcı yapmak istersen,

       ```bash
       export ANSIBLE_GATHERING=explicit
       ansible-playbook myplaybook.yml
       ```

       şeklinde iki satırda yazabilirsin. Bu sayede, o terminal oturumu boyunca (exit yapana kadar) `gathering` ayarı `explicit` kalır.
3. **Sadece Birkaç Ayar Değiştirmek İstediğinde**
   * Bazen tüm `ansible.cfg` dosyasını kopyalayıp düzenlemek yerine, **yalnızca 1-2 küçük ayar** değiştirmen gerekebilir.
   *   Bu durumda, **environment variable** yöntemi çok pratiktir. Tek bir ayarı override (geçersiz kıl) etmek için:

       ```bash
       ANSIBLE_GATHERING=explicit ansible-playbook myplaybook.yml
       ```
   * Böylece sistemde başka yerde hangi ayar varsa varsın, bu tek komutluk değişken **her şeyin önüne geçer**.

#### Kısa Özet

* **Klasöre `ansible.cfg` kopyalamak**: Proje/klasör bazında kalıcı ayarlar yapmak istediğinde kullanılır.
* **Environment variable**: Hızlı veya tek seferlik ayar değişikliği için idealdir.

{% hint style="info" %}
_ANSIBLE\_GATHERING_ sadece bir örnek. Ansible’ın konfigürasyon dosyasında yer alan hemen hemen her parametre, aynı mantıkla bir çevresel değişken (environment variable) olarak geçersiz kılınabilir. Yani `ANSIBLE_GATHERING=explicit` yerine başka bir ayarı da `ANSIBLE_XXX=...` şeklinde değiştirebilirsin.\
\
Ansible’da bir konfigürasyon parametresini çevresel değişken aracılığıyla değiştirmek istiyorsan, genellikle şu kural geçerlidir:

1. **Önce** ilgili parametrenin adını **büyük harflere** çevir.
2. **Başına** `ANSIBLE_` önekini (prefix) ekle.
3. Değerini atayıp komut çalıştır (veya `export` komutu kullan).

Örneğin, Ansible’ın `ansible.cfg` dosyasında şu parametrelerin olduğunu düşünelim:

* `gathering`
* `forks`
* `host_key_checking`
* `timeout`

Bunları çevresel değişken olarak atayabilmek için sırasıyla:

* `ANSIBLE_GATHERING=...`
* `ANSIBLE_FORKS=...`
* `ANSIBLE_HOST_KEY_CHECKING=...`
* `ANSIBLE_TIMEOUT=...`

şeklinde kullanman gerekir.
{% endhint %}

