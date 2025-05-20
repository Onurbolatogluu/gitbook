---
icon: line-height
---

# Install required packages

### 1) Ansible Kontrol Makinesi Nedir?

* **Kontrol makinesi**, Ansible’ın kurulduğu ana makinedir. Bütün playbook’larınızı burada tutar ve yönetilen hedef sistemlere (managed nodes) bu makineden komut gönderirsiniz.
* Ansible **doğrudan Windows’a** kurulmaz; ancak bir **Linux VM** üzerinde çalıştırıp Windows sistemlerini **hedef** olarak yönetebilirsiniz.

### 2) Ansible Nasıl Kurulur?

Ansible’ı kurmak için iki temel yöntem vardır:

1. **Paket Yöneticileri** (Yum, DNF, APT, vb.)
2. **Python PIP** (daha esnek, genelde daha güncel sürümü kurmanızı sağlar)

#### a) Paket Yöneticisi ile Kurulum

*   **CentOS / Red Hat**:

    ```bash
    sudo yum install ansible
    ```
*   **Fedora**:

    ```bash
    sudo dnf install ansible
    ```
*   **Ubuntu / Debian**:

    ```bash
    sudo apt-get install ansible
    ```

Kurulum tamamlandığında, tipik olarak Ansible ile ilgili varsayılan dosyalar (`/etc/ansible/ansible.cfg` ve `/etc/ansible/hosts`) da oluşturulur.

#### b) PIP ile Kurulum

Eğer Python (ve PIP) kuruluysa, şu şekilde Ansible’ı kurabilirsiniz:

```bash
sudo pip install ansible
```

*   **Güncellemek** isterseniz:

    ```bash
    sudo pip install --upgrade ansible
    ```
*   **Belirli bir sürüm** kurmak isterseniz (örnek: 2.4):

    ```bash
    sudo pip install ansible==2.4
    ```

> **Not**: PIP ile kurduğunuzda **varsayılan envanter dosyası** (`/etc/ansible/hosts`) ve **varsayılan konfigürasyon dosyası** (`/etc/ansible/ansible.cfg`) oluşturulmaz. Bunları **manuel** oluşturmanız gerekir.

### 3) Statik Envanter (Hosts) Dosyası Oluşturma

Ansible’da yönetilecek hedef makinelerin IP veya hostname bilgileri **envanter (inventory) dosyasında** yer alır.

* Paket yöneticisi ile kurduysanız varsayılan olarak `/etc/ansible/hosts` dosyası gelir.
*   İsterseniz kendi projenizde, playbook’ların yanında bir **hosts** dosyası oluşturabilir ve onu kullanabilirsiniz.

    ```bash
    # /opt/my-playbook/hosts

    [webservers]
    web1 ansible_host=192.168.1.100
    web2 ansible_host=192.168.1.101
    ```
*   Playbook çalıştırırken bu özel envanter dosyasını şu şekilde belirtebilirsiniz:

    ```bash
    ansible-playbook site.yml -i /opt/my-playbook/hosts
    ```

### 4) Ansible Konfigürasyon Dosyası (ansible.cfg) Ayarlama

* Varsayılan konum: `/etc/ansible/ansible.cfg` (paket yöneticisi ile kurulduğunda oluşur).
* İçinde şunlar gibi ayarlar bulunur:
  * `inventory`: Varsayılan envanter dosyası konumu
  * `gathering`: Fact toplama davranışı (implicit/explicit)
  * `timeout`: SSH bağlantısı için zaman aşımı
  * `forks`: Paralel çalışacak host sayısı
  * ve daha fazlası…
*   Her projede farklı ayarlar kullanmak istiyorsanız, projenin dizinine **yerel** bir `ansible.cfg` oluşturabilir ve sadece değiştirmek istediğiniz değerleri yazabilirsiniz:

    ```ini
    # /opt/my-playbook/ansible.cfg
    [defaults]
    gathering = explicit
    timeout   = 20
    ```

    Ansible, playbook’u bu dizinden çalıştırdığınızda **önce** burada tanımlı ayarları uygular.

> **Önemli**: PIP ile kurduğunuzda `/etc/ansible/ansible.cfg` otomatik gelmez, bu dosyayı el ile oluşturmanız gerekir (veya bulunduğunuz proje dizinine `ansible.cfg` koyabilirsiniz).

### 5) Özet

1. **Kurulum**:
   * **Paket yöneticisi** (örn. `yum install ansible`) ile kolay kurulum.
   * **PIP** ile daha güncel veya belirli sürümleri kurmak mümkün. Varsayılan dosyalar otomatik gelmez.
2. **Envanter Dosyası**:
   * `/etc/ansible/hosts` veya istediğiniz herhangi bir konumda kendi `hosts` dosyanız.
   * Playbook’ta `-i` parametresiyle hangi envanterin kullanılacağını belirtebilirsiniz.
3. **Konfigürasyon Dosyası (`ansible.cfg`)**:
   * Paket yöneticisi kurulumunda `/etc/ansible/ansible.cfg` olarak gelir.
   * Proje bazında **yerel `ansible.cfg`** oluşturarak varsayılanları kolayca geçersiz kılabilirsiniz.

