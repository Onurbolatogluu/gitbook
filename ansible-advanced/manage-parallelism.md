# Manage parallelism

### 1. Strategy nedir?

<table><thead><tr><th width="119.7816162109375">Strategy</th><th width="152.4893798828125">Varsayılan mı?</th><th>Ne yapar?</th></tr></thead><tbody><tr><td><strong>linear</strong></td><td>✔️</td><td>Aynı task’ı tüm hostlarda <em>paralel</em> çalıştırır; herkes bitmeden sonraki task’a geçmez.</td></tr><tr><td><strong>free</strong></td><td>❌</td><td>Her host kendi yolunda ilerler; bir task bitince beklemez, direkt sıradakine geçer.</td></tr><tr><td><strong>serial</strong></td><td>(linear’ın ayarı)</td><td>Kaç hostluk “batch” işleneceğini söyler; örneğin <code>serial: 3</code> → üçer üçer.</td></tr></tbody></table>

### 2. Linear (default) – “Hep beraber, adım adım”

```yaml
- name: Deploy web app
  hosts: webservers
  tasks:
    - name: Install dependencies
      # ...
    - name: Run webserver
      # ...
```

* Aynı task eş zamanlı tüm hostlarda çalışır.
* En yavaş host, “barajı geçen son atlet” olur; diğerleri onu bekler.
* **Avantaj:** Tutarlı ilerler, aynı adımda herkes aynı durumda.
* **Dezavantaj:** Yavaş host = tüm playbook yavaşlar.

### 3. Free – “Herkes kendi temposunda”

```yaml
- hosts: webservers
  strategy: free
  tasks:
    - name: Install dependencies
      # ...
    - name: Run webserver
      # ...
```

* Host A birinci task’ı bitirince hemen ikincisine geçer; Host B hâlâ birincide olabilir.
* **Avantaj:** Toplam süre kısalabilir.
* **Risk:** Adımlar arasında bağımlılık varsa (örneğin DB henüz yokken app servis başlarsa) sorun çıkar.

### 4. Serial – “Parça parça ilerle”

```yaml
- hosts: webservers          # diyelim 10 host
  serial: 3                  # 3’lük batch'ler
  tasks:
    - name: Deploy app
      # ...
```

Çalışma akışı:

1. İlk 3 host üzerinde tüm task’lar biter.
2. Sonra sonraki 3 host, … derken 10 host tamamlanır.

> **Kullanım alanı:**
>
> * Prod cluster’da “az az güncelle, bak bir şey kırılıyor mu?”
> * Network/load azaltmak.

**Dinamik seri:** İki değer verebilirsin: `serial: [2, 4]`

* İlk tur 2 host, başarılıysa her turda 4 host şeklinde büyür. Yükseltme rollout’larında işe yarar.

### 5. Parallel sürecin motoru: Forks

* Ansible, her host için bir **fork (alt proses)** açar.
* **Default:** `forks = 5` ( `/etc/ansible/ansible.cfg` içinde).
* 100 host hedeflesen bile; ilk 5’i biter → sonraki 5, vs.

#### Değiştirme yolları

1. **Geçici:** `ansible-playbook -f 20 site.yml`
2. **Kalıcı:** `forks = 20` satırını `ansible.cfg` altına ekle.

> Kaynak güçlü, ağ hızlıysa arttır; aksi hâlde SSH bağlantı sayısı yüzünden CPU/net boğulur.

### 6. Strateji + Serial + Forks nasıl birleşir?

* `serial` batch boyutunu sınırlar.
* `forks` ise **aynı anda** kaç host’la konuşabileceğini sınırlar.\
  Örnek:

```ini
forks = 10
```

```yaml
serial: 20
```

Inventoride 100 host var ➜ Her batch’te 20 host işlenecek, ama aynı anda maksimum 10’u aktif olacak.

| Durum                                    | Öneri                                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Canlı sistem, riskli update**          | `serial: 1` + `strategy: linear` ↗ sırayla güncelle, sorun çıkarsa hemen fark et.           |
| **Lab ortamı, hız önemli**               | `strategy: free`, `forks`’u yükselt.                                                        |
| **DB kurulum + app kurulum tek play’de** | DB blok’u bitmeden app başlatmak tehlikeli → `linear` kullan ya da **iki ayrı play** yaz.   |
| **Özel ihtiyaç**                         | Python ile _strategy plugin_ yazarak tamamen kendi akışını oluşturabilirsin (ileri seviye). |

