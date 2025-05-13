---
icon: flask-gear
---

# Playbook run options

Aşağıdaki bilgileri, Ansible playbook’larınızı çalıştırırken kullanabileceğiniz ek komut seçenekleri olarak düşünebilirsiniz. Her bir seçeneğin amacı ve kullanım şekli farklıdır. Bunları adım adım inceleyelim.&#x20;

### 1) Dry Run (Check Mode)

**Nedir?**\
Dry run veya check mode, playbook’unuzu “gerçekten” çalıştırmadan önce neler olacağını görmek için kullandığımız bir özelliktir. Bu moda alınca Ansible, hedef sunucularda herhangi bir değişiklik yapmaz; sadece hangi değişikliklerin yapılacağını ve sonuçların ne olacağını raporlar.

**Neden Kullanılır?**

* Üretim (production) ortamındaki kritik sunucularda playbook çalıştırmadan önce, yapılacak değişikliklerin neler olacağını görmek istediğinizde güvenlik sağlar.
* Beklenmedik sonuçlarla karşılaşmamak için, özellikle önemli güncellemelerde ön kontrol yapmanıza imkân tanır.

**Nasıl Kullanılır?**\
Playbook’u aşağıdaki gibi `--check` parametresiyle çalıştırırsınız:

```bash
ansible-playbook playbook.yml --check
```

Örnek bir `playbook.yml` içinde “httpd” paketini kuran bir görev olsun:

```yaml
---
- name: Install httpd
  hosts: all
  tasks:
    - name: Ensure httpd is installed
      yum:
        name: httpd
        state: installed
```

Dry run ile çalıştırdığınızda, Ansible size hangi değişiklikleri **yapacak** olduğunu ancak **henüz yapmadığını** gösterir.

### 2) Start at a Specific Task

**Nedir?**\
`--start-at-task` seçeneği, playbook’unuzu belirli bir görevden (task) başlatmanızı sağlar. Bu, playbook içerisinde birden fazla görev (task) olduğunda ve siz önceki görevleri atlayıp yalnızca belirli bir noktadan ilerlemek istediğinizde çok işe yarar.

**Neden Kullanılır?**

* Playbook’un başındaki görevler zaman alıyor veya sık test etmek istediğiniz kısım sonlara yakınsa, her seferinde baştan başlamanızı önler.
* Debug (hata ayıklama) yaparken, sadece belirli bir görevi tekrar tekrar test etmeniz gerekebilir.

**Nasıl Kullanılır?**\
Playbook dosyasında her görevin bir `name` alanı bulunur. O ismi baz alarak aşağıdaki gibi komut yazabilirsiniz:

```bash
ansible-playbook playbook.yml --start-at-task "Start httpd service"
```

Bu örnekte, `Start httpd service` adlı görevden itibaren playbook işlemleri başlayacaktır.

Örnek playbook:

```yaml
---
- name: Install httpd
  hosts: all
  tasks:
    - name: Install httpd
      yum:
        name: httpd
        state: installed

    - name: Start httpd service
      service:
        name: httpd
        state: started
```

Bu örnekte, `Install httpd` görevi atlanır ve doğrudan `Start httpd service` görevinden yürütmeye başlar.

### 3) Tags (Görevleri Etiketlemek ve Seçici Çalıştırma)

**Nedir?**\
Tags (etiketler), playbook içindeki belirli görevleri veya hatta tüm play’leri etiketlemenize yarar. Bu etiketler sayesinde playbook çalıştırırken, hangi görevlerin çalıştırılacağını veya hangi görevlerin atlanacağını seçebilirsiniz.

**Neden Kullanılır?**

* Büyük bir playbook’u parçalara ayırmak ve sadece belirli bölümleri çalıştırmak istediğinizde.
* “Yükleme (install) görevlerini çalıştır ama servis başlatma (start) görevlerini atla” gibi senaryolarda işinizi kolaylaştırır.
* Geniş bir otomasyon sürecinde, test veya bakım amaçlı sadece belirli kısımları hızlıca tekrar çalıştırmanız gerekebilir.

**Nasıl Kullanılır?**\
Öncelikle play veya task’lara etiketler eklenir:

```yaml
---
- name: Manage httpd
  hosts: all
  tags: [install, configure]
  tasks:
    - name: Install httpd
      yum:
        name: httpd
        state: installed
      tags: install

    - name: Start httpd service
      service:
        name: httpd
        state: started
      tags: start
```

#### Sadece Belirli Tag’leri Çalıştırmak

Sadece `install` etiketli görevleri çalıştırmak için:

```bash
ansible-playbook playbook.yml --tags "install"
```

Bu komut, yalnızca `install` etiketi bulunan görevleri işletecektir (Örneğin paket kurulumunu yapar, servisi başlatma görevini atlar).

#### Belirli Tag’leri Atlamak (Skip)

Eğer kurulum görevini atlamak ama diğer tüm görevleri çalıştırmak isterseniz:

```bash
ansible-playbook playbook.yml --skip-tags "install"
```

Böylece `install` etiketli görev atlanır, diğer bütün görevler çalışır.



### Özet ve İpuçları

1. **Check Mode (`--check`)**
   * Kullanmadan önce mantığınızı ve beklenen değişiklikleri görmek için harika bir yöntem.
   * Prod ortamlarında risk almadan önce mutlaka denemenizi öneririm.
2. **Start at Task (`--start-at-task`)**
   * Uzun veya çok adımlı playbook’larda zaman kazanmak için belirli bir adımdan başlama imkânı sağlar.
   * Hata ayıklama (debug) süreçlerinde çok kullanışlıdır.
3. **Tags**
   * İş akışınızı bölümlere ayırmanıza, gerektiğinde sadece istenilen bölümleri (görevleri) çalıştırmanıza olanak tanır.
   * Hem işletilecek görevleri seçmeye (`--tags`) hem de atlamaya (`--skip-tags`) yarar.

