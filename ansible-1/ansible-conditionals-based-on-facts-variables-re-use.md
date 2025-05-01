---
icon: grid
---

# Ansible Conditionals based on facts, variables, re-use

### 1. OS Tabanlı Koşullar (Facts Kullanımı)

Diyelim ki filonuzda **Ubuntu 18.04**, **CentOS 7**, ve **Windows Server 2019** var. Bazı görevleri sadece belirli işletim sistemlerinde çalıştırmak isteyebilirsiniz. İşte burada **Ansible Facts** devreye girer.

1. **Ansible Facts** nedir?
   * Ansible, her hedef makineye bağlanıp sistem bilgilerini toplar (OS türü, sürüm, vs.). Bu bilgilere `ansible_facts[...]` üzerinden ulaşabilirsiniz.
   * Örnek: `ansible_facts['os_family']`, `ansible_facts['distribution_major_version']`.
2. **Örnek Senaryo**:
   * **Ubuntu 18.04** üzerinde belirli bir **Nginx** sürümünü kuracaksınız.
   *   Koşul ifadesi (when) şu şekilde olabilir:

       ```yaml
       - name: Install Nginx on Ubuntu 18.04
         apt:
           name: nginx=1.18.0
           state: present
         when: ansible_facts['os_family'] == 'Debian'
               and ansible_facts['distribution_major_version'] == '18'
       ```
   * Böylece **yalnızca** Ubuntu 18.04 olan makinelerde bu görev çalışır; CentOS veya Windows’ta atlanır (skipped).

#### Neden Faydalı?

* Aynı playbook içinde Debian ve RedHat gibi farklı OS’lere özel görevleri ayırabilir, tek bir dosya ile tüm makineleri yönetebilirsiniz.

### 2. Değişken (Variable) Tabanlı Koşullar

Sıklıkla, ortamlara (development, staging, production) göre farklı yapılandırmalar gerekebilir. Bunu bir **değişken** (örn. `app_env`) kullanarak kolayca yönetebilirsiniz.

1.  **Örnek**:

    ```yaml
    - name: Deploy config based on environment
      template:
        src: "{{ app_env }}_config.j2"
        dest: "/etc/myapp/config.conf"
    ```

    * `app_env` değişkenine `production` veya `staging` atarsanız, sırasıyla `production_config.j2` veya `staging_config.j2` dosyasını uygular.
2. **Koşul Eklemek**:
   *   Örneğin, “Production dışında konfig dosyasını yükleme” gibi bir ifadeniz varsa:

       ```yaml
       when: app_env == 'production'
       ```
   * Değişkeni envanterde (inventory), group\_vars, host\_vars veya playbook içinde `vars:` tanımlayabilirsiniz.

#### Neden Faydalı?

* Farklı ortamlara (dev, staging, prod) göre tek bir playbook kullanabilir, sadece değişkeni değiştirerek farklı yapılandırmaları otomatik uygulayabilirsiniz.

### 3. Görevleri Yeniden Kullanırken (Re-Use) Koşullar

Bazen **tüm sunucularda** çalışacak ortak görevleriniz olur (örn. paket kurma, dizin oluşturma), ancak **bazı eylemleri** sadece belirli bir ortama (production) veya belirli bir OS’ye yönelik yapmak istersiniz.

**Örnek**:

```yaml
- name: Common tasks for all servers
  hosts: all
  tasks:
    - name: Install required packages
      apt:
        name: [ 'package1', 'package2' ]
        state: present

    - name: Create necessary directories
      file:
        path: /path/to/directory
        state: directory

    - name: Start web application (only on production)
      service:
        name: myapp
        state: started
      when: environment == 'production'
```

1. **İlk iki görev** her sunucuda çalışır (paket kurma, dizin oluşturma).
2. **Son görev** sadece “environment” değişkeni “production” olan makinelerde çalışır.

