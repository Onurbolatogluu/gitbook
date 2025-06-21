---
icon: j
---

# Jinja2 Templates for Dynamic Configs

### 1. Basit Senaryo: HTML Dosyası Kopyalama

#### 1.1 Statik Kopyalama (Önceki Durum)

Diyelim ki `index.html` adında yerel bir dosyamız var ve bunu Ansible’ın `copy:` modülüyle her bir web sunucusuna `/var/www/nginx-default/index.html` konumuna kopyalıyoruz.

**inventory (hosts) dosyası** (örnek):

```ini
[web_servers]
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101
web3 ansible_host=172.20.1.102
```

**playbook.yml** (statik kopya yaklaşımı):

```yaml
- hosts: web_servers
  tasks:
    - name: Copy index.html to remote servers
      copy:
        src: index.html
        dest: /var/www/nginx-default/index.html
```

Bu playbook, `index.html` dosyasını **olduğu gibi** kopyalar. Sonuçta her sunucuda aynı içerik olur. Eğer her sunucu için özel bilgi (hostname vb.) göstermek istersek, statik bir dosya yeterli olmaz.

### 2. Dinamik İçerik: Jinja2 Şablonu

**Hedef**: Her sunucuda HTML sayfası “`This is HOSTNAME Server`” şeklinde hostun adını göstersin.

1. **index.html** dosyamızı “template” mantığında kullanmak için Jinja2 sözdizimine uyarlıyoruz.
2. Değişkenleri `{{ ... }}` formatında belirtiyor, dosyanın uzantısını `*.j2` yapıyoruz (örneğin: `index.html.j2`).

#### 2.1 Jinja2 Şablon Dosyası (`index.html.j2`)

```jinja
<!DOCTYPE html>
<html>
<body>
This is {{ inventory_hostname }} Server
</body>
</html>
```

Açıklama:

* `{{ inventory_hostname }}` Ansible’daki **her hostun** en temel değişkenlerinden biridir (host ismini tutar).
* Bu şablon dosyası, “gerçek” HTML değildir; içindeki `{{ ... }}` ifadeleri Ansible/Jinja2 tarafından işlenince HTML haline gelecektir.

### 3. Template Modülüyle Dağıtma

Artık `copy:` modülü yerine `template:` modülünü kullanacağız, çünkü `template:` modülü `.j2` dosyasını açar, içindeki değişkenleri/koşulları Jinja2 motoru aracılığıyla çözümler ve **işlenmiş** bir dosyayı hedef sunucuya kopyalar.

#### 3.1 Güncellenmiş Playbook (playbook.yml)

```yaml
- hosts: web_servers
  tasks:
    - name: Copy index.html to remote servers (using template)
      template:
        src: index.html.j2
        dest: /var/www/nginx-default/index.html
```

**Önceki** `index.html` yerine artık `index.html.j2` kullanıyoruz. Ansible, bu şablonu her hosta özel değerlerle dolduracak.

1. `src: index.html.j2`: Yerel kontrol makinemizdeki Jinja2 şablon.
2. `dest: /var/www/nginx-default/index.html`: Uzaktaki dizin.

#### 3.2 Çalıştırma

```bash
ansible-playbook playbook.yml
```

* Ansible her web server için `index.html.j2` dosyasını açar, `inventory_hostname` değerini (mesela “web1” / “web2” / “web3”) yerleştirir.
* Sonunda `/var/www/nginx-default/index.html` içerik olarak “`This is web1 Server`” (web1’de), “`This is web2 Server`” (web2’de) vb. haline gelir.

**Sonuç**: Artık her sunucuda kendine ait bir `index.html` var. Web tarayıcıdan `http://<web1_ip>/` açtığımızda “This is web1 Server” görürüz.

### 4. Jinja2 Filtreleri ve Gelişmiş Özellikler

Sadece `{{ inventory_hostname }}` kullanmakla kalmayabiliriz; **filtreleri** kullanarak stringi büyütebilir, YAML/JSON dönüşümü yapabilir, vs.

Örnek küçük değişiklik:

```jinja
This is {{ inventory_hostname | upper }} Server
```

Üstteki kod `web1` yerine `WEB1` (büyük harfle) yazacaktır.

***

### 5. Daha Karmaşık Senaryolar: Koşullar ve Döngüler

Jinja2, `for` veya `if` gibi yapıları da destekler.

**Örnek**: Birtakım DNS kayıtlarını döngüyle yazmak (ör. `resolv.conf.j2`):

```jinja
{% for ns in name_servers %}
nameserver {{ ns }}
{% endfor %}
```

Playbook’ta:

```yaml
vars:
  name_servers:
    - 8.8.8.8
    - 1.1.1.1
```

Her `ns` değeri satır satır eklenir.

***

### 6. Daha Fazla Bilgi ve Dokümantasyon

* Ansible dokümantasyonunda Templating with Jinja2 bölümü,
* Jinja2 resmi dokümantasyonu (tüm filtreler, kontrol akışları vb.),
* Ansible’a özgü ek filtreler (örn. `win_basename`, `to_yaml`, `json_query`, vb.) \[ansible-doc -t filter --list] ile veya docs.ansible.com’da bulunabilir.

***

### 7. Özet

1. **copy:** statik dosyayı olduğu gibi kopyalar.
2. **template:** `.j2` uzantılı Jinja2 şablonu alır, Ansible değişkenlerini yerleştirir ve hedefe **işlenmiş** dosya olarak yazar.
3. `{{ inventory_hostname }}` gibi yerleşik değişkenler veya kendi tanımladığınız vars’lar kullanılabilir.
4. Filtreler (`| upper`, `| default("6379")` vs.) ve blok yapıları (`{% for ... %}`, `{% if ... %}`) ile dosyalarınız **dinamik** hale gelir.
5. Bu yöntemle her hosta özel konfig dosyaları, web sayfaları vb. **kolayca** oluşturulur.

**Unutmayın**: Templating Ansible’da pek çok yerde geçer. Bir dosyayı “template” modülüyle dağıtmak en tipik senaryo. Gelişmiş filtre, döngü ve koşullarla karmaşık yapılandırmaları bile **tek bir şablon** dosyasıyla yönetebilirsiniz.
