# ⏰ Slack integration with Prometheus

Slack ile Prometheus entegrasyonu yapmak için aşağıdaki adımları takip etmeniz gerekmektedir:\
\


<figure><img src="../.gitbook/assets/image (148).png" alt=""><figcaption></figcaption></figure>

1. Slack hesabınıza girip, bir oda oluşturmalısınız. Bizim örneğimizde bu odanın adı '#prometheus' olarak belirledik.

<figure><img src="../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

2. Add apps kısmından "incoming Webhooks" uygulamasını slack 'e yükleyip, az önce oluşturduğumuz odamıza atamamız gerekmektedir.  Ve alt kısımda bulunan webhook url kısmını kopyalamamız gerekmektedir, bunu alertmanager config de kullanacağız.

<figure><img src="../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

```yaml
- name: windows-team-manager
  slack_configs:
  - channel: '#prometheus'
    api_url: 'https://hooks.slack.com/services/T0234479W8JXT7/B01258SSSSA4563YSCAZ/BhgtsdasadDSDSasdERYHklbqvhh9'
    send_resolved: true
```

3. Slack üzerinden alarm göndereceğimiz grubun altına slack "webhook url" idmizi yapıştırıyoruz ardından alertmanager servisini restart edip, işlemleri tamamlıyoruz.

<figure><img src="../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>
