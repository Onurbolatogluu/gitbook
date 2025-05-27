---
icon: magnet
---

# Magic Variables

Ansible'da her bir sunucu (host) için değişkenler tanımlayabilirsin. Örneğin, bir `web2` adlı sunucun için özel bir DNS sunucusu IP adresi tanımladın. Normal şartlarda bu `dns_server` değişkeni sadece `web2` sunucusu üzerinde çalıştırılan görevler tarafından bilinir ve kullanılabilir. `web1` veya `web3` sunucuları bu bilgiye doğrudan erişemez.

İşte tam bu noktada **Magic Variables** devreye giriyor! 🪄

### Magic Variables Nedir?

**Magic Variables**, Ansible'ın sana sunduğu, önceden tanımlanmış ve her zaman erişebileceğin özel değişkenlerdir. Bu değişkenler, playbook çalışırken Ansible'ın kendisi tarafından otomatik olarak doldurulur. En büyük faydalarından biri, bir sunucunun başka bir sunucuya ait bilgilere veya genel envanter yapısına dair verilere ulaşmasını sağlamaktır.

Ansible bir playbook başlattığında her bir host için ayrı işlemler (sub-processes) oluşturur. Normalde her işlem sadece kendi host'una ait değişkenleri bilir. Magic variables bu sınırları aşmamızı sağlar.

### En Sık Kullanılan Magic Variables

#### 1. `hostvars`

Bu, belki de en sihirli olanıdır ✨ `hostvars` sayesinde **bir host, envanterdeki başka bir host'un değişkenlerine erişebilir.**

Diyelim ki envanter dosyan (`/etc/ansible/hosts`) şöyle:

```ini
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101 dns_server=10.5.5.4
web3 ansible_host=172.20.1.102
```

`web2` için bir `dns_server` tanımladık. Şimdi `web1` veya `web3` üzerinden bu bilgiye ulaşmak istersen, playbook'unda şöyle bir ifade kullanırsın:

```yaml
- name: Print DNS server IP from web2
  hosts: all
  tasks:
    - debug:
        msg: "Web2'nin DNS sunucusu: {{ hostvars['web2'].dns_server }}"
```

Bu görev çalıştığında, `web1`, `web2` ve `web3` sunucularının hepsi `web2`'nin `dns_server` değerini (yani `10.5.5.4`) ekrana basacaktır. Harika, değil mi?

Sadece kendi tanımladığın değişkenlere değil, Ansible'ın topladığı **facts** (sunucu hakkındaki bilgiler: işletim sistemi, IP adresi, mimari vb.) bilgilerine de `hostvars` ile erişebilirsin.

Örnek: `{{ hostvars['web2'].ansible_facts.architecture }}` ifadesi `web2` sunucusunun mimarisini (örn: x86\_64) döndürür.

Altyazılarda da belirtildiği gibi, şu iki yazım şekli de aynı sonucu verir:

* `hostvars['web2'].dns_server`
* `hostvars.web2.dns_server` (Eğer host adında özel karakterler yoksa bu daha kısa bir yoldur)

#### 2. `groups`

Bu magic variable, envanterindeki **grupları ve o gruplara dahil olan host'ların bir listesini** verir.

Diyelim ki envanterin şöyle:

```ini
[web_servers]
web1
web2
web3

[db_servers]
db1
```

Playbook'unda `{{ groups['web_servers'] }}` ifadesini kullanırsan, bu sana `['web1', 'web2', 'web3']` listesini döndürür. Ya da `{{ groups['all'] }}` tüm hostları listeler.

Örnek: `americas` grubundaki tüm host'ları yazdırmak için:

```yaml
- debug:
    msg: "{{ groups['americas'] }}"
```

#### 3. `group_names`

Bu değişken, o an görevin çalıştığı **host'un hangi gruplara üye olduğunu** bir liste olarak verir.

Örneğin, `web1` host'u hem `web_servers` hem de `americas` grubuna üye ise, `web1` üzerinde çalışan bir görevde `{{ group_names }}` ifadesi `['web_servers', 'americas']` listesini döndürür.

#### 4. `inventory_hostname`

Bu değişken, o an görevin çalıştığı **host'un envanter dosyasındaki adını** verir. Bu, sunucunun gerçek hostname'i (FQDN) olmak zorunda değildir; envanterde nasıl tanımladıysan o isimdir.

Örneğin, envanterde `webserver_ana` olarak tanımlı bir host varsa, `{{ inventory_hostname }}` ifadesi `webserver_ana` değerini döndürür, sunucunun kendi içindeki `hostname` komutunun çıktısını değil.

***

### Neden Önemliler?

Magic variables, playbook'larını daha dinamik ve esnek hale getirmeni sağlar:

* Bir sunucunun yapılandırmasını, başka bir sunucunun özelliklerine göre ayarlayabilirsin (örneğin, bir web sunucusuna veritabanı sunucusunun IP'sini bildirmek).
* Belirli bir gruptaki tüm sunuculara toplu işlemler yapabilirsin.
* Raporlama ve denetleme için host'lar arası bilgi toplayabilirsin.

Unutma, Ansible dokümantasyonu bu konuda en iyi dostundur. "Magic variables" veya "special variables" olarak arama yaparsan daha fazlasını bulabilirsin.
