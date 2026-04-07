---
icon: simplybuilt
---

# Demo

Teoriyi geride bıraktık, şimdi sıra eBPF'in sağladığı o muazzam observability kendi gözlerimizle görmekte. Bu laboratuvar çalışmasında, kapalı bir kutu gibi duran Kubernetes `cluster`'ımızın içini şeffaf bir cama dönüştüreceğiz.

Hubble'ı aktif edecek, metrikleri Prometheus ve Grafana'ya bağlayacak, ardından oluşturduğumuz güvenlik kurallarının (Network Policies) trafiği nasıl kestiğini saniye saniye izleyeceğiz.

***

### 🏗️ Demo Topolojisi ve Veri Akışı

Bu çalışmada sistemin arka planında şu akış gerçekleşecek:

```
  [ Admin Pod ] ──(HTTP İstekleri)──▶ [ Demo Pod ]
        │                                  │
        └──────( eBPF Sensörleri Yakalar )─┘
                        │
                [ Hubble Relay ] (Veriyi Toplar)
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
   [ Hubble UI ]   [ Grafana ]   [ Hubble CLI ]
  (Servis Haritası) (Metrikler)  (Canlı Terminal)
```

Hazırsanız terminal başına geçelim!

***

### 1. Mevcut Durumu Analiz Etmek (Ön Kontrol)

`cluster`'ımızda Cilium (v1.15.3+) halihazırda kurulu, Prometheus ve Grafana ise `cilium-monitoring` isimli `namespace` içinde bekliyor. Ancak Hubble bileşenleri varsayılan olarak kapalıdır.

Önce sistemin mevcut durumunu teyit edelim:

```bash
# Cilium'un durumunu kontrol edelim
cilium status
```

_Çıktıda `Hubble Relay: disabled` yazdığını göreceksiniz. Cilium sağlıklı çalışıyor ama veri toplamıyor._

Grafana ve Prometheus'un ayakta olduğundan emin olalım:

```bash
kubectl get all -n cilium-monitoring
```

_Her ikisinin de `Running` statüsünde olduğunu doğrulamalıyız._

***

### 2. Hubble'ı Uyandırmak (Helm Upgrade)

Şimdi `cluster`'a, "Artık uyuma, Relay'i, UI'ı ve Prometheus metriklerini aktif et" diyeceğiz. Bunu `helm upgrade` komutuyla, mevcut ayarları bozmadan (`--reuse-values`) tek seferde yapıyoruz:

```bash
helm upgrade cilium cilium/cilium --version 1.15.4 \
  --namespace kube-system \
  --reuse-values \
  --set hubble.enabled=true \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true \
  --set hubble.metrics.enableOpenMetrics=true \
  --set prometheus.enabled=true \
  --set operator.prometheus.enabled=true \
  --set hubble.metrics.enabled="{dns,drop,tcp,flow,port_distribution,icmp,httpV2:exemplars=true;labelsContext=source_ip,source_namespace,destination_ip,destination_namespace,destination_workload,traffic_direction}"
```

Birkaç dakika bekleyip durumu tekrar kontrol edelim:

```bash
cilium status
```

Artık `Hubble Relay: OK` ve `Hubble UI: OK` ibarelerini görmelisiniz.&#x20;

***

### 3. Hubble UI'ı Dışarı Açmak (NodePort ile)

Varsayılan kurulumda Hubble UI bir `ClusterIP` servisidir (Sadece `cluster` içinden erişilebilir). Grafik arayüzü tarayıcımızdan görebilmek için (eğer `port-forward` kullanmak istemiyorsak) servisi `NodePort`'a çevirmeliyiz.

```bash
# Servis objesini düzenleme modunda açalım
kubectl edit svc hubble-ui -n kube-system
```

Açılan editörde `spec` kısmını bulup `type: ClusterIP` satırını `type: NodePort` olarak değiştirelim ve dışarıdan erişeceğimiz portu (Örn: 30000) belirtelim:

```yaml
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 8081
      nodePort: 30000 # Dışarıdan erişeceğimiz port
  selector:
    k8s-app: hubble-ui
```

Artık tarayıcınızdan `http://<Node_IP_Adresiniz>:30000` adresine girerek interaktif hizmet haritasına ulaşabilirsiniz.

> ⚠️ Güvenlik Uyarısı: `NodePort` kullanmak, o portu dış dünyaya açmak demektir. Gerçek `production` ortamlarında bunun yerine şifrelenmiş (HTTPS) ve kimlik doğrulamalı bir `Ingress` kullanmalısınız.

***

### 4. Trafik Simülasyonu ve Politikaları Test Etmek

Görselleştirilecek veri yaratmak için sisteme biraz trafik sokalım. Bir önceki konudan hatırlayacağınız üzere; `/api` yoluna (path) sadece `X-API-KEY: abc123` başlığı (header) olan `admin` pod'ları erişebiliyordu.

Bu kuralı (`CiliumNetworkPolicy`) uyguladıktan sonra, sistemi test edelim:

✅ Başarılı İstek (İzin Verilen):

```bash
kubectl run --rm -i --tty admin --labels=app=admin \
  --image=curlimages/curl --restart=Never -- \
  curl -H "X-API-KEY: abc123" http://app-svc-80/api

# Çıktı: {"message":"Have a great day!","method":"GET","url":"/api"}
```

❌ Başarısız İstek (Engellenen):

API anahtarını eklemeden istek atalım:

```bash
kubectl run --rm -i --tty admin --image=curlimages/curl \
  --restart=Never -- curl http://app-svc-80/api --connect-timeout 2

# Çıktı: Timeout was reached (Çünkü Cilium paketi sessizce DROP etti)
```

***

### 5. İzleme: Hubble UI ve Grafana'da Neler Oluyor?

Bu testleri yaptıktan sonra iki farklı ekrandan harika veriler okuyabilirsiniz:

* Grafana Paneli (`cilium-monitoring`): Dashboard'a girdiğinizde "Forwarded vs Dropped" (İletilen vs Düşürülen) grafiğinde, az önce API keysiz attığınız isteğin kırmızı bir "Drop" barı olarak yükseldiğini görebilirsiniz.
* Hubble UI (`<Node_IP>:30000`): `admin` ve `demo` kutucukları arasında gidip gelen okları göreceksiniz. Engellenen istekler haritada kırmızı bir hat olarak, başarılı olanlar ise yeşil olarak çizilir.

***

### 6. Hubble CLI: Terminalin Derinliklerine İnme

Grafik arayüzler harikadır, ancak sistem yöneticileri terminalde yaşamayı sever. Hızlı bir "troubleshooting" (sorun giderme) yapmak isterseniz, `cluster` içindeki herhangi bir Cilium ajanının (`pod`) içine girip doğrudan canlı akışı okuyabilirsiniz.

```bash
# Önce cilium-agent pod'unun içine girelim
kubectl exec -it -n kube-system cilium-xxxx -c cilium-agent -- /bin/bash

# Canlı ağ akışını (network flows) ekrana yazdıralım
hubble observe
```

Etkili Filtreleme Komutları:

Ekranda binlerce paket akıyorsa, aradığınızı bulmak için `hubble observe` komutunu harika parametrelerle daraltabilirsiniz:

```bash
# Sadece 'default' namespace içindeki trafiği izle:
hubble observe --namespace default

# Sadece 'admin' isimli pod'dan çıkan trafiği izle:
hubble observe --from-pod admin

# Sadece son 30 dakika içindeki olayları göster:
hubble observe --since 30m

# Logları JSON formatında formatlı şekilde bas (Otomasyon ve jq için mükemmeldir):
hubble observe -o json | jq .
```

