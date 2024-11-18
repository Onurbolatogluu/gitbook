---
icon: mailbox
---

# Workflow & Schedule Triggers

#### **Workflow Nedir?**

GitHub Workflow, yazılım geliştirme süreçlerinizi otomatikleştirmenize olanak tanıyan, GitHub Actions'ın temel yapı taşlarından biridir. Bir workflow, belirli olaylar gerçekleştiğinde (örneğin, kod push'ları, pull request'ler) otomatik olarak tetiklenen ve bir dizi işi (job) sıralı veya paralel olarak çalıştıran bir süreçtir. Bu işler, kodun derlenmesi, test edilmesi ve dağıtılması gibi adımları içerebilir.

Workflow'lar, projenizin kök dizininde bulunan `.github/workflows` klasöründe YAML dosyaları olarak tanımlanır. Bu dosyalarda, workflow'un ne zaman tetikleneceği, hangi işlerin yapılacağı ve bu işlerin hangi ortamlarda çalışacağı gibi bilgiler yer alır.\
\
Her bir workflow, bir veya daha fazla işten (job) oluşur. Bu işler, belirli bir sırayla veya paralel olarak çalıştırılabilir ve her biri, bir dizi adımdan (step) oluşur. Adımlar, komutların çalıştırılması, betiklerin yürütülmesi veya önceden tanımlanmış eylemlerin kullanılması gibi işlemleri içerir. Örneğin, kodunuzu `main` dalına her push ettiğinizde, testlerin otomatik olarak çalıştırılması ve uygulamanızın derlenmesi için bir workflow oluşturabilirsiniz.

GitHub Workflow'ları kullanarak, tekrarlayan görevleri otomatikleştirebilir, tutarlı süreçler uygulayabilir ve geliştirme hattınızın verimliliğini artırabilirsiniz.

***

#### **Workflow Triggers Türleri,**

1. **Depo İçindeki Olaylar:**
   * Push, pull request gibi GitHub deposunda gerçekleşen işlemler tetikleme yapar.
   * Örnek: Kod gönderildiğinde (push), testler otomatik olarak çalıştırılır.
2. **Depo Dışındaki Olaylar:**
   * GitHub dışında gerçekleşen olaylar, **repository\_dispatch** ile tetiklenebilir.
   * Örnek: Harici bir API çağrısı yapılarak workflow başlatılır.
3. **Zamanlanmış Olaylar (Scheduled Times):**
   * Belirli aralıklarla veya spesifik zamanlarda workflow çalıştırılabilir.
   * **cron formatı** kullanılarak zamanlama yapılır.
   * Örnek: Her gün saat 12:00'de bir işlem başlatma.
4. **Manuel Tetikleme (Manual):**
   * GitHub arayüzünden kullanıcı tarafından elle başlatılabilir.
   * Özellikle kullanıcı tarafından kontrol edilmesi gereken işlemler için faydalıdır.

#### **GitHub Actions Workflow Bileşenleri**

1. **Actions**
   * Yeniden kullanılabilir görevlerdir.
   * Spesifik işler için hazırlanmış kod parçacıklarıdır.
   * Örnek: Kod klonlama, container oluşturma.
2. **Workflows**
   * Depoda tanımlanmış otomatik süreçlerdir.
   * Birden fazla job içerir.
   * Tetikleyicilerle (event) veya zamanlanmış olarak çalışır.
3. **Jobs**
   * Aynı ortamda çalışan bir grup adımdır.
   * Varsayılan olarak paralel çalışır.
   * Birbirine bağımlı olarak çalışacak şekilde yapılandırılabilir.
4. **Steps**
   * Bir job içindeki bireysel görevlerdir.
   * Komutları veya action’ları çalıştırır.
   * Örnek: `echo "Hello, world!"` komutunun çalıştırılması.
5. **Runs**
   * Workflow’un her çalıştırılmasını temsil eder.
   * Tetikleyici olaylardan sonra başlatılır.
   * Tam yürütüm sırasında oluşan işlemleri kaydeder.
6. **Runners**
   * Job’ların çalıştırıldığı sunuculardır.
   * Türler:
     * **GitHub-hosted runners:** GitHub tarafından sağlanır.
     * **Self-hosted runners:** Kullanıcının kendi ortamında barındırılır.
7. **Marketplace**
   * Yeniden kullanılabilir action’ların bulunduğu platformdur.
   * Topluluk tarafından geliştirilmiş hazır araçları içerir.
   * Örnek: Docker image oluşturmak için bir action indirilebilir.



#### **Schedule Event Nedir?**

* Schedule Event, belirli bir zaman veya gün aralığında bir workflow’un otomatik olarak tetiklenmesini sağlar.
* Cron ifadeleri kullanılarak zamanlama yapılır.

```yaml
on:
  schedule:
    - cron: '30 5 * * 1,3'  # Pazartesi ve Çarşamba
    - cron: '30 5 * * 2,4'  # Salı ve Perşembe

jobs:
  test_schedule:
    runs-on: ubuntu-latest
    steps:
      - name: Not on Monday or Wednesday
        if: github.event.schedule != '30 5 * * 1,3'
        run: echo "Skip this step on Monday and Wednesday"

      - name: Every time
        run: echo "This step will always run"

```

* **Trigger:**
  * Bu workflow, her hafta:
    * Pazartesi ve Çarşamba 05:30'da.
    * Salı ve Perşembe 05:30'da tetiklenir.
* **Adımlar:**
  * İlk adım (`Not on Monday or Wednesday`):
    * Pazartesi ve Çarşamba günlerinde bu adımı atlar.
  * İkinci adım (`Every time`):
    * Her trigger sırasında çalışır.



Örnek bir senaryoda:

* Haftanın belirli günlerinde veritabanı yedekleme.
* Her gece saat 00:00’da kod analizi çalıştırma.



{% embed url="https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule" %}



