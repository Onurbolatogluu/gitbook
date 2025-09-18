---
icon: folder-tree
---

# Jenkins Architecture

**Jenkins** dağıtık (distributed) bir mimariyle çalışır:

* **Controller (Jenkins server)** → “beyin / orkestratör”
* **Nodes (worker machines)** → “işi yapan makineler”
* **Agents** → node’un controller’a **nasıl** bağlandığını (protokol/kimlik) ve **nerede** çalışacağını tanımlar
* **Executors** → her node üzerindeki **eşzamanlı iş çalıştırma slot’ları**

```
                 +----------------------+
                 |   Jenkins Controller |
                 |  (UI, jobs, plugins, |
  Webhook/SCM -> |   credentials, API)  |
                 +----------+-----------+
                            |
         Schedules jobs --->|  Queue & Label-based dispatch
                            |
      +---------------------+---------------------+
      |                                           |
+-----v-----+                               +-----v-----+
|  Node #1  |<-- agent (SSH/JNLP/Docker) -->|  Node #2  |
| executors |                               | executors |
| tools     |                               | tools     |
+-----------+                               +-----------+

```

## 2) Bileşenler

### Controller (Jenkins server)

* **İşlev**: kullanıcı/kimlik yönetimi (authentication/authorization), job/pipeline tanımlama, **scheduling**, plugin’ler, web UI, REST API.
* **Kural**: Prod ortamda controller üzerinde job koşturma **yapma**. Controller’ın **# of executors = 0** yap (sadece orkestrasyon yapsın).
* **Neden ayrı dursun?** Güvenlik (config’ler bozulmasın), performans (build yükü nodes’a gitsin), ölçeklenebilirlik (node ekledikçe kapasite artar).

### Nodes (Workers)

* **Ne?** Build’leri çalıştıran makineler (Linux/Windows/macOS fark etmez).
* **Bağlantı**: Controller’a **SSH** veya **JNLP (inbound)** ile bağlanır.
* **Executors**: Node üzerindeki “aynı anda kaç job?” sorusunun cevabı.
  * Ağır işler → **1 executor per node** (en güvenlisi).
  * Hafif işler → **1 executor per CPU core** kabul edilebilir.

### Agents

* **Anlamı**: Node’un controller’a **hangi protokolle** ve **hangi kimlikle** bağlandığını ve job’un **nerede** koşturulacağını belirler.
* Türler:
  * **SSH Agent**: Controller, node’a SSH ile bağlanır (dışa açık port gerekir).
  * **JNLP (Inbound) Agent**: Node, controller’a bağlanır (firewall arkasında kolay).
  * **Docker Agent**: Her build izole bir container’da çalışır → “temiz ve tekrarlanabilir” ortam.
  * **Kubernetes Agent (Pods)**: Her build için dinamik pod; peak yüklerde otomatik ölçeklenir.

### Executors

* **Thread** gibi düşün: aynı anda kaç job çalışır.
* Çok yüksek sayı → context switching/artış = “daha yavaş”. Az sayı → kaynak boş kalabilir. Node’un CPU/RAM’ine ve job tipine göre ayarla.

## 3) SSH vs JNLP (Ne zaman hangisi?)

* **SSH Agent (controller → node)**
  * Artı: Basit, SSH anahtarıyla yönetim kolay.
  * Eksi: Node’un SSH portu controller’dan erişilebilir olmalı.
* **JNLP/Inbound Agent (node → controller)**
  * Artı: NAT/firewall arkasında rahat; node dışarıya initiates connection.
  * Eksi: Java agent çalıştırırsın; network policy’e göre outbound izin gerekir.

Prod’da genelde **JNLP inbound** tercih edilir (özellikle cloud/VPC içinde).

## 4) Docker ve Kubernetes ile Agents

* **Docker Agent**
  * Jenkinsfile’da `agent { docker { image '...' } }`
  * Her build **izole** container’da → bağımlılık çatışmaları biter.
  * Örnek: Java 17 isteyen proje için `eclipse-temurin:17` image’i.
* **Kubernetes Agent**
  * Jenkins **Kubernetes plugin** ile her job için **pod** oluşturur.
  * On-demand scaling → **AKS** gibi ortamlarda muazzam.
  * Tooling’i pod template ile verirsin (maven, node, docker-cli sidecar vb.).

## 5) Job Akışı (Controller ↔ Nodes)

1. Job/Pipeline controller’da **tanımlanır** (UI/CLI/REST/Jenkinsfile).
2. Controller **queue**’ya alır, **label** uyumlu ve **boş executor**’ı olan node’u bulur.
3. Agent üzerinden node’da **workspace** oluşturulur ve job çalışır.
4. **Logs / Artifacts** node’dan controller’a raporlanır/arsivlenir (JUnit raporları, build artifacts).
5. Sonuç: **SUCCESS/FAILURE/UNSTABLE** + log + artifacts + metrics.

**İpucu**: Tool’ları (JDK, Maven, Node.js, Docker vb.) **Global Tool Configuration** üzerinden veya **container image** içinde standartlaştır.

## 6) Deployment Modelleri

* **Basic**: Controller + single node aynı VM → küçük POC/denemeler.
* **Recommended (prod)**:
  * Controller ayrı VM/pod, executors = 0
  * Birden çok **static node** (kalıcı VM) veya **ephemeral** (Docker/K8s)
  * **Labels** ile iş yükünü yönlendir: `linux`, `windows`, `docker`, `gpu`, `arm64`…

**Azure örneği**:

* Controller: Azure VM (ya da AKS’de stateful pod + external storage).
* Agents:
  * **AKS (Kubernetes)** pod’ları (dinamik) **veya**
  * VM Scale Set ile **ephemeral** Linux nodes (SSH veya inbound).



