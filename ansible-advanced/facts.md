---
icon: sim-cards
---

# Facts

Ansible, yönetilen hedef makineler hakkında detaylı bilgileri (sistem özellikleri, ağ bilgileri, disk yapıları gibi) otomatik olarak toplayabilir. Bu bilgilere **“facts”** denir. Facts, her playbook çalıştığında **setup** modülü sayesinde varsayılan olarak toplanır ve `ansible_facts` isimli bir değişkende saklanır. Bu sayede ister debug amaçlı ister yapılandırma süreçlerinde (örneğin diske dair özel işlem yapmanız gerekiyorsa) bu bilgilere kolayca ulaşabilirsiniz.

### 1) Facts Nedir?

* **Tanım**: Facts, Ansible’ın yönetilen node’lardan (hedef makineler) otomatik olarak topladığı sistem bilgileri bütünüdür.
* **Neleri içerir?**
  * İşletim sistemi versiyonu (örneğin Ubuntu 20.04 mü, CentOS 7 mi?)
  * Mimari bilgisi (32 bit mi, 64 bit mi, x86 mı, ARM mi?)
  * İşlemci (CPU) detayları
  * Bellek (RAM) bilgileri
  * Disk ve dosya sistemi bilgileri (mount noktaları, kullanılabilir alan vb.)
  * Ağ (Network) arayüzleri, IP adresleri, MAC adresleri
  * Makinenin hostname, FQDN (Fully Qualified Domain Name) bilgileri
  * Saat/tarih gibi sistemsel bilgiler
  * DNS sunucuları, gateway bilgileri gibi ağ konfigürasyonları
  * Ve çok daha fazlası.

Bu bilgiler, otomasyon süreçlerinde farklı kararlara rehberlik eder. Örneğin, diskte yeterli yer yoksa farklı bir disk bölümü seçmek veya işletim sistemi bir Ubuntu türevi ise farklı paket yöneticisi kullanmak gibi.

### 2) Facts Nasıl Toplanır?

* Ansible her çalıştığında, **setup modülü** otomatik olarak devreye girer.
* “Gathering Facts” olarak göreceğiniz bu adım, henüz playbook’taki asıl görevler başlamadan önce varsayılan biçimde gerçekleşir.
* Toplanan veriler `ansible_facts` isimli bir sözlük (dictionary) yapısında saklanır.

#### Örnek Playbook ve Çıktı

```yaml
- name: Print hello message
  hosts: all
  tasks:
    - debug:
        msg: "Hello from Ansible!"
```

Bu playbook basitçe “Hello from Ansible!” mesajı bastıracaktır. Ancak çalıştırdığınızda, konsolda iki görev satırı görürsünüz:

1. `TASK [Gathering Facts]`
2. `TASK [debug]`

İlk görev, Ansible’ın otomatik olarak facts topladığı aşamadır. İkinci görev ise bizin tanımladığımız asıl debug görevidir.

### 3) Toplanan Facts’i Nasıl Görüntülerim?

`ansible_facts` değişkenini görebilmek için debug modülünü kullanabiliriz:

```yaml
- name: Print all gathered facts
  hosts: all
  tasks:
    - debug:
        var: ansible_facts
```

Bu kez playbook’u çalıştırdığınızda, Ansible hedef makinelerden topladığı **tüm** bilgileri (IP adresleri, hostname, diskler, bellek, işletim sistemi detayları vb.) ayrıntılı şekilde listeleyecektir.

#### Örnek Çıktıdan Bir Kesit

```json
"ansible_facts": {
    "architecture": "x86_64",
    "distribution": "Ubuntu",
    "distribution_version": "20.04",
    "fqdn": "web1",
    "interfaces": [
        "lo",
        "eth0"
    ],
    "memory_mb": {
        "real": {
            "free": 512,
            "total": 1024,
            "used": 512
        }
    },
    ...
}
```

### 4) Facts’i Neden Kullanırım?

1. **Karar Verme**: Farklı işletim sistemi veya donanım yapılarına göre koşullu görevler çalıştırabilirsiniz. Örneğin “Eğer `ansible_facts['distribution'] == 'CentOS'` ise yum kullan, yoksa apt kullan.”
2. **Konfigürasyon Yönetimi**: Disk bölümleri, ağ arayüzleri veya bellek gibi sistem bilgilerine göre farklı roller uygulayabilirsiniz.
3. **Debug ve İzleme**: Sistemleri hızlıca tanımak ve doğrulamak için toplanan bilgileri görüntüleyebilirsiniz.

### 5) Facts Toplamayı Devre Dışı Bırakma (gather\_facts: no)

Her zaman facts’e ihtiyaç duymayabilirsiniz. Özellikle sadece basit bir komut çalıştıracaksanız veya performans kaygınız varsa, facts toplamayı kapatabilirsiniz.

```yaml
- name: Print hello message without gathering facts
  hosts: all
  gather_facts: no
  tasks:
    - debug:
        msg: "Hello from Ansible!"
```

Bu durumda, playbook çıktısında yalnızca `debug` görevi çalışır. “Gathering Facts” adımı atlanır ve `ansible_facts` değişkeni boş olur.

**Önemli Not**:

* Eğer playbook içerisinde facts’e dayanan bir mantık varsa (örneğin bellek boyutuna göre bir işlem yapmak gibi), `gather_facts: no` ile devre dışı bırakırsanız bu veriler kullanılamaz.
* Varsayılan davranış Ansible yapılandırma dosyasındaki `gathering` ayarına (genelde `/etc/ansible/ansible.cfg` içinde) bağlıdır:
  * `implicit` → Otomatik olarak toplar.
  * `explicit` → Toplamaz, açıkça `gather_facts: yes` yazarsanız toplar.
  * `smart` → Aynı playbook içerisinde aynı host için facts zaten toplandıysa tekrar toplamaz (daha verimli).

Playbook içinde yazdığınız `gather_facts: no` veya `gather_facts: yes` ayarı, her zaman konfigürasyon dosyasındaki `gathering` değerine baskındır (önceliklidir).

### 6) Hangi Hostlardan Facts Toplanır?

Playbook hangi hostlara yönelik çalışıyorsa, facts sadece o hedef hostlardan toplanır.

* Örneğin envanterinizde `web1` ve `web2` diye iki makine olsun, ancak playbook sadece `web1` üzerinde çalışsın (`hosts: web1`). Bu durumda `ansible_facts` sadece `web1`’deki bilgileri içerecektir.
* Eğer `web2` makinesinden facts çekmek isterseniz, playbook içinde `web2`’yi de hedeflemeniz gerekir.

### Özet

* **Facts**: Ansible’ın hedef sistemlerden topladığı otomatik bilgiler.
* **Otomatik Toplama**: Varsayılan olarak `TASK [Gathering Facts]` şeklinde gerçekleşir.
* **Kullanım**: `ansible_facts` sözlüğünden değer çekerek koşullar yazmak, debug yapmak veya konfigürasyonları dinamikleştirmek mümkün.
* **Devre Dışı Bırakma**: `gather_facts: no` ile kapatabilirsiniz, ancak facts’e ihtiyaç duyan görevleriniz varsa dikkatli olun.
* **Performans İpuçları**: Büyük ortamda gerekmiyorsa `smart` veya `implicit` ayarlarını ihtiyacınıza göre özelleştirip playbook çalışma süresini hızlandırabilirsiniz.

