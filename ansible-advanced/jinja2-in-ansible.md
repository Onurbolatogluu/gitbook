---
icon: puzzle-piece
---

# Jinja2 in Ansible

### 1) Hatırlatma: Jinja2 Nedir?

Daha önceki adımlarda gördüğümüz gibi **Jinja2**, değişkenleri ve şablon sözdizimini kullanarak metin dosyaları üretmemize izin veren bir şablonlama (templating) motorudur.

* `{{ variable }}` ile değişken yerleştirme
* `| upper`, `| default` gibi **filtreler** ile veriyi dönüştürme
* `{% if %}`, `{% for %}` gibi yapılarla koşul/döngü

**Ansible** ise bu motoru kullanarak playbook dosyaları, konfigürasyon şablonları (template) vb. içinde değişkenleri otomatik olarak yerleştirir.

### 2) Ansible’ın Jinja2’ye Eklediği Filtreler

Jinja2 aslında Python için yazılmış bir kütüphanedir. Ansible, **altyapı yönetimine** özel gereksinimleri karşılamak için bunu **genişletmiş**, fazladan filtreler eklemiştir. Örneğin:

1. **YAML-JSON dönüşümleri** (`to_yaml`, `to_json`, `from_yaml`, `from_json`)
2. **Dosya yolu** (path) ve **dosya adı** (basename) işlemleri (Linux/Windows farklı)
3. **Şifreler**, **regex** (normal ifade) işlemleri vs.

Bu ek filtreler, _ansible-core_ veya _ansible_ belgelerinde “Ansible Filters” başlığı altında bulunabilir.

***

#### 2.1 Dosya Tabanlı Filtreler (Linux vs. Windows)

Örneğin, `/etc/hosts` gibi bir **tam yolu** (full path) elimizde varsa, **dosya adını** çekmek için:

```jinja
{{ "/etc/hosts" | basename }}
```

Çıktı: `hosts`

* Windows’ta ise yol yapısı `C:\Windows\System32\drivers\etc\hosts` şeklinde ters taksim (backslash) içerir. Bu nedenle Ansible, `win_basename` gibi özel filtreler sunar:

```jinja
{{ "C:\\Windows\\hosts" | win_basename }}
```

Çıktı: `hosts`

**win\_splitdrive**

*   Windows’ta sürücü harfi ve geri kalan yolun ayrılması için:

    ```jinja
    {{ "C:\\Windows\\hosts" | win_splitdrive }}
    ```

    Çıktı: `["C:", "\\Windows\\hosts"]` (bir liste döner; ilk eleman sürücü, ikinci eleman yol).
*   Yalnızca sürücü harfine ihtiyacınız varsa filtreyi zincirleyebilirsiniz:

    <pre class="language-jinja"><code class="lang-jinja"><strong>{{ "C:\\Windows\\hosts" | win_splitdrive | first }}
    </strong></code></pre>

    Çıktı: `C:`

### 3) Jinja2 Playbook’larda Nasıl Çalışıyor?

Bir **playbook** veya **template** dosyasında `{{ ... }}` gördüğünüz her yer, Ansible tarafından Jinja2 ile **işlenir**:

1. **Envanter (inventory)** ve **değişken dosyalarından (vars files)** topladığı değerleri Jinja2 motoruna iletir.
2. Jinja2, bu değerleri `{{ variable }}` yerlerine koyar ve `| filter` ifadelerini uygular.
3. Sonuçta **gerçek** playbook veya şablon dosyası, **değişkenler yerleştirilmiş** halde Ansible tarafından yürütülür.

Örnek:

```ini
# inventory
web1 ansible_host=192.168.1.10 file_path=/etc/hosts
```

```yaml
# playbook.yml
- name: Show filename
  hosts: web1
  tasks:
    - debug:
        msg: "The file is: {{ file_path | basename }}"
```

Ansible playbook çalışırken, `basename` filtresi devreye girer, `/etc/hosts` → `hosts` şeklinde dönüşür ve ekrana:

```
TASK [debug] ******************************************************************
ok: [web1] => {
    "msg": "The file is: hosts"
}
```

çıktısı basılır.

### 4) Filtreleri Nerede Bulurum?

* Ansible resmi belgelerinde (“Ansible Filters” veya “Jinja2 Filters” başlığı altında) pek çok filtre örneği bulabilirsiniz.
* `yaml` ve `json` çevirileri, `regex_replace`, `ipaddr`, `hashing (md5, sha256)` gibi bir dizi ek filtre mevcuttur.
