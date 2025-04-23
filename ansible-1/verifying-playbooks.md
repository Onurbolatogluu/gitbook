---
icon: money-check-pen
---

# Verifying Playbooks

### 1. Neden Playbook’ları Doğrulamak Gerekir?

* Ufak bir yazım hatası veya yanlış bir task, yüzlerce sunucuda kritik hizmetlerin durmasına sebep olabilir.
* “Ön izleme” yaparak riskleri düşürür, üretim ortamında sorun çıkmasını engellersiniz.
* Hataları erken yakalamak, sonradan düzeltmekten çok daha hızlı ve az maliyetlidir.

**Kısacası**, “check”/“diff”/“syntax check” modlarını kullanarak **ne olacağını** önceden görür, hataları giderebilir ve playbook’un prod ortamlarda _tam olarak beklediğiniz gibi_ çalışacağından emin olursunuz.

### 2. Check Mode (Dry Run)

#### 2.1 Ne Yapar?

* **Ansible’ın “dry-run” özelliğidir**: Playbook içindeki görevler **gerçekten** uygulanmaz; sadece _hangi değişikliklerin yapılacağını_ gösterir.
* Sunucularda **hiçbir değişiklik olmaz**. Yalnızca “Şu paket kurulu değilmiş, kuracaktım” gibi bir ön izleme verir.

#### 2.2 Nasıl Kullanılır?

```bash
ansible-playbook <playbook_ismi>.yml --check
```

**Örnek**: `install_nginx.yml` playbook’unuz varsa:

```bash
ansible-playbook install_nginx.yml --check
```

* Çıktıda “changed” gibi ibareler görürseniz, bunlar **gerçek bir değişiklik** değil, “değişiklik olsa ne olurdu?” bilgisidir.
* **Önemli**: Bazı modüller check mode’u desteklemeyebilir, bu durumda “SKIPPED” olarak geçer.

### 3. Diff Mode (Öncesi ve Sonrası Karşılaştırma)

#### 3.1 Ne İşe Yarar?

* **Sistemdeki dosya, ayar** vb. yapılarda **öncesi/sonrası farklarını** gösterir.
* Özellikle **yapılandırma dosyası** değiştiren (ör. `lineinfile`, `template`) görevlerde, “Hangi satır eklenecek veya silinecek?” gibi ayrıntılı bilgi verir.

#### 3.2 Nasıl Kullanılır?

```bash
ansible-playbook <playbook_ismi>.yml --diff
```

*   Tipik olarak `--check` ile birlikte kullanılır:

    ```bash
    ansible-playbook configure_nginx.yml --check --diff
    ```
* Çıktıda `--- before` ve `+++ after` gibi satırlar görürsünüz. `+` olan satırlar **eklenecek** bölümü, `-` olanlar **çıkarılacak** bölümü ifade eder.

**Örnek**: `configure_nginx.yml` ile `/etc/nginx/nginx.conf` dosyasına `client_max_body_size 100M;` ekleniyorsa, diff çıktısında:

```bash
--- before: /etc/nginx/nginx.conf
+++ after: /etc/nginx/nginx.conf
@@ -20,3 +20,4 @@
 # some existing lines
 #
+client_max_body_size 100M;
```

gibi bir ek satır görürsünüz.

### 4. Syntax Check Mode (YAML Yazım Kontrolü)

#### 4.1 Ne İşe Yarar?

* Playbook’unuzun **YAML sözdizimi**ni denetler.
* Basit hataları (“`:` unutulmuş”, “girinti bozuk” vb.) hızlıca yakalar.
* Görevleri çalıştırmaz, sadece “YAML dosyasında mantıksal hata var mı?” diye bakar.

#### 4.2 Nasıl Kullanılır?

```bash
ansible-playbook <playbook_ismi>.yml --syntax-check
```

* Hata yoksa “Syntax check complete. No errors found” benzeri bir mesaj alırsınız.
* Hata varsa, size “satır 5, kolon 9’da beklenen anahtar bulunamadı” gibi ipuçları verir.

### 5. Kullanım Senaryoları ve Önemli Notlar

1. **Check Mode**:
   * Prod ortamlara göndermeden önce “acaba gerçekten ne yapardı?” diye bakmak için ideal.
   * Bazı modüller (ör. `command`, `shell`) tam desteklemeyebilir, görevi “SKIPPED” olarak atlar.
2. **Diff Mode**:
   * Konfigürasyon dosyalarında satır satır ne değişecek görmek için mükemmel.
   * Özellikle hassas dosyaları güncellerken faydalı.
3. **Syntax Check**:
   * Basit ama çok faydalı. Özellikle “uygunsuz girinti” veya “:\` unutulması” gibi YAML hatalarını hızla yakalar.
   * Kod gözden geçirirken zaman kazandırır.

**Tavsiye**:

* **Prod ortamına** playbook uygulamadan önce sırasıyla “Syntax check → Check mode (ve diff mode)” yaparsanız riskleri büyük ölçüde azaltırsınız.
* Daha gelişmiş test yaklaşımları için **Molecule** gibi framework’ler veya `--limit` parametresiyle önce test sunucularda deneme yapma da kullanılabilir.

### 6. Özet

* **Check Mode (`--check`)**: Dry-run yapar, değişiklikleri **uygulamadan** raporlar.
* **Diff Mode (`--diff`)**: Dosya/konfig değişikliklerinde **öncesi/sonrası** farklılıkları satır satır gösterir.
* **Syntax Check (`--syntax-check`)**: YAML formatında yazdığınız **playbook dosyasının** yazım (sözdizimi) hatalarını kontrol eder.

***

```bash
$ ansible-playbook install_nginx.yml --check
PLAY [webservers] ****************************************************************
TASK [Gathering Facts] ***********************************************************
ok: [webserver1]


TASK [Ensure nginx is installed] **************************************************
changed: [webserver1]


PLAY RECAP ***********************************************************************
webserver1 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

{% hint style="info" %}
* **ok=2**
  * Bu, Ansible’ın **toplam 2 task** yürüttüğünü ve bunların her birinde “mantıksal olarak başarıyla sonuçlandığını” anlatır.
  * Task 1: `Gathering Facts` (her zaman ilk aşamada sistem bilgileri toplanır veya check mode’da simüle edilir)
  * Task 2: `Ensure nginx is installed` (asıl “package install” aşaması)
* **changed=1**
  * Ansible “check mode”da bile olsa, “bu görev normalde bir değişiklik yapacaktı” anlamını verir.
  * Örneğin “Ensure nginx is installed” adlı task, **nginx paketinin kurulu olmadığını** algılar ve “kurulacaktı” diye raporlar.
  * Gerçekte (dry-run olduğu için) paket **kurulmadı**, ama eğer check mode olmasaydı “changed=1” demek o görev gerçek bir değişiklik yapacaktı.
* **unreachable=0**
  * “0” demek, hiçbir host’a “bağlanılamadı” hatası alınmadı. Tüm hostlar (burada `webserver1`) başarıyla erişilebilir durumda.
* **failed=0**
  * “0” demek, hiçbir görev hata verip kesilmedi. Tüm görevler “success” şekilde (veya check mode’da “başarıyla test edildi”) tamamlandı.
* **skipped=0**
  * Bazı modüller check mode’u desteklemez veya `when:` koşulu nedeniyle atlanırsa “skipped” artar. Burada “0” demek, “hiç atlanmış görev yok”.
* **rescued=0**
  * Ansible’ın **blocks** yapısında bir hata olduğunda `rescue:` bölümü çalışabilir. Bu sayılar “rescue” mekanizmasının kaç kere tetiklendiğini gösterir. “0” demek, hiç devreye girmedi.
* **ignored=0**
  * `ignore_errors: yes` gibi ayarlar varsa ve bir görev hata alırsa “ignored” sayısı artar. Burada “0” demek, öyle bir durum da yaşanmamış.
{% endhint %}
