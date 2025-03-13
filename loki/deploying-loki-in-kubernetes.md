---
icon: caret-right
---

# Deploying Loki in Kubernetes

### 1. Grafana Helm Repo Eklenmesi

İlk adım, **Grafana**’nın Helm deposunu Kubernetes kümesine tanıtmaktır:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

* `helm repo add`: Belirtilen URL’yi “grafana” adıyla Helm repositories listesine ekler.
* `helm repo update`: Mevcut Helm repo’larınızı günceller, böylece en yeni chart sürümlerine erişebilirsiniz.

### 2. Loki ile İlgili Chart’ları Aramak

Repo eklendikten sonra, “Loki” anahtar sözcüğüyle ilgili chart’ları listeleyebilirsiniz:

```bash
helm search repo loki
```

Arama sonucunda “`grafana/loki-stack`”, “`grafana/loki-canary`”, “`grafana/loki-distributed`” gibi seçenekler görürsünüz. Burada:

* **loki-stack**: Tek seferde _Loki_, _Grafana_ ve _Promtail_ kurulumunu yapar.

### 3. Chart Değerlerini (values) İncelemek

Her chart, özelleştirilmesine izin veren bir `values.yaml` içeriğine sahiptir. `helm show values` komutuyla varsayılan değerleri görebilirsiniz:

```bash
helm show values grafana/loki-stack > values.yaml
```

Bu sayede `values.yaml` içeriğini metin editöründe inceleyip özelleştirebilirsiniz. Tipik olarak şunlar yer alır:

* **Loki** konfigürasyonu (ör. depolama, replikalar vb.)
* **Promtail** ayarları (hangi etiketlerle log toplayacağı, hangi node selector’ları vb.)
* **Grafana** bileşenini etkinleştirme veya devre dışı bırakma ( enable duruma getiriyoruz )
* Diğer opsiyonel kısımlar (örn. Fluent Bit entegrasyonu)

<figure><img src="../.gitbook/assets/Screenshot 2025-03-13 at 16.35.57.png" alt=""><figcaption></figcaption></figure>

### 4. `values.yaml` Özelleştirmesi

1. `grafana.enabled = true` yaparak **Grafana** bileşenini aktif hale getiriyoruz (varsayılan olarak `false` olabilir).
2. `grafana.image.tag = latest` diyerek Grafana’nın son sürümünün çekilmesini sağlıyoruz (daha sabit bir sürüm kullanmak isterseniz tag’i belirtebilirsiniz).
3. `promtail.enabled` varsayılan olarak zaten `true` görünüyor; bu sayede Promtail **DaemonSet** olarak kurulup her node’dan log toplayacak şekilde çalışıyor.

Bu ayarları kaydedip `values.yaml` dosyasından çıkıyoruz.

### 5. Loki Stack’i Kurma

Şimdi özelleştirdiğimiz `values.yaml` dosyasıyla beraber `loki-stack` chart’ını kurabiliriz:

```bash
helm install \
  --values values.yaml \
  loki \
  grafana/loki-stack
```

* `loki` burada “release name” olarak geçiyor. İstediğiniz bir isim verebilirsiniz.
* `grafana/loki-stack`: Kullanmak istediğimiz chart.

Kurulum tamamlandığında `Helm` bize, `loki-stack`’in başarılı şekilde kurulduğunu ve Grafana üzerinde otomatik olarak Loki veri kaynağının oluşturulduğunu belirten bir mesaj verir.

### 6. Kurulum Sonrası Kontrol (kubectl get all)



<figure><img src="../.gitbook/assets/Screenshot 2025-03-13 at 16.59.44.png" alt=""><figcaption></figcaption></figure>

`kubectl get all` komutu ile kaynakları görüntülediğinizde şunları görürsünüz:

1. **Pod’lar**:
   * `loki-0` adlı bir **StatefulSet** pod’u (Loki’nin kendisi).
   * `loki-grafana-xxxx` adlı bir **Deployment** pod’u (Grafana).
   * `loki-promtail-xxxxx` adlı bir **DaemonSet** pod’u (Promtail, her node’da bir tane).
2. **Services**:
   * `service/loki` (Loki’yi erişilebilir kılar).
   * `service/loki-grafana` (Grafana için Service).
   * `service/loki-headless`, `service/loki-memberlist` vb. (Loki’nin dahili iletişimi).
3. **StatefulSet / Deployment / ReplicaSet / DaemonSet**:
   * Grafana (Deployment)
   * Loki (StatefulSet)
   * Promtail (DaemonSet)

Bu düzen sayesinde, Promtail tüm node’lardaki log’ları okur ve Loki’ye iletir. Grafana arayüzünde (varsayılan 80 veya 3000 portuyla) Loki datasource’u otomatik tanımlanmış şekilde görürsünüz.

### 7. Özet

* **Helm repo** ekledik ve güncelledik.
* **Loki-stack** chart’ını aradık ve `values.yaml`’ı inceleyip özelleştirdik (Grafana’yı etkinleştirdik, sürüm tag’lerini güncelledik).
* **Helm install** komutuyla tek seferde Loki, Grafana ve Promtail’i Kubernetes üzerinde çalışır hale getirdik.
* **kubectl get all** komutuyla, StatefulSet (Loki), Deployment (Grafana) ve DaemonSet (Promtail) kaynaklarının başarılı şekilde oluşturulduğunu doğruladık.



