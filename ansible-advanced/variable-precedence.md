---
icon: square-root-variable
---

# Variable Precedence

**Ansible Variable Precedence Nedir ve Nasıl Çalışır?**

Ansible’da değişken (variable), her bir host üzerinde farklılık gösterebilen parametreleri saklamak için kullanılır. Örneğin, birden fazla sunucuyu yönetirken her birine farklı IP adresi veya DNS sunucusu tanımlamak istediğinizde bu değerleri değişkenler aracılığıyla belirtirsiniz.

Fakat bazen aynı değişken birden fazla yerde tanımlanmış olabilir. Örneğin:

* Inventory içindeki group (grup) değişkenlerinde
* Inventory içindeki host (makine) değişkenlerinde
* Playbook içinde (vars: satırlarında)
* Komut satırından gönderilen extra\_vars parametresi ile

Bu durumda “herhangi bir değişken birden çok yerde tanımlıyken Ansible hangisini kullanır?” sorusu ortaya çıkar. İşte **variable precedence (değişken önceliği)** tam da bu sorunun cevabını düzenler.

### 1. Inventory Dosyaları ve Değişkenler

En temel kullanım şeklinde değişkenleri Ansible’ın `inventory` dosyasında tanımlayabilirsiniz. Örneğin:

```ini
# /etc/ansible/hosts
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101
web3 ansible_host=172.20.1.102

[web_servers]
web1
web2
web3

[web_servers:vars]
dns_server=10.5.5.3
```

Yukarıdaki inventory’de şu noktalar önemlidir:

* `web1`, `web2`, `web3` isimli üç host var (örnek olarak 3 sunucu).
* `[web_servers]` isimli bir grup yaratılıyor ve bu gruba `web1`, `web2`, `web3` dahil ediliyor.
* `[web_servers:vars]` altında `dns_server` değişkeni tanımlanıyor ve değeri `10.5.5.3` olarak set ediliyor. Bu, `web_servers` grubundaki _tüm_ host’lara uygulanacak bir değişken anlamına gelir.

Ansible playbook çalıştığında, ilk olarak **group** değişkenleri host’lara atanır. Bu nedenle `web1`, `web2` ve `web3`’ün hepsi `dns_server=10.5.5.3` değeriyle başlar.

### 2. Host Değişkenleri (Host Vars)

Aynı değişkeni (örneğin `dns_server`) ayrıca tek bir host üzerinde de tanımlayabilirsiniz. Başka bir deyişle, `[web_servers:vars]` altındaki `dns_server` değeri yerine, host özelinde farklı bir `dns_server` değeri yazabilirsiniz. Örneğin:

```ini
# /etc/ansible/hosts
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101  dns_server=10.5.5.4
web3 ansible_host=172.20.1.102

[web_servers]
web1
web2
web3

[web_servers:vars]
dns_server=10.5.5.3
```

Bu örnekte:

* Grup düzeyinde `dns_server=10.5.5.3` tanımlanmış.
* `web2` host’u için ayrıca `dns_server=10.5.5.4` yazılmış.

Ansible’da, **host değişkenleri grup değişkenlerinden daha yüksek önceliğe** sahiptir. Yani `web2` host’u için playbook çalışırken `dns_server` değeri `10.5.5.4` kullanılır; diğerleri için (örneğin, `web1` ve `web3`) `10.5.5.3` kullanılır.

### 3. Playbook İçinde Tanımlı Değişkenler (Play Vars)

Bir değişkeni doğrudan bir playbook içinde de tanımlamanız mümkündür. Örneğin aşağıdaki gibi bir playbook düşünün:

```yaml
---
- name: Configure DNS Server
  hosts: all
  vars:
    dns_server: 10.5.5.5
  tasks:
    - name: Update DNS settings
      nsupdate:
        server: '{{ dns_server }}'
```

Burada `vars:` altında `dns_server` değişkeni `10.5.5.5` olarak tanımlandı.\
Bu durumda artık:

* Grup veya host’taki `dns_server` değişkenleri **geçersiz** kalır.
* Playbook içindeki tanım (**play-level** variables), inventory’deki tanımları **override** eder (yani onların üstüne yazar).

**Öncelik sırası**:

1. (Daha düşük) Grup değişkeni → 2. Host değişkeni → 3. (Daha yüksek) Playbook içindeki `vars` tanımı

### 4. Komut Satırından Gönderilen Extra Vars (En Yüksek Öncelik)

Playbook çalıştırırken şu şekilde bir komut kullanabilirsiniz:

```bash
ansible-playbook playbook.yml --extra-vars "dns_server=10.5.5.6"
```

Bu durumda `extra-vars` parametresiyle gönderilen `dns_server=10.5.5.6` değeri, _en yüksek düzeydeki önceliğe_ sahip olur. Yani playbook içindekileri de, inventory içindekileri de geçersiz kılar.\
Dolayısıyla `dns_server`, çalışma anında `10.5.5.6` şeklinde ayarlanmış olur.

### 5. Özet: Değişken Önceliği Sıralaması

Ansible belgelerinde, değişkenlerin öncelik sıralamasını (yukarıdan aşağıya doğru) şu şekilde görebilirsiniz:

1. **Role Defaults** (en düşük öncelik)
2. **Inventory Group Vars**
3. **Inventory Host Vars**
4. **Playbook Vars**
5. **Extra Vars** (en yüksek öncelik)

Bunun anlamı, bir değişken aynı isimle birden fazla yerde tanımlanmış olsa bile, **en yüksek önceliğe sahip tanım** en sonunda geçerli olur.

### 6. Niçin Bu Kadar Seçenek Var?

* **Grup Değişkenleri** ile aynı rolü üstlenen (örneğin hepsi “web server” olan) birden çok makineyi hızlıca ortak değerlerle donatabilirsiniz.
* **Host Değişkenleri** ile nadir durumlarda spesifik bir makine için farklı bir parametre belirleyebilirsiniz.
* **Playbook İçinde Tanımlı Değişkenler**, play’e özel anlık değişiklikler veya senaryoya göre atama yapmak istediğinizde size esneklik sağlar.
* **Extra Vars**, en kritik ve öncelikli bilgileri çalışma anında `ansible-playbook` komutu üzerinden iletmenizi sağlar.

Bu mantık, büyük ölçekli projelerde bile aynı şekilde geçerli kalır. Hangi değişkenin hangi yerden geldiğini iyi organize ederseniz, konfigürasyon yönetimini daha rahat yaparsınız.

***

### Bonus

#### 1. `vars_files`

* **Nedir?**\
  Playbook’larınızda `vars_files:` ifadesiyle harici bir YAML dosyası içindeki değişkenleri yükleyebilirsiniz.
*   **Nasıl Kullanılır?**\
    Örneğin bir playbook içinde şu satırla harici değişken dosyasını dahil edebilirsiniz:

    ```yaml
    - hosts: all
      vars_files:
        - vars/main.yml
      tasks:
        ...
    ```

    Böylece `vars/main.yml` adlı dosyada tanımlı bütün değişkenler bu playbook’ta kullanılabilir hale gelir.
* **Avantajı?**\
  Değişkenleri ayrı bir dosyada tutarak playbook’unuzu daha düzenli ve okunabilir kılarsınız. Ayrıca aynı `vars_files` dosyasını birden çok playbook’ta yeniden kullanabilirsiniz.

***

#### 2. `set_fact`

* **Nedir?**\
  `set_fact`, bir **task** sırasında dinamik olarak yeni bir değişken oluşturmanızı (veya mevcut bir değişkeni güncellemenizi) sağlar.
*   **Nasıl Kullanılır?**\
    Basit bir örnek:

    ```yaml
    - name: Set dynamic variable
      set_fact:
        my_dynamic_var: "Value calculated at runtime"
    ```

    Bu task tamamlandığında, `my_dynamic_var` adlı değişkenin değeri, ileride gelen diğer task’lar tarafından **aynen bir değişken gibi** kullanılabilir.
* **Neden Kullanılır?**\
  Çalışma esnasında ortaya çıkan bir sonucu (örneğin komut çıktısı veya koşullu bir değeri) saklamak ve sonradan kullanmak için idealdir. Varsayılan değişken tanımlama yöntemlerine göre daha dinamik bir yaklaşım sunar.

***

#### 3. `include_vars`

* **Nedir?**\
  `include_vars`, genellikle bir task içerisinde başka bir YAML dosyası içindeki değişkenleri “içe aktarmanızı” sağlar. `vars_files`’e benzer, fakat playbook’un bir **task** adımı olarak çalışmasıyla devreye girer.
*   **Nasıl Kullanılır?**\
    Örnek bir kullanım:

    ```yaml
    - name: Include variables
      include_vars:
        file: extra_vars.yml
    - name: Print included variable
      debug:
        var: included_var
    ```

    Böylece `extra_vars.yml` dosyasında tanımladığınız `included_var` gibi değişkenler, `debug` ile görüntülenebilir.
* **Farkı Nedir?**\
  `vars_files:` daha çok playbooks seviyesinde “en başta” yüklenen değişkenler içindir. `include_vars` ise bir **task** aşaması olarak istediğiniz zaman (mesela koşullu olarak) yükleme yapmanıza olanak tanır.

