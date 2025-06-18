---
icon: wagon-covered
---

# Dynamic Inventory

### 1. Neden statik yetmez?

* Küçük lab’da 3‐5 makine → `hosts` dosyasına elle yaz, bitti.
* AWS / GCP’de onlarca VM otomatik açılıp kapanıyor → IP’ler, etiketler sürekli değişiyor.
* Elle güncellemek hem zor hem hata davetiyesi.
  * _“Ansible play çalıştı ama makineyi bulamadı”_ 😖

### 2. Dinamik envanter nasıl çalışır?

1. **Kaynak** (CMDB, bulut API, kendi servisiniz) → JSON veya YAML olarak host listesi döndürür.
2. **Envanter eklentisi** (plugin) veya **envanter script’i** bu listeyi üretir.
3. `ansible-playbook -i <kaynak>` çağırınca Ansible önce bu kaynağı okur, sonra playbook’u uygular.

### 3. En basit demo – mini Python script

`inventory.py`:

```python
#!/usr/bin/env python3
import argparse, json, sys

inv = {
    "web_servers": { "hosts": ["web1", "web2"] },
    "_meta": {
        "hostvars": {
            "web1": { "ansible_host": "10.0.0.11" },
            "web2": { "ansible_host": "10.0.0.12" }
        }
    }
}

parser = argparse.ArgumentParser()
parser.add_argument('--list', action='store_true')
parser.add_argument('--host')
args = parser.parse_args()

if args.list:
    print(json.dumps(inv))
elif args.host:
    print(json.dumps(inv['_meta']['hostvars'].get(args.host, {})))
else:
    sys.exit(1)
```

Kullanım:

```bash
chmod +x inventory.py
./inventory.py --list       # tüm envanter JSON
./inventory.py --host web1  # tek host detay
ansible-playbook site.yml -i inventory.py  # play’i koş
```

> **Önemli:** Script mutlaka `--list` ve `--host` argümanlarını anlamalı, çıktı **JSON** olmalı.

### 4. “Ben script yazmak istemiyorum” dersen → **Inventory Plugin’leri**

<table><thead><tr><th width="118.3663330078125">Platform</th><th>Plugin/Script</th><th>Nasıl kurulur?</th><th>Örnek çağrı</th></tr></thead><tbody><tr><td>AWS EC2</td><td><code>amazon.aws.aws_ec2</code> plugin<br>(eski <code>ec2.py</code> script’i)</td><td><code>pip install amazon.aws</code> veya Ansible Collection</td><td><code>ansible-playbook site.yml -i aws_ec2.yml</code></td></tr><tr><td>GCP</td><td><code>google.cloud.gcp_compute</code></td><td><code>pip install google.cloud</code></td><td><code>ansible-inventory -i gcp_compute.yml --graph</code></td></tr><tr><td>VMware</td><td><code>community.vmware.vmware_inventory</code></td><td><code>pip install community.vmware</code></td><td>Kullanım benzer</td></tr></tbody></table>



***

### 5. YAML envanter dosyası (statik ama daha okunur)

```yaml
all:
  children:
    web_servers:
      hosts:
        web1:
          ansible_host: 10.0.0.11
        web2:
          ansible_host: 10.0.0.12
```

Bunu `-i inventory.yml` ile kullanırsın; plugin otomatik `yaml`’dır.

### 6. Envanteri test etmek: **ansible-inventory**

```bash
ansible-inventory -i inventory.py --list -y    # İnsan okuyabilir YAML
ansible-inventory -i aws_ec2.yml --graph       # Grup ağacını çiz
```

Debug sırasında _“Bu variable nereden gelmiş?”_ diyorsan en hızlı yol.

***

#### AWS EC2 için Dinamik Envanter - Adım Adım Süper Basit Rehber

Aşağıdaki 5 adımı uygula; kodları kopyala-yapıştır, çalıştır, bitti.

***

### 1. Gerekli paketleri kur

```bash
pip install boto3 botocore          # AWS Python SDK
ansible-galaxy collection install amazon.aws
```

> `amazon.aws` koleksiyonu içinde **aws\_ec2** envanter plug-in’i geliyor. [docs.ansible.com](https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html?utm_source=chatgpt.com)

***

### 2. AWS kimlik bilgilerini ayarla

En kolay yol: çevre değişkenleri (terminalde bir kere yaz, sonra playbook öncesi aynı shell’i kullan):

```bash
export AWS_ACCESS_KEY_ID=AKIAXXXXXXXX
export AWS_SECRET_ACCESS_KEY=abcd1234xxxxxxxx
export AWS_DEFAULT_REGION=eu-central-1      # Frankfurt örneği
```

***

### 3. Envanter dosyasını oluştur (YAML)

`aws_ec2.yml` **ismi önemli** — dosya adı “aws\_ec2.(yml|yaml)” olunca Ansible plug-in’i otomatik seçer. [stackoverflow.com](https://stackoverflow.com/questions/54563408/ansible-aws-ec2-inventory-plugin-issue?utm_source=chatgpt.com)

```yaml
plugin: amazon.aws.aws_ec2          # plug-in’in tam adı
regions:
  - eu-central-1                    # istersen birden çok
filters:                            # sadece etiket “Role = web” olan EC2’leri çek
  tag:Role: web
keyed_groups:                       # host'ları otomatik gruplandır
  - prefix: aws_region              # örnek: aws_region_eu_central_1
    key: placement.region
```

Dosyayı proje köküne (veya `inventory/` altına) koy.

***

### 4. Çıktıyı test et (debug)

```bash
ansible-inventory -i aws_ec2.yml --list -y     # tüm host & değişkenler
ansible-inventory -i aws_ec2.yml --graph       # grup ağacı
```

Bu komut, plug-in’in AWS API’ye bağlanıp canlı envanteri çektiğini gösterir. Sorun varsa burada görürsün. [docs.ansible.com](https://docs.ansible.com/ansible/latest/collections/amazon/aws/docsite/aws_ec2_guide.html?utm_source=chatgpt.com)

***

### 5. Playbook’u çalıştır

```bash
ansible-playbook -i aws_ec2.yml site.yml
```

* Playbook’taki `hosts:` kısmına `aws_region_eu_central_1` veya `web` gibi dinamik grupları yazabilirsin.
* Ansible her çalıştırmada API’den **güncel** listeyi çeker; yeni VM eklendiğinde ekstra iş yok.

***

#### Sık Sorulanlar

| Soru                                     | Cevap                                                                                                    |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **“Plugin’i göremiyorum”**               | \`ansible-galaxy collection list                                                                         |
| **Proxy/VPC yüzünden API’ye ulaşamıyor** | AWS CLI ile `aws ec2 describe-instances` komutu çalışıyorsa Ansible da çalışır; önceliği orada debug et. |
| **Birden çok etiketle filtre?**          | `filters: { tag:Environment: prod, tag:Role: web }` YAML sözlüğü olarak ekleyebilirsin.                  |
| **Dosyayı neden INI değil?**             | aws\_ec2 plug-in sadece YAML/içerik okur; INI yalnız statik envanter içindir.                            |

***

**Özet**

1. **amazon.aws** koleksiyonunu kur.
2. AWS kimlik bilgilerini ENV’e koy.
3. `aws_ec2.yml` içinde `plugin: amazon.aws.aws_ec2` satırını yaz, gerekli filtreleri ekle.
4. `ansible-inventory -i aws_ec2.yml --list` ile test et.
5. `ansible-playbook -i aws_ec2.yml ...` → dinamik envanter hazır.



