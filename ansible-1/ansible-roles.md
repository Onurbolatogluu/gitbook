---
icon: user-tie-hair
---

# Ansible Roles

### 1. Roles Nedir ve Neden Gerekli?

* Bir kişiye doktor rolü verince, tıp eğitimi almak, staj yapmak vb. adımları oluyor. Benzer şekilde, Ansible’da bir sunucuya “MySQL rolü” atayınca, o rol _otomatik_ şu işleri yapar: paketleri kurar, konfigürasyonu yapar, servisi başlatır vs.
* **Ansible’da Rol Vermek**: Sunucuya “web server” rolü, “database server” rolü gibi atama yaparak gereken tüm adımları (yükleme, konfigürasyon, servis) toplu hâlde tanımlayabilirsiniz.

**Neden Önemli?**

* Tekrardan kurtarır (DRY). Aynı MySQL kurulum adımlarını yüz defa yazmak yerine, bir kez “mysql-role” oluşturur, istediğiniz sunucularda kullanırsınız.
* “tasks/”, “vars/”, “handlers/”, “templates/” gibi dizinlerde kodunuzu parçalayıp daha okunaklı tutarsınız.
* Paylaşılabilir: Ansible Galaxy gibi platformlarda hazır rolleri bulabilir veya kendi oluşturduğunuzu paylaşabilirsiniz.

### 2. Rolün Bileşenleri

Bir rol genellikle şu klasörleri içerir:

* **tasks/**: Yapılacak adımlar (ör. paket kurma, servis başlatma vb.)
* **handlers/**: Tetiklendiğinde çalışan işlemler (örn. “mysql restarted” gibi)
* **vars/**: Rol içinde kullanılacak değişkenler
* **defaults/**: Varsayılan değişken değerleri
* **templates/**: Jinja2 şablon dosyaları
* **files/**: Kopyalanacak dosyalar vb.

Bu dizin yapısı sayesinde rol kodu net biçimde ayrılır ve bakımı kolaylaşır.

### 3. Rol Nasıl Oluşturulur?

1. **Elle klasör oluşturmak** yerine, `ansible-galaxy init <rol_adı>` komutunu kullanıp iskeleti otomatik oluşturun.
2. Ortaya çıkan klasöre girip (ör. `mysql/`), `tasks/main.yml` dosyasına Ansible görevlerinizi yazın, gerekirse `vars/main.yml`, `handlers/main.yml` vb. ekleyin.

**Örnek**:

```bash
$ ansible-galaxy init mysql
```

Bu, “mysql” adlı rolün klasör yapısını oluşturur.

{% hint style="info" %}
`ansible-galaxy init mysql` komutunu çalıştırdığınızda, varsayılan olarak bulunduğunuz **mevcut çalışma dizininde** (current directory) **`mysql/`** adlı bir klasör oluşturulur. Bu klasör içinde `tasks/`, `handlers/`, `vars/`, `defaults/` vb. standart rol dizinleri yer alır.
{% endhint %}

### 4. Rolü Playbook’ta Kullanmak

Playbook’unuzda:

```yaml
- name: Install and Configure MySQL
  hosts: db-server
  roles:
    - mysql
```

Böylece Ansible, `mysql` rolündeki görevleri sırasıyla uygular. Sizin playbook’ta tekrar paket kurma adımlarını yazmanıza gerek kalmaz.

**Rolü Bulma**:

* Ansible, rolü önce **playbook dizinindeki** `roles/` klasöründe arar.
* Ardından `/etc/ansible/roles` dizininde (varsayılan) veya `ansible.cfg` içindeki `roles_path` ayarlarına bakar.

### 5. Ansible Galaxy

* **Galaxy**: Ansible topluluğunun paylaştığı binlerce hazır rol bulunur (ör. “geerlingguy.mysql”).
*   Rolü yüklemek için:

    ```bash
    ansible-galaxy install geerlingguy.mysql
    ```

    Bu rol sistemin `roles_path` dizinine iner ve `playbook.yml` içinde `roles: - geerlingguy.mysql` diyerek kullanabilirsiniz.

### 6. Faydaları ve Özet

* **Tekrar Kullanım (Re-Use)**: Aynı rolü başka projelerde veya yeni sunucularda hızlıca kullanırsınız.
* **Organizasyon**: Kodunuz `tasks/`, `vars/`, `templates/` gibi klasörlerde düzenlenir, karmaşıklık azalır.
* **Paylaşım**: Rolü Galaxy’de yayınlayıp başkalarının da faydalanmasını sağlayabilir veya topluluktan gelen rolleri kullanabilirsiniz.

**Sonuç**: Ansible Roles, büyük veya küçük ölçekli her projede **düzenli, modüler ve paylaşılabilir** bir yapı kurmanıza yardımcı olur. Sunucuya “Nginx” rolü, “MySQL” rolü atayarak, tanımlı best practice dizin yapısı içinde temiz, tekrar edilebilir kod yazarsınız.

{% embed url="https://galaxy.ansible.com/ui/standalone/roles/" %}
