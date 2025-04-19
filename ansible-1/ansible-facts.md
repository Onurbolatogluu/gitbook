---
icon: face-viewfinder
---

# Ansible Facts

Bu bilgilerle, bir playbook çalıştırıldığında Ansible’ın hedef makineler (host’lar) hakkında nasıl sistem bilgisi topladığını ve bu bilgileri nasıl kullanabileceğinizi öğreneceksiniz.

### 1. Ansible Facts Nedir?

**Ansible Facts**, Ansible’ın her playbook çalıştığında hedef makinelerden (host’lardan) otomatik olarak topladığı sistem bilgileridir. Örneğin:

* İşletim sistemi (distribution, versiyon, vb.)
* Sistem mimarisi (32-bit mi, 64-bit mi?)
* Ağ arayüzleri (ip adresleri, MAC adresleri, vb.)
* Bellek (memory), disk, CPU gibi donanım detayları
* Makinenin tam adı (FQDN), tarih/saat bilgisi, vs.

Bu bilgiler Ansible tarafından, **setup** isimli özel bir modül aracılığıyla toplanır. Her host için ayrı ayrı toplanan bu bilgiler, `ansible_facts` adlı değişkenin içinde saklanır.

### 2. Basit Bir Örnek: `ansible_facts` Değişkenini Yazdırmak

Aşağıdaki örnek playbook’ta sadece `debug` modülüyle `ansible_facts` adlı değişkeni ekrana bastırıyoruz:

```yaml
---
- name: Print Ansible Facts
  hosts: all
  tasks:
    - debug:
        var: ansible_facts
```

**Ne olur?**

1. Playbook başladığında, Ansible **her host** için otomatik olarak **facts** toplar (`TASK [Gathering Facts]`).
2. Sonra `debug` modülü, `ansible_facts` değişkenini ekrana basar.
3. Çıktıda “architecture”, “distribution”, “ansible\_facts.interfaces” gibi pek çok bilgi göreceksiniz.

Örnek Kısa Çıktı:

```bash
"ansible_facts": {
  "distribution": "Ubuntu",
  "distribution_version": "16.04",
  "all_ipv4_addresses": [
    "192.168.56.101"
  ],
  "machine": "x86_64",
  ...
}
```

Bu şekilde, host’un işletim sistemi, IP adresi, donanım bilgisi vb. her şeyi görebilirsiniz.

### 3. Fact Toplama (Gathering Facts) Nasıl ve Neden Otomatik Olur?

Standart Ansible davranışında, playbook’ta **hiçbahsetmeseniz** bile, her host’a ilk bağlandığında bir “setup” görevi çalıştırır. Bu görevin çıktısı `ansible_facts` olarak depolanır. Örneğin:

```yaml
- name: Print hello message
  hosts: all
  tasks:
    - debug:
        msg: "Hello from Ansible!"
```

Playbook’ta tek görev olsa bile, **playbook çıktısında** iki görev görürsünüz:

1. `TASK [Gathering Facts]`
2. `TASK [debug]`

Çünkü Ansible, `setup` modülünü otomatik çağırarak **facts** toplar.

### 4. Facts Kullanarak Dinamik Yapılandırma

Toplanan bu bilgiler, playbook’larınızı **daha akıllı** ve **dinamik** yapmaya yarar. Örneğin:

* **İşletim sistemine göre** paket kurma (“Ubuntu ise `apt`, CentOS ise `yum` kullan” şeklinde).
* **Disklerinizin boyutuna göre** bir mantık.
* **İnterface adlarına göre** ağ ayarları yapma.

Hepsinde `ansible_facts` altındaki verilerden yararlanabilirsiniz.

### 5. Facts’i Kapatmak (gather\_facts: no)

Eğer playbook’ta bu bilgilere **hiç ihtiyacınız yoksa** veya her seferinde toplanması gerekmiyorsa:

```yaml
---
- name: Print hello message without gathering facts
  hosts: all
  gather_facts: no
  tasks:
    - debug:
        var: ansible_facts
```

Bu durumda “`TASK [Gathering Facts]`” aşaması oluşmaz ve `ansible_facts` boş (`{}`) döner. Böylece zaman kazandırabilir, gereksiz network trafiğinden kaçınabilirsiniz.

### 6. Ansible.cfg’deki Ayarlar

`/etc/ansible/ansible.cfg` içinde `gathering` parametresi vardır:

* `implicit` (varsayılan): Otomatik fact toplar. İsterseniz `gather_facts: no` diyerek kapatabilirsiniz.
* `explicit`: Otomatik toplamaz, `gather_facts: yes` demeniz gerekir.
* `smart`: Bir kez toplar, sonra tekrar toplamaz gibi optimize davranış.

**Playbook’taki ayar** (`gather_facts: no` veya `yes`) her zaman konfigürasyondaki değerin üzerindedir (**daha yüksek öncelik**).

### 7. Neden Bazı Host’ların Facts Verisi Yok?

Ansible sadece **playbook’un hedef aldığı host’lar** için fact toplar. Örneğin envanterinizde `web1`, `web2` var ama playbook’u sadece `hosts: web1` derseniz, `web2` için hiç bir facts toplanmaz. Bu nedenle `web2` üzerinde `ansible_facts` “tanımsız” olabilir.

### 8. Özet

* **Facts**, Ansible’ın otomatik topladığı kapsamlı sistem bilgileri (`ansible_facts`).
* **Varsayılan**: Otomatik olarak toplanır (`gather_facts: yes`).
* **Kullanım**: `ansible_facts` içindeki alanlar (ör. `ansible_facts['distribution']`, `ansible_facts.network_interfaces` vb.) ile koşullu mantıklar, dinamik konfigürasyon yapabilirsiniz.
* **Kapatmak**: `gather_facts: no` ile kapatarak performans artırabilirsiniz (host bilgisine gerek yoksa).

Bu şekilde Ansible Facts ile sistemlerinizi **daha akıllı** yöneten playbook’lar hazırlayabilir, **farklı OS/donanım** senaryolarına **dinamik** şekilde uyum sağlayabilirsiniz.

