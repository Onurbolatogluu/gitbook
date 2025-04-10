---
icon: text-size
---

# Inventory Formats

Ansible’da **inventory**, yönetmek istediğimiz sunucuların listesini tutan bir dosyadır. Bu dosyadaki format, **INI** veya **YAML** olabilir. Ansible’ın iki ana formatı desteklemesinin nedeni, farklı ölçek ve karmaşıklıktaki projelerin ihtiyacına göre esneklik sağlamaktır.

#### INI Formatı

* **Küçük Bir Startup Örneği**:
  * Henüz birkaç sunucu (web, database vb.) yönetiliyor.
  * INI formatı burada **temel bir organizasyon şeması** gibi iş görüyor.
  * `[web]`, `[db]` grupları açıp altlarına sunucularını yazarsınız.
* **Basit, Hızlı ve Kolay**: Birkaç satırlık tanımla hemen çalışmaya hazır.

Örnek:

```ini
[web]
web_node1 ansible_host=web01.xyz.com ansible_connection=ssh ansible_user=ubuntu

[db]
sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root
```

Bu yapı, küçük bir organizasyonu (ör. 2-3 makineden oluşan bir startup) rahatlıkla idare eder.

#### YAML Formatı

* **Büyük Bir Şirket (Multinational) Örneği**:
  * Yüzlerce sunucunuz olabilir: ABD, Avrupa, Asya; web, db, cache, load balancer vb.
  * YAML formatı sayesinde **alt gruplar** ve çok katmanlı yapılar oluşturmak kolaylaşır.
  * Okunabilirlik ve hiyerarşik yapı (indentation) açısından avantajlıdır.
* **Daha Esnek ve Yapılandırılmış**:
  * Gruplar içinde alt gruplar oluşturabilir, her sunucuya alias, port, özel parametreler ekleyebilirsiniz.

Örnek (YAML biçiminde):

```yaml
all:
  children:
    web_nodes:
      hosts:
        web_node1:
          ansible_host: web01.xyz.com
          ansible_user: ubuntu
        web_node2:
          ansible_host: web02.xyz.com
          ansible_user: ubuntu
    db_nodes:
      hosts:
        sql_db1:
          ansible_host: sql01.xyz.com
          ansible_user: root
```

Bu örnekte “web\_nodes” ve “db\_nodes” gibi alt gruplar, en üstteki “all” grubunun çocukları (children) olarak tanımlanmıştır. Çok daha karmaşık ortamlarda bölgelere, veri merkezlerine, rollere göre ayrım yapmak kolaylaşır.

***

### 1. Sunucuları ve Grupları YAML Formatında Tanımlama

```yaml
all:
  children:
    db_nodes:
      hosts:
        sql_db1:
          ansible_host: sql01.xyz.com
          ansible_connection: ssh
          ansible_user: root
          ansible_password: Lin$Pass

        sql_db2:
          ansible_host: sql02.xyz.com
          ansible_connection: ssh
          ansible_user: root
          ansible_password: Lin$Pass

    web_nodes:
      hosts:
        web_node1:
          ansible_host: web01.xyz.com
          ansible_connection: winrm
          ansible_user: administrator
          ansible_password: Win$Pass

        web_node2:
          ansible_host: web02.xyz.com
          ansible_connection: winrm
          ansible_user: administrator
          ansible_password: Win$Pass

        web_node3:
          ansible_host: web03.xyz.com
          ansible_connection: winrm
          ansible_user: administrator
          ansible_password: Win$Pass

    boston_nodes:
      hosts:
        # Bu grup, 'sql_db1' ve 'web_node1' makine alias'larını içeriyor.
        sql_db1: {}
        web_node1: {}

    dallas_nodes:
      hosts:
        # Bu grup, 'sql_db2', 'web_node2' ve 'web_node3' alias'larını içeriyor.
        sql_db2: {}
        web_node2: {}
        web_node3: {}

    us_nodes:
      children:
        # 'us_nodes' grubu, 'boston_nodes' ve 'dallas_nodes' gruplarını alt grup (children) olarak tutar.
        boston_nodes:
        dallas_nodes:
```

#### Neden Bu Şekilde Yazıyoruz?

1. **`all` → `children`**
   * En tepede `all` grubu, onun altında “db\_nodes”, “web\_nodes”, “boston\_nodes” vb. gibi alt grupları (`children`) tanımlıyoruz.
2. **db\_nodes / web\_nodes**
   * Bu grupların altında, her bir makine “alias” ismiyle (`sql_db1`, `web_node1` vb.) tanımlanıyor.
   * Hedef sunucunun gerçek adresi (`ansible_host`), bağlantı türü (`ansible_connection`), kullanıcı ve parola bilgileri burada giriliyor.
3. **boston\_nodes / dallas\_nodes**
   * Burada sadece `hosts` alanında hangi alias’ların bu gruba dahil olduğunu yazıyoruz.
   * Çünkü `sql_db1` ve `web_node1` zaten üstte ayrıntılı parametreleriyle tanımlandı. Ansible, aynı alias’ı gördüğünde parametreleri birleştirerek yönetir.
4. **us\_nodes**
   * `us_nodes` doğrudan sunucuları değil, başka grupları (boston\_nodes, dallas\_nodes) **children** olarak referans gösteriyor.
   * Böylece `ansible-playbook -i inventory.yaml -l us_nodes myplaybook.yml` dendiğinde, otomatik olarak hem boston\_nodes hem dallas\_nodes altındaki tüm sunucular hedeflenir.

***

### 2. Nasıl Kullanılır?

* **Inventory Dosyası**: Yukarıdaki içeriği `/home/bob/playbooks/inventory` (ya da `inventory.yaml`) dosyasında saklayın.
*   **Playbook Çalıştırma** (Örnek):

    ```bash
    ansible-playbook -i /home/bob/playbooks/inventory my_playbook.yml
    ```

    Böylece `db_nodes`, `web_nodes`, `boston_nodes`, `dallas_nodes`, `us_nodes` gibi tanımladığınız **tüm gruplar** Ansible tarafından görülebilir.
*   **Belirli Bir Gruba Hedeflemek**:

    ```bash
    ansible-playbook -i /home/bob/playbooks/inventory my_playbook.yml -l db_nodes
    ```

    Bu komut, sadece `db_nodes` grubundaki (`sql_db1`, `sql_db2`) makinelerde çalışır.

***

#### Özet

1. **Alias ve Parametreler**: `ansible_host`, `ansible_connection`, `ansible_user`, `ansible_password` (Linux için `ssh`, Windows için `winrm` vb.).
2. **Gruplar**: `db_nodes`, `web_nodes` gibi doğrudan “hosts” tanımlı gruplar.
3. **Alt Gruplar**: `boston_nodes`, `dallas_nodes` sunucu alias’larını yeniden listeler (parametreler yukarıdan birleştirilir).
4. **Üst Grup**: `us_nodes`, “boston\_nodes” ve “dallas\_nodes” gruplarını children olarak içerir.

Bu şekilde, tabloda belirtilen tüm sunucu bilgileri (OS, kullanıcı, parola) bir **YAML** envanter dosyasında net bir şekilde temsil edilmiş olur.
