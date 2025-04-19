---
icon: hat-witch
---

# Magic Variables

Bu değişkenler sayesinde, bir host üzerinde tanımlanmış değişkeni başka bir host’tan okumak, belli bir grubun tüm üyelerini listelemek veya envanterdeki host’un ismine ulaşmak gibi “normalde” basit yollarla yapılamayan işlemleri gerçekleştirebilirsiniz.

### 1. Neden Magic Variables?

Daha önce öğrenmiş olabileceğiniz gibi, Ansible normalde her host’a ayrı bir “alt süreç” (subprocess) açar ve o host’a tanımlanan değişkenler **sadece** o host içinde geçerli olur. Örneğin, `web2`’de tanımlanan `dns_server=10.5.5.4` değeri, normal koşullarda `web1` veya `web3`’te tanımsız olur. Peki diğer host’lar bu değere ihtiyaç duyarsa ne yapacağız?

**Magic variables** burada devreye girer ve **başka host’ların** değişken ve fact bilgilerine erişme, bir grubun üyelerini listeleme gibi özel işlemleri mümkün kılar.

### 2. `hostvars` – Başka Bir Host’taki Değişkenleri Okumak

#### 2.1 Nasıl Çalışır?

`hostvars` adındaki sihirli değişken, tüm host’ların değişkenlerini (envanterde tanımlı veya gather\_facts ile elde edilen bilgileri) bir sözlük (dictionary) biçiminde tutar. Yani:

```django
hostvars["web2"]["dns_server"]
```

ya da

```django
hostvars["web2"].dns_server
```

ile `web2` host’una ait `dns_server` değişkenine erişirsiniz.

**Örnek Envanter**:

```ini
web1 ansible_host=192.168.56.101
web2 ansible_host=192.168.56.102 dns_server=10.5.5.4
web3 ansible_host=192.168.56.103
```

**Örnek Playbook**:

```yaml
- name: Print DNS server of web2 from all hosts
  hosts: all
  tasks:
    - debug:
        msg: "web2's DNS = {{ hostvars['web2'].dns_server }}"
```

* Bu playbook’u çalıştırdığınızda, `web1`, `web2`, `web3` üzerinde **her biri** “web2’nin dns\_server değeri 10.5.5.4” diye çıktı verir.
* Çünkü `hostvars` sihirli değişkeni, normalde “kendi host” dışında olan `web2` değişkenlerini de almamıza izin verir.

#### 2.2 Diğer Bilgiler (ansible\_facts)

`hostvars["web2"].ansible_facts` altında, `web2` host’unda gather\_facts ile toplanan CPU, disk, mount, network vb. bilgileri de bulabilirsiniz. Örnek:

```django
hostvars["web2"].ansible_facts.processor
hostvars["web2"].ansible_facts.architecture
```

***

### 3. `groups` – Bir Grubun Tüm Host’larını Listelemek

Envanterdeki `[web_servers]`, `[db_servers]` gibi grupların hangi host’lara sahip olduğunu `groups` sihirli değişkeni ile görebilirsiniz:

```yaml
- name: Show members of 'web_servers' group
  hosts: all
  tasks:
    - debug:
        msg: "{{ groups['web_servers'] }}"
```

Çıktıda mesela `["web1", "web2", "web3"]` gibi bir liste dönecektir.

### 4. `group_names` – Mevcut Host’un Üye Olduğu Gruplar

Hangi host üzerinde komut çalışıyorsa, o host’un üye olduğu grupları öğrenmek için `group_names` değişkeni kullanabilirsiniz:

```yaml
- name: Show which groups I'm in
  hosts: web1
  tasks:
    - debug:
        var: group_names
```

Eğer `web1`, `[web_servers]` ve `[americas]` gibi iki gruba üyeyse, `group_names` değeri `["web_servers", "americas"]` olur.

### 5. `inventory_hostname` – Envanterdeki Host İsmi

Bu sihirli değişken, **envanter dosyasında** o host için kullandığınız **takma ad** (alias) veya kısa ismi döndürür. Örneğin envanterde:

```ini
web1 ansible_host=192.168.56.101
```

varsa, `inventory_hostname` “web1” olur.\
Bazen bu, gerçek sunucu adı veya FQDN’den farklı olabilir (`ansible_host` ile IP veya FQDN verirsiniz). Örnek kullanım:

```yaml
- name: Show my inventory hostname
  hosts: web1
  tasks:
    - debug:
        msg: "My inventory name is {{ inventory_hostname }}"
```



* **`hostvars`**: Başka bir host’un envanter değişkenlerini ve facts bilgilerini okumanızı sağlar.
* **`groups`**: Bir grup adını (“web\_servers” vb.) verirsiniz, size o gruptaki host listesini döndürür.
* **`group_names`**: Mevcut host, hangi gruplara üye?
* **`inventory_hostname`**: Envanterdeki takma ad (alias). Gerçek IP veya FQDN’den farklı olabilir.
* Daha fazlası da var: `ansible_play_hosts`, `ansible_play_batch` vb.

Bu “sihirli değişkenler”, normalde “kendi host’u dışında tanımsız” olan bilgilere **paylaşım** yapmamızı ya da envanterin grup/host yapısını **dinamik** şekilde kullanmamızı sağlar. Böylece **karmaşık** senaryolarda “web2 sunucusunun IP’sini web1’de kullanmam gerek” gibi ihtiyaçlar karşılanır.

***

### 6. Diğer Magic Variables

* **`ansible_play_hosts`**: Mevcut play’de hala aktif olan host’ların listesi.
* **`ansible_play_batch`**: “Bölümler” halinde çalışıyorsanız (ör. serial veya batch), o anki batch’te bulunan host’ların listesi.
* **`ansible_playbook_python`**: Ansible komutunu çalıştırmak için kullanılan Python yolunu verir.

### Örnek Playbook

```yaml
---
- name: Demo Magic Variables
  hosts: all
  serial: 2  # Bu sayede 'batch' mantığını görebiliriz (2 host birden)
  gather_facts: no

  tasks:
    - name: Show ansible_play_hosts
      debug:
        var: ansible_play_hosts

    - name: Show ansible_play_batch
      debug:
        var: ansible_play_batch

    - name: Show ansible_playbook_python
      debug:
        var: ansible_playbook_python
```

#### Nasıl Çalışır?

1. **`hosts: all`**: Envanterdeki tüm host’ları hedefliyoruz.
2. **`serial: 2`**: Ansible, host’ları 2’şerli gruplar halinde çalıştıracak. Dolayısıyla “batch” dediğimiz kavram devreye girecek.

**Görevler:**

1. **`debug: var=ansible_play_hosts`**
   * Bu sihirli değişken, _mevcut play’deki tüm host’ların_ listesidir.
   * Örneğin 4 host’unuz varsa (`web1`, `web2`, `db1`, `db2`), `ansible_play_hosts` listesi `["web1", "web2", "db1", "db2"]` olabilir.
2. **`debug: var=ansible_play_batch`**
   * Bu, _o anki “batch”te_ bulunan host’ların listesi.
   * `serial: 2` kullandığımız için, ilk batch’te belki `["web1", "web2"]`, ikinci batch’te `["db1", "db2"]` şeklinde çalışabilir.
   * Her batch tamamlandıktan sonra diğer batch devreye girer ve o sırada `ansible_play_batch` değişir.
3. **`debug: var=ansible_playbook_python`**
   * Ansible komutunun kullandığı Python yorumlayıcısının tam yolunu verir.
   * Örneğin `/usr/bin/python3` gibi bir output alabilirsiniz.

**Örnek Çıktı (Temsili)**:

İlk batch (örneğin `web1`, `web2`):

```bash
TASK [Show ansible_play_hosts] ***********************************************
ok: [web1] => {
    "ansible_play_hosts": [
        "web1",
        "web2",
        "db1",
        "db2"
    ]
}
ok: [web2] => {
    "ansible_play_hosts": [
        "web1",
        "web2",
        "db1",
        "db2"
    ]
}

TASK [Show ansible_play_batch] ***********************************************
ok: [web1] => {
    "ansible_play_batch": [
        "web1",
        "web2"
    ]
}
ok: [web2] => {
    "ansible_play_batch": [
        "web1",
        "web2"
    ]
}

TASK [Show ansible_playbook_python] ******************************************
ok: [web1] => {
    "ansible_playbook_python": "/usr/bin/python3"
}
ok: [web2] => {
    "ansible_playbook_python": "/usr/bin/python3"
}
```

İkinci batch (örneğin `db1`, `db2`):

```bash
TASK [Show ansible_play_hosts] ***********************************************
ok: [db1] => {
    "ansible_play_hosts": [
        "web1",
        "web2",
        "db1",
        "db2"
    ]
}
ok: [db2] => {
    "ansible_play_hosts": [
        "web1",
        "web2",
        "db1",
        "db2"
    ]
}

TASK [Show ansible_play_batch] ***********************************************
ok: [db1] => {
    "ansible_play_batch": [
        "db1",
        "db2"
    ]
}
ok: [db2] => {
    "ansible_play_batch": [
        "db1",
        "db2"
    ]
}

TASK [Show ansible_playbook_python] ******************************************
ok: [db1] => {
    "ansible_playbook_python": "/usr/bin/python3"
}
ok: [db2] => {
    "ansible_playbook_python": "/usr/bin/python3"
}
```

***

### Ek Bilgiler

1. **`ansible_play_hosts` vs `ansible_play_batch`**
   * `ansible_play_hosts`: Bu play’de _genel olarak_ hedeflenen tüm host’ların listesidir. Batch ayarından bağımsız herkes yer alır.
   * `ansible_play_batch`: _Anlık_ olarak işlem gören host’lar (ör. `serial: 2` ile 2’şerli gruplar).
2. **`ansible_playbook_python`**
   * Ansible hangi Python yorumlayıcısı ile çalışıyorsa onun yolunu döndürür (sistem genelinde `/usr/bin/python3`, sanal ortamdaysa başka bir yol vb.).
3. **Farklı Ayarlar**
   * `serial` veya `batch_size` gibi ayarları yoksa, `ansible_play_batch` çoğu zaman `ansible_play_hosts` ile aynı olur (tüm hostlar tek batch).
4. **Kullanım Senaryoları**
   * `ansible_play_hosts` → Koşullar (`when`) veya döngü (`loop`) oluştururken “Play’deki tüm host’ların adına göre bir işlem yapma” gibi senaryolarda.
   * `ansible_play_batch` → Paralel işlemlerde, “anlık” partiyi hedefleyen advanced senaryolarda.
   * `ansible_playbook_python` → Belirli bir Python path’ini bilmek veya `command` modülünde “hangi Python ile” bazı script’leri çalıştırmak istediğinizi kontrol etmek için kullanılabilir.

Bu şekilde, **ansible\_play\_hosts**, **ansible\_play\_batch** ve **ansible\_playbook\_python** gibi sihirli değişkenlerin nasıl çalıştığına dair net bir örnek ve senaryo görmüş oldunuz.
