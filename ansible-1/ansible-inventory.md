---
icon: warehouse
---

# Ansible Inventory

**Inventory**, Ansible’ın hangi sunucularla/cihazlarla çalışacağını tanımladığımız listedir. Bu listede genellikle:

* Sunucu (veya cihaz) isimleri,
* IP adresleri veya tam alan adı (FQDN) bilgileri,
* Hangi port üzerinden bağlanılacağı (SSH için 22, WinRM için farklı port vb.),
* Bağlanılırken kullanılacak kullanıcı adı (user) ve bazen parola (ssh\_pass) gibi bilgiler yer alır.

Ansible, bu bilgiler sayesinde her bir hedef sunucuya “agent” yüklemeye gerek kalmadan (yani “agentless”) uzaktan bağlanır ve gereken işlemleri yapar.

#### Varsayılan Inventory Dosyası

* Ansible kurulduğunda, varsayılan olarak `/etc/ansible/hosts` dosyası **“default inventory”** kabul edilir.
* Eğer özel bir inventory oluşturmazsanız, Ansible komutları bu dosyadaki bilgileri okur.

Ancak ihtiyaca göre:

* Proje klasörünüzde farklı bir `hosts` veya `inventory` dosyası oluşturabilir,
* Ansible komutunu çalıştırırken `-i /path/to/myinventory` diyerek farklı bir inventory dosyası belirtebilirsiniz.

#### Basit Bir Inventory Örneği

INI-benzeri bir biçimde yazılır:

```ini
server1.company.com
server2.company.com

[mail]
server3.company.com
server4.company.com

[db]
server5.company.com
server6.company.com

[web]
server7.company.com
server8.company.com
```

* Köşeli parantez `[mail]`, `[db]`, `[web]` gibi “grup adları”dır.
* Gruplar, benzer roller üstlenen sunucuları bir arada toplar (ör. web sunucuları, veritabanı sunucuları).
* Bir sunucu tek bir grupta olabileceği gibi birden çok grupta da bulunabilir.

***

#### Ansible Inventory Parametreleri

INI formatında, her satıra **ek parametre** ekleyerek sunucuya özel ayarları yapabilirsiniz. Örneğin:

```ini
# Alias vererek
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root
db1  ansible_host=server2.company.com ansible_connection=winrm ansible_user=administrator
```

* `ansible_host`: Hedef sunucunun IP adresini veya FQDN’ini belirtir. (Örn: `192.168.1.10` veya `server1.company.com`)
* `ansible_connection`: Bağlantı türü (`ssh`, `winrm`, `localhost` vb.)
  * `ssh` → Linux sunucular
  * `winrm` → Windows sunucular
  * `localhost` → Yerel makine (Ansible’ın çalıştığı makine)
* `ansible_user`: Bağlanırken kullanılacak kullanıcı adı (Linux için genelde `root`, ya da Ubuntu sunucularda `ubuntu`vb.)
* `ansible_port`: SSH için varsayılan 22’dir. Eğer farklı bir port kullanıyorsanız (`ansible_port=2222` gibi) değiştirebilirsiniz.
* `ansible_ssh_pass`: Parola yazabileceğiniz parametre. Tavsiye edilmez, çünkü **metin olarak** saklamak güvenlik zafiyeti yaratır. Daha iyisi **SSH anahtarı** (key-based authentication) kullanmak veya Ansible Vault ile parolayı şifrelemektir.

Bunlara “inventory parametreleri” diyoruz; her makineye özel ayarları ya satır sonuna ekleriz ya da group\_vars/host\_vars yapılarını kullanarak daha düzenli saklayabiliriz (ileri konular).

#### Neden Inventory Kullanıyoruz?

1. **Kolay Yönetim**\
   Yüzlerce sunucunuz varsa, hepsini tek tek IP yazmak yerine gruplara ayırır ve playbook’larda sadece `[web]`, `[db]`gibi grup adları kullanarak işlemleri hedeflersiniz.
2. **Agentless Olma**\
   Ansible, hedef sunucularda ayrı bir program (agent) kurmaya ihtiyaç duymaz. Sadece SSH (Linux) veya WinRM (Windows) bağlantısı yeterlidir.
3. **Esneklik**
   * Aynı inventory içindeki bazı makineler Linux, bazıları Windows olabilir.
   * İster IP ister domain name kullanabilirsiniz.
4. **Parametrik Tanımlama**
   * Her sunucuya özel kullanıcı, port, parola gibi ayarları tek satırda belirtebilirsiniz.
   * Daha da ileride, `group_vars` ve `host_vars` adlı dizin yapılarıyla daha düzenli bir biçimde değişken tanımlayabilirsiniz.

#### Parola Konusu (ansible\_ssh\_pass)

* Demo veya test amaçlı hızlı denemeler için `ansible_ssh_pass` kullanabilirsiniz.
* **Güvenliğe önem verilen ortamlarda**, genellikle:
  * **SSH key-based authentication** tercih edilir (Linux tarafında).
  * Windows tarafında ise, Kerberos veya güvenli yöntemlerle etkileşim yapılır.
* Ya da **Ansible Vault** gibi bir mekanizma kullanarak parolaları şifreli bir dosyada saklayabilir, playbook çalışırken açabilirsiniz.

#### Kısa Özet

* **Inventory**, Ansible’ın hangi sunucularla çalışacağını tanımlar.
* Varsayılan dosya `/etc/ansible/hosts` olsa da, kendi özel inventory dosyanızı oluşturabilirsiniz.
* **INI Formatı**: Grup adlarını `[groupname]` şeklinde tanımlar, altına sunucu adreslerini yazarsınız.
* **Alias ve Parametreler** (örn. `ansible_host`, `ansible_connection`, `ansible_user`, vb.) sayesinde her sunucuya özel ayar girebilirsiniz.
* **Basit şifre** yerine **anahtarsız (passwordless) SSH** kullanmak veya Ansible Vault ile şifre saklamak güvenlik açısından önerilir.

***

#### Farklı Bir Inventory Dosyası Kullanmanın Yolları

1.  **Komut Satırında `-i` Parametresi ile Belirtmek**\
    En sık kullanılan yöntem budur. Playbook’u çalıştırırken şu şekilde yazarsınız:

    ```bash
    ansible-playbook -i /path/to/my_inventory.ini my_playbook.yaml
    ```

    Böylece Ansible, varsayılan `/etc/ansible/hosts` yerine sizin belirttiğiniz `my_inventory.ini` dosyasını kullanır.
2.  **`ansible.cfg` Dosyasında Belirtmek**\
    Projenizin kök dizininde (veya çalıştığınız herhangi bir dizinde) `ansible.cfg` dosyanız varsa, `[defaults]`bölümüne şu satırı ekleyebilirsiniz:

    ```ini
    [defaults]
    inventory = /path/to/my_inventory.ini
    ```

    Bu şekilde `ansible-playbook my_playbook.yaml` komutunu yazdığınızda, otomatik olarak bu inventory dosyası kullanılacaktır (başka bir `-i` parametresi verilmediği sürece).
3.  **Ortam Değişkeni (Environment Variable) Kullanmak**\
    Bir diğer alternatif, geçici veya kalıcı olarak `ANSIBLE_INVENTORY` değişkenini ayarlamaktır. Örneğin:

    ```bash
    export ANSIBLE_INVENTORY=/path/to/my_inventory.ini
    ansible-playbook my_playbook.yaml
    ```

    Bu yöntemle de varsayılan envanter dosyası yerine sizin dosyanız kullanılır.

{% hint style="info" %}
* “export” yöntemiyle ayarlanan değişken, **o terminal oturumunu kapatana** veya değişkeni `unset ANSIBLE_INVENTORY`ile silene kadar **kalıcı** hale gelir.
* Dolayısıyla ikinci, üçüncü komutlarınız da aynı inventory dosyasını kullanır.
{% endhint %}

{% hint style="info" %}
**1. Aynı Satırda Kullanma (Tek Komut için)**

```bash
ANSIBLE_INVENTORY=/path/to/my_inv.ini ansible-playbook my_playbook.yml
```

* Bu yöntemde **yalnızca** o komut çalışırken `ANSIBLE_INVENTORY` geçerli olur.
* Komut bittiğinde değişken sıfırlanır (shell’de kalıcı olmaz).
{% endhint %}

***

#### Kısa Özet

* **En Kolay Yol**: `ansible-playbook -i <inventory_dosyası> <playbook.yml>`
* **Kalıcı Yol**: `ansible.cfg` içinde `[defaults] inventory = <inventory_dosyası>`
* **Geçici Yol**: `ANSIBLE_INVENTORY` ortam değişkeni ayarlamak

Bu üç yöntemden hangisini seçtiğiniz, projeyi nasıl organize ettiğinize ve hangi ortamda çalıştığınıza göre değişir. Tek komutla sınamak isterseniz, en pratik olan `-i` parametresidir.
