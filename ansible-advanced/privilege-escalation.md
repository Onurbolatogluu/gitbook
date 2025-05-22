---
icon: user-tie-hair
---

# Privilege Escalation

### 1) Privilege Escalation Nedir?

Bir sistemde farklı kullanıcıların farklı izin seviyeleri (privileges) vardır. Normal kullanıcıların her işlemi yapmaya yetkisi yoktur; bazıları **root** veya yönetici izni (sudo vb.) gerektirir.

* **Yetki Yükseltme (Privilege Escalation)**: Geçici olarak daha yüksek bir izin seviyesine (genellikle root gibi) geçmektir.
* Günlük hayattan örnek: “Admin” isimli bir kullanıcıysanız, paket kurmak ya da sistem dosyalarını değiştirmek istediğinizde `sudo` komutu kullanarak “root hakları” elde edersiniz.

Ansible da aynı mantığı uygular. Normal bir kullanıcıyla sisteme bağlandığınızda, root izni gerektiren görevlerde (ör. paket kurulumu, servis ayarları) “become” (eski adıyla “sudo”) özelliğini kullanarak geçici olarak yüksek yetki kullanır.

### 2) Neden Gerekli?

* Pek çok yönetimsel görev (örn. paket yükleme, servis başlatma, sistem dosyası düzenleme) normal kullanıcı izinleriyle yapılamaz.
* Ansible, _root_ olarak bağlanmak yerine genellikle daha güvenli bir yöntemi tercih eder: Normal kullanıcıyla SSH’ye bağlanıp sadece gerektiği anda yetki yükseltir.

### 3) Ansible’da Become Direktifi (Sudo)

**Playbook içinde** ya da **envanter (hosts) dosyasında** belirtebilirsiniz. En basit haliyle, playbook içinden:

```yaml
- name: Install Nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Install package
      yum:
        name: nginx
        state: present
```

* `become: yes` → Görevleri root haklarıyla çalıştır (sudo).
* Varsayılan yükseltme yöntemi çoğunlukla `sudo`dur.

Görevlerinizde root iznine ihtiyaç olduğunda, Ansible “sudo” kullanarak yüksek hakları devreye sokar.

### 4) Diğer Become Parametreleri

1.  **`become_method`**: Hangi yöntemi kullanacağınızı belirtir. Varsayılan `sudo`. Diğer yöntemler: `doas`, `pfexec`, `runas` (Windows), vb.

    ```yaml
    become_method: sudo
    ```
2.  **`become_user`**: Root yerine **farklı bir kullanıcı** (ör. `nginx`, `mysql`) olup o kullanıcının haklarıyla işlem yapmak.

    ```yaml
    become_user: nginx
    ```

Bu parametreleri _playbook_, _inventory_, _konfigürasyon dosyası (ansible.cfg)_ veya _komut satırı_ düzeyinde tanımlayabilirsiniz.

### 5) Envanter (Hosts) Dosyasından Örnek

İsterseniz **envanter** dosyanızda her host için hangi yetki yükseltme ayarlarını kullanacağınızı belirtebilirsiniz. Mesela:

```
bashKopyala[webservers]
web1 ansible_host=192.168.1.101 ansible_user=admin ansible_become=yes ansible_become_method=sudo
```

Bu durumda web1’e `admin` kullanıcı olarak SSH yapacak, görevleri `sudo` ile yükselterek çalıştıracak.

### 6) Parola Gerekiyorsa?

Bazı sunucularda `sudo` yaparken bir **sudo parolası** (sudo password) gerekir. Bunu iki şekilde halledebilirsiniz:

1.  **Envanterde Parola Tanımlamak** (riskli, düz metin):

    ```ini
    web1 ansible_host=192.168.1.101 ansible_user=admin ansible_become_pass=SuperSecret123
    ```

    > Daha güvenli kullanım için **Ansible Vault** önerilir.
2.  **Komut Satırında Parola Sorma**:

    ```bash
    ansible-playbook site.yml --ask-become-pass
    ```

    Playbook çalışırken sizden interaktif olarak sudo parolasını ister.

### 7) Switch User (become\_user) Kullanımı

* Sistemde bir uygulama kullanıcı hesabı varsa (örn. `nginx`, `mysql`), o kullanıcı hesabına geçerek dosyaları düzenlemeniz gerekebilir.
*   Örneğin, bir görevi “nginx” kullanıcısı haklarıyla yapmak için:

    ```yaml
    - name: Configure Nginx
      hosts: webservers
      become: yes
      become_user: nginx
      tasks:
        - name: Create config file in Nginx home
          template:
            src: nginx.conf.j2
            dest: /home/nginx/nginx.conf
    ```

Bu şekilde Ansible, önce normal kullanıcıyla SSH yapar, sonra `sudo su - nginx` benzeri bir komutla o hesaba geçip görevi gerçekleştirir.

### 8) Ayarların Öncelik Sırası

Ansible’da aynı ayar (ör. `become`, `become_user`) birden fazla yerde tanımlanabilir:

1. **Komut satırında** (en yüksek öncelik)
2. **Playbook içinde**
3. **Inventory (hosts) dosyasında**
4. **ansible.cfg** (en düşük öncelik)

Hangisinde tanımlarsanız, **daha üst öncelikli** bir yerde tanımlanmadığı sürece o ayar geçerli olur.

### 9) Özet

* **Yetki yükseltme** (Privilege Escalation), Ansible’daki “become” mekanizmasıyla **root** veya başka bir kullanıcıya geçer.
* **Sudo** varsayılan yöntemdir (`become_method: sudo`), ancak `doas`, `pfexec` vb. kullanılabilir.
* **Parola gerektiren sunucularda** `ansible_become_pass` veya `--ask-become-pass` kullanabilirsiniz.
* **Switch user** senaryosu için `become_user` parametresiyle, başka bir sistem hesabına geçerek görevleri yürütmek mümkündür.

