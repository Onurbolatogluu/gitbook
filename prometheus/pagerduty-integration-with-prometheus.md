# 🤯 PagerDuty integration with Prometheus

1. Öncelikle [_www.pagerduty.com_](https://www.pagerduty.com/) _adresine gidip, bir hesap oluşturuyoruz._

<figure><img src="../.gitbook/assets/Ekran görüntüsü 2023-05-22 143120.png" alt=""><figcaption></figcaption></figure>

2. Ardından service oluşturma ekranına gelip, "New Service" butonuna tıklıyoruz.

<figure><img src="../.gitbook/assets/image (155).png" alt=""><figcaption></figcaption></figure>

3. Name kısmına bir isim verip, "integrations" kısmına kadar ilerliyoruz.

<figure><img src="../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

4. integrations kısmında "prometheus" seçip, "create service" butonuna tıklıyoruz.

Böylelikle servisimiz oluşturuldu. En son gelen ekranda lazım olacak 2 kısım mevcut,

1. Integration Key
2. Integration URL

Bu bilgileri bir yere kaydediyoruz. Ardından alertmanager yapılandırma dosyamıza giriyoruz.

```yaml
global:
  smtp_from: 'alert@example.com'
  smtp_smarthost: mail.example.com:587
  smtp_auth_username: 'alert@example.com'
  smtp_auth_identity: 'alert@example.com'
  smtp_auth_password: 'JhwhWAbVcWG3b734Zjq6U'
  resolve_timeout: 1m
  pagerduty_url: 'https://events.eu.pagerduty.com/generic/2010-04-15/create_event.json'
```

Service oluştururken aldığımız Integration URL bilgisini, alertmanager yapılandırma dosyası içerisinde bulunan global argümanının altına "pagerduty\_url": parametresine yapıştırıyoruz.

```yaml
- name: linux-team-manager
  pagerduty_configs:
  - service_key: 'fe5512dsadasd220bfbsss32236d'
    send_resolved: true
```

Ardından, alarmlarını pagerduty'e göndereceğimiz grubu yukarıdaki şekilde tanımlıyoruz. "service\_key" kısmını, yukarıda kopyaladığımız Integration Key bilgisinden yapıştırıyoruz.



<figure><img src="../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

Görüldüğü üzere, alarmlar pagerduty hesabımıza ulaşıyor.

