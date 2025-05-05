---
icon: rectangle-history
---

# Ansible Collections

### 1. Ansible Collections Nedir, Ne İşe Yarar?

Ansible “Collection” (koleksiyon), **modüller**, **roller**, **eklentiler** (plugins) ve ilişkili diğer öğelerin **tek bir paket** (self-contained) halinde dağıtılmasını sağlayan bir yapı sunar.

* **Örnek**: “network.cisco” adında bir koleksiyon, Cisco cihazlarını yönetmek için özel modüller, roller vb. içerebilir.
* Kısacası, “**Bir pakette** vendor’a/belirli bir amaca özel tüm Ansible içeriklerini toplamak ve paylaşmak” demektir.

**Neden Gerekli?**

* Ansible’ın kendi çekirdek modülleri her durumda yeterli olmayabilir (özelleşmiş vendor veya bulut API’lerine ihtiyaç duyabilirsiniz).
* Collections, **topluluk** veya **vendor** tarafından yazılmış gelişmiş/özel fonksiyonları kolayca yükleyip **kullanmanızı** sağlar.

### 2. Koleksiyonların Faydaları

1. **Genişletilmiş Fonksiyonellik**
   * Örneğin, bulut sağlayıcı (AWS, Azure) veya network vendor (Cisco, Juniper) modülleri, Ansible core’da sınırlı olabilir.
   * Bir koleksiyon yükleyerek, “cloud.xyz” veya “network.abc” adıyla ekstra modüllere/rollere erişirsiniz.
2. **Modülerlik ve Tekrar Kullanım**
   * Koleksiyonlar, “rol”, “modül” ve “plugin” gibi parçaları **bir arada** paketler.
   * Böylece bir projede kullanırken, **başka projede** de aynı koleksiyonu ekleyip tekrar kullanabilirsiniz.
3. **Dağıtım ve Versiyon Yönetimi**
   * “requirements.yml” dosyasında hangi koleksiyonların hangi sürümlerini istediğinizi tanımlayabilir, tek komutla hepsini kurabilirsiniz.
   * Bu, sürüm çakışmalarını veya yanlış modül sürümü gibi hataları önler.

### 3. Örnek Kullanım: Network Cihazları (Cisco, Juniper, vs.)

1. **Senaryo**: Büyük bir ağ altyapısında Cisco, Juniper, Arista cihazlarını yönetmek istiyorsunuz.
2. **Varsayılan** Ansible network modülleri yetmeyebilir.
3. Her vendor’ın “network.cisco”, “network.juniper” benzeri koleksiyonları olabilir.
4. Koleksiyonu yükleyerek (örn. `ansible-galaxy collection install network.cisco`), vendor’a özgü modülleri elde edersiniz.
5. Artık `network.cisco` koleksiyonu içindeki modülleri playbook’larınızda doğrudan çağırabilirsiniz (örn. “cisco\_ios\_config” gibi).

### 4. Koleksiyon Nasıl Kurulur ve Kullanılır?

#### 4.1 Basit Kurulum

```bash
ansible-galaxy collection install network.cisco
```

Bu komut, `network.cisco` koleksiyonunu indirip sisteminizdeki koleksiyon dizininde (örn. `~/.ansible/collections/`) saklar.

#### 4.2 Playbook’ta Kullanmak

```yaml
- name: Configure Cisco
  hosts: cisco-devices
  collections:
    - network.cisco
  tasks:
    - name: Example Cisco command
      cisco.ios.ios_config:  # buradaki "cisco.ios.ios_config" modülü koleksiyon içinden gelir
        lines:
          - hostname Router1
```

**collections:** altında “network.cisco” belirterek bu koleksiyondaki modülleri (ör. `cisco.ios.ios_config`) kullanabilirsiniz.

#### 4.3 requirements.yml ile Versiyon Belirtmek

```yaml
collections:
  - name: network.cisco
    version: "2.1.0"
  - name: amazon.aws
    version: "3.5.0"
```

*   Sonra:

    ```bash
    ansible-galaxy collection install -r requirements.yml
    ```
* Tüm koleksiyonlar istenen sürümlerde kurulur.

### 5. Kendi Koleksiyonunuzu Oluşturmak

* İhtiyaç duyduğunuz modüller, roller veya plugin’ler varsa **kendi namespace** altında bir koleksiyon oluşturabilirsiniz (örn. “my\_namespace.my\_collection”).
* Bu koleksiyonu `ansible-galaxy collection init my_namespace.my_collection` komutuyla başlatır, oluşturulan dizin yapısına kodlarınızı yerleştirir, sonra ister lokal projelerde kullanır ister Galaxy’de paylaşırsınız.

{% embed url="https://docs.ansible.com/ansible/latest/collections/index.html" %}

#### Colections Example,

Aşağıda **basit** bir Ansible **collection** kullanım örneği gösteriyorum. Bu örnekte, topluluk tarafından sağlanan **“community.general”** adlı koleksiyonu kullanıp bir görevi yerine getireceğiz. “community.general” koleksiyonunda pek çok ek modül yer alır (örneğin, zamanla ilgili bir modül veya sistemle ilgili ekstra komutlar).

***

### 1. Collection’ı Kurmak

```bash
# Terminalden çalıştırın:
ansible-galaxy collection install community.general
```

Bu komut, `community.general` koleksiyonunu varsayılan koleksiyon dizinine indirir (ör. `~/.ansible/collections/` veya `roles_path` / `collections_paths` ayarlarınıza göre).

***

### 2. Basit Playbook Örneği

Aşağıda örnek bir **playbook** var; bu playbook’ta `community.general` koleksiyonu içindeki bir modülü (örnek olarak `community.general.shell` gibi) kullanacağız.

```yaml
# file: playbook.yml
- name: Demo of using community.general collection
  hosts: localhost
  collections:
    - community.general    # <-- Koleksiyonu burada belirtiyoruz
  gather_facts: no
  tasks:
    - name: Example usage of a module from community.general
      # Örnek olarak 'community.general.shell' modülü 
      # (aslında builtin 'shell'e benzer, ama genişletilmiş fonksiyonellik sunabilir)
      community.general.shell: 
        cmd: "echo 'Hello from community.general!'"
        chdir: /tmp
      register: shell_output

    - name: Show output
      debug:
        var: shell_output.stdout
```

1. **collections:** altında `community.general` yazdığımız için, bu playbook o koleksiyondaki modülleri arar ve kullanabilir.
2. `community.general.shell:` satırında, koleksiyonun içindeki `shell` modülünü çağırıyoruz (örneğin, ek parametre veya gelişmiş seçenekler sunuyor olabilir).
3. Komut çalıştırıldıktan sonra sonucu `shell_output`’a kaydedip (`register: shell_output`), sonrasında `debug` ile ekrana basıyoruz.

#### Nasıl Çalıştırılır?

```bash
ansible-playbook playbook.yml
```

Çıktıda:

```
TASK [Example usage of a module from community.general] ************************
changed: [localhost]

TASK [Show output] **************************************************************
ok: [localhost] => {
    "shell_output.stdout": "Hello from community.general!"
}
```

Böylece `community.general` koleksiyonundan bir modülü **doğrudan** kullanmış olduk.

***

### 3. Kısa Özet

1. **Koleksiyonu Yükle**: `ansible-galaxy collection install <koleksiyon_ismi>`
2. **Playbook’ta Belirt**: `collections:` altında `<namespace>.<collection_name>` ifadesini ekleyin.
3. **Modülü Kullanın**: `<namespace>.<collection_name>.<modül>` şeklinde çağırabilirsiniz (veya eğer `collections:` satırında belirttinizse, doğrudan `collection_name.modul` ismi de çalışır).

Bu şekilde, koleksiyonlardaki ek modüller, roller ve plugin’lerden **kolayca** yararlanabilirsiniz.

