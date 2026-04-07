---
icon: seal
---

# Cilium Hubble

Bir önceki konumuzda CNI Network Policies ile `cluster`'ımızın kapılarına kilit vurduk ve trafiği sınırlandırdık. Ancak `production` ortamında işler her zaman yolunda gitmez. Bir uygulamanız aniden veritabanına bağlanamamaya başlarsa ne olacak? İstek `NetworkPolicy` tarafından mı engellendi (drop), DNS mi çözülemedi, yoksa TCP bağlantısında mı bir sorun var?

Ağınızda neler olup bittiğini göremiyorsanız, sorunları çözemezsiniz. Kilitleri taktık, şimdi sıra koridorlara 7/24 kayıt yapan gelişmiş bir güvenlik kamerası sistemi kurmaya geldi. İşte Cilium Hubble tam olarak budur.

***

### 🏗️ Hubble Nasıl Çalışır?

Hubble'ın mimarisi üç temel bileşenden oluşur. Bu yapıyı zihnimizde şu şekilde canlandırabiliriz:

```
       [ Node 1 ]                      [ Node 2 ]
    [Pod A] ──▶ [Pod B]             [Pod C] ──▶ [Dış Dünya]
       │           │                   │
    (eBPF Datapath / İşletim Sistemi Sensörleri)
       │           │                   │
       └───────────┼───────────────────┘
                   ▼
          [ Hubble Relay ] (Tüm node'lardan gelen veriyi toplar)
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
[ Hubble UI ]  [ CLI ]  [ Prometheus & Grafana ]
(Görsel Harita) (Terminal)  (Metrikler ve Alarmlar)
```

1. eBPF Datapath (Sensörler): Her `node` üzerinde çalışır. `Pod`'lar ağa bir paket gönderdiği an, bu paket eBPF sayesinde çekirdek seviyesinde yakalanır ve incelenir.
2. Relay: `Cluster` içindeki tüm `node`'lardan gelen bu devasa sensör verilerini tek bir merkezde toplar ve anlamlı hale getirir.
3. Integrations: Toplanan verileri UI (Arayüz), CLI (Komut Satırı) veya Prometheus gibi izleme araçlarına aktarır.

***

### 📊 Prometheus İçin Yerleşik Metrikler

Hubble, ağ trafiğinizi analiz eder ve standart OpenMetrics formatında dışarı aktarır. Bu metrikleri Prometheus ile toplayıp Grafana'da dashboard'lar çizebilirsiniz.

Hangi metrikleri izleyebiliyoruz?

* dns: DNS sorguları, hataları ve gecikmeleri (Örn: DNS hata oranı artarsa alarm üret).
* drop: Engellenen paketler (Yanlış yazdığınız bir policy yüzünden paketler drop ediliyorsa anında yakalarsınız).
* tcp: TCP bağlantı durumları, yeniden iletimler (retransmissions).
* flow: Genel trafik akışı, bant genişliği.
* port-distribution: Hangi portların ne kadar kullanıldığı.
* icmp: Ping (echo) istekleri ve yanıtları.
* httpV2: L7 seviyesinde HTTP/2 metrikleri ve etiketleri.

> 💡 İpucu: Sadece ihtiyacınız olan metrikleri aktif edin. Tüm metrikleri açmak, Prometheus üzerinde gereksiz veri yükü (storage/CPU) oluşturabilir.

***

### 🛠️ Helm ile Hubble ve Metrikleri Aktifleştirme

Cilium'u kurarken veya güncellerken, Hubble'ı ve Prometheus entegrasyonunu tek bir Helm komutuyla ayağa kaldırabiliriz.

_Kaynak dokümandaki eksiksiz komut şudur:_

```bash
helm upgrade cilium cilium/cilium --version CILIUM_VERSION \
  --namespace kube-system \
  --reuse-values \
  --set hubble.enabled=true \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true \
  --set hubble.metrics.enableOpenMetrics=true \
  --set prometheus.enabled=true \
  --set operator.prometheus.enabled=true \
  --set hubble.metrics.enabled="{dns,drop,tcp,flow,port-distribution,icmp,httpV2:exemplar=true;labelsContext=source_ip\,source_namespace\,source_workload\,destination_ip\,destination_namespace\,destination_workload\,traffic_direction}"
```

***

### 🖥️ Hubble UI

Hubble UI, `cluster`'ınızın adeta bir röntgenini çeker ve size interaktif bir Service Dependency Map (Servis Bağımlılık Haritası) sunar. Kimin kiminle konuştuğunu, hangi portların kullanıldığını oklarla görselleştirir. Ayrıca alt kısımdaki Flow Table ile saniye saniye akan trafiği (HTTP kodları, policy kararları) canlı izleyebilirsiniz.

Hubble UI'a Nasıl Erişilir?

Güvenlik gereği bu arayüzü internete açık bir Ingress ile dışarı açmak tehlikelidir. En güvenli yöntem, kendi bilgisayarınızdan `port-forward` yapmaktır:

```bash
# UI portunu kendi bilgisayarınıza (localhost) yönlendirin
cilium hubble ui
```

Bu komutu çalıştırdıktan sonra tarayıcınızdan `http://localhost:12000` adresine giderek arayüze ulaşabilirsiniz.

***

### 💻 Hubble CLI (Terminal Gücü)

Eğer grafik arayüzler yerine terminalde çalışmayı seviyorsanız veya logları script'lerle otomatize etmek istiyorsanız, Hubble CLI tam size göredir.

Yöntem 1: Doğrudan Pod İçinden Kontrol

Cilium agent `pod`'unun içine girerek Hubble'ın anlık sağlık durumunu görebilirsiniz:

```bash
kubectl exec -it -n kube-system cilium-xxxxxx -c cilium-agent -- hubble status
# Örnek Çıktı:
# Healthcheck (via unix:///var/run/cilium/hubble.sock): Ok
# Current/Max Flows: 4,095/4,095 (100.00%)
# Flows/s: 4.72
```

Yöntem 2: Hubble CLI'ı Kendi Linux Makinenize Kurmak

Sürekli `exec` yapmak yerine, CLI aracını doğrudan kendi makinenize kurarak `cluster` trafiğini kendi terminalinizden izleyebilirsiniz. Kurulum script'i:

```bash
# Hubble sürümünü çek ve işlemci mimarisine (amd64/arm64) göre ayarla
HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
HUBBLE_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then
  HUBBLE_ARCH=arm64
fi

# Dosyayı indir
curl -L --fail --remote-name-all \
  https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz \
  https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum

# Sıkıştırılmış dosyayı aç ve binary'i sistemin çalıştırılabilir dizinine taşı
sudo tar xvzf hubble-linux-${HUBBLE_ARCH}.tar.gz -C /usr/local/bin

# Temizlik
rm hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
```
