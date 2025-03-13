---
icon: wifi-fair
---

# Connecting to Grafana

Kubernetes üzerinde Helm ile kurduğumuz Grafana arayüzüne nasıl bağlanacağımızı anlatacağım.&#x20;

### 1. Port Forward ile Grafana’ya Erişim

1. **Grafana Pod’unu Bulun**
   * `kubectl get pods` komutuyla “`loki-grafana-xxx`” adında bir pod olduğundan emin olun.
2. **Port Forward Başlatma**
   *   Aşağıdaki komutla, pod’daki **3000** portunu yerel makinenizin **3000** portuna yönlendirirsiniz:

       ```bash
       kubectl port-forward pod/loki-grafana-<pod-ismi> 3000:3000 --address="0.0.0.0"
       ```
   * Bu sayede tarayıcınızda `http://localhost:3000` (veya `http://<sunucu_ip_adresi>:3000`) üzerinden Grafana arayüzüne ulaşabilirsiniz.

### 2. Varsayılan Admin Parolasını Öğrenme

1. **Grafana Secret’larını Görüntüleme**
   *   Helm, Grafana’yı kurarken “`loki-grafana`” isimli bir “secret” kaynağı oluşturur. Bunu aşağıdaki komutla görebilirsiniz:

       ```bash
       kubectl get secret
       ```
   * Listede “`loki-grafana`” secret’ını görün.
2. **Admin Şifresini Almak**
   *   Aşağıdaki komutla secret içindeki `admin-password` alanını Base64 çözerek alırsınız:

       ```bash
       kubectl get secret loki-grafana \
         -o jsonpath="{.data.admin-password}" | base64 --decode
       ```
   * Çıktı olarak bir metin elde edersiniz; bu, **admin** kullanıcısı için geçici (varsayılan) paroladır.

### 3. Grafana Arayüzüne Giriş

1. **Tarayıcıyı Açın**
   * `http://localhost:3000` veya port-forward yaptığınız sunucunun IP adresine girin.
2. **Kullanıcı Adı ve Parola**
   * Kullanıcı adı: `admin`
   * Parola: Bir önceki adımda Base64 çözerek elde ettiğiniz metin.
3. **Giriş Yaptıktan Sonra**
   * Dilerseniz yeni bir parola belirlemeniz istenir veya “Skip” (Atla) diyerek varsayılanı koruyabilirsiniz.

### 4. Otomatik Olarak Eklenen Loki Datasource

<figure><img src="../.gitbook/assets/Screenshot 2025-03-14 at 00.10.47.png" alt=""><figcaption></figcaption></figure>

Grafana arayüzünde sol menüden **Connections > Data sources** (veya “Configuration > Data sources”) bölümüne gittiğinizde **Loki** veri kaynağının (Data Source) **otomatik** eklenmiş olduğunu görürsünüz. Helm chart bu ayarı kurulum sırasında sizin yerinize yaptığı için ek bir yapılandırmaya ihtiyaç kalmaz.



