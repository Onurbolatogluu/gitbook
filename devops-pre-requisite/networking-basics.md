# 💛 Networking Basics

### Switch ve Ağ İletişimi:

<figure><img src="../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

Bir switch, ağdaki cihazların (bilgisayarlar, sunucular vb.) birbirleriyle iletişim kurmasına olanak sağlayan bir ağ cihazıdır. Switch, gelen veri paketlerini doğru hedefe yönlendirir, bu sayede cihazlar arasında iletişim sağlanır.

* **Cihaz A**:
  * IP adresi: 192.168.1.10
  * Ağ arayüzü: eth0
* **Cihaz B**:
  * IP adresi: 192.168.1.11
  * Ağ arayüzü: eth0
* **Switch**:
  * Her iki cihazı (A ve B) birbirine bağlayan bir ağ cihazı.

**Ağ Arayüzünün durumunu kontrol etmek,**

```bash
ip link
# Açıklama: Bu komut, ağ arayüzlerinin durumunu gösterir.
# Çıktı Örneği:
# eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
# Bu çıktı, eth0 arayüzünün aktif (UP) olduğunu gösterir.
```

**IP Adresi Ekleme (Cihaz A)**

```bash
ip addr add 192.168.1.10/24 dev eth0
# Açıklama: Bu komut, eth0 arayüzüne 192.168.1.10 IP adresini ekler.
```

**IP Adresi Ekleme (Cihaz B)**

```bash
ip addr add 192.168.1.11/24 dev eth0
# Açıklama: Bu komut, eth0 arayüzüne 192.168.1.11 IP adresini ekler.
```

**İki Cihaz Arasında Ping Testi**

```bash
ping 192.168.1.11
# Açıklama: Cihaz A'dan Cihaz B'ye ping atarak bağlantının aktif olup olmadığını test eder.
# Çıktı Örneği:
# Reply from 192.168.1.11: bytes=32 time=4ms TTL=117
# Bu çıktı, Cihaz A'dan Cihaz B'ye veri paketlerinin başarıyla ulaştığını gösterir.
```

**Özetle,**

1. `ip link` komutları ile her iki cihazda ağ arayüzlerinin aktif olduğunu kontrol ettik.
2. `ip addr add` komutları ile her iki cihaza IP adresleri atadık.
3. `ping` komutu ile Cihaz A'dan Cihaz B'ye veri göndererek bağlantıyı test ettik.



### Router Nedir?

Router, iki veya daha fazla farklı ağı (subnet) birbirine bağlayan bir ağ cihazıdır. Router, paketleri bir ağdan diğerine yönlendirir ve bu sayede ağlar arasında veri iletişimi sağlar.

<figure><img src="../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

**Ağ Yapısı:**

* **Ağ A (Subnet 1)**:
  * Cihaz A: IP 192.168.1.10
  * Cihaz B: IP 192.168.1.11
  * Router'ın Ağ A tarafındaki IP'si: 192.168.1.1
* **Ağ B (Subnet 2)**:
  * Cihaz C: IP 192.168.2.10
  * Cihaz D: IP 192.168.2.11
  * Router'ın Ağ B tarafındaki IP'si: 192.168.2.1



* Router, IP adreslerini kullanarak paketlerin hedefe ulaşmasını sağlar. IP paketlerinin hedef adresine bakarak uygun ağ arayüzüne yönlendirir.
* Farklı subnet'ler arasındaki bağlantıyı sağlar ve bu ağlar arasında veri iletimine olanak tanır.
* Router, kendisine gelen veri paketlerini hedef ağın IP adresine göre ilgili ağ arayüzüne iletir.



**Router'ın IP Adreslerini Atama:**

Ağ A tarafı (192.168.1.1)

```bash
ip addr add 192.168.1.1/24 dev eth0
```

Ağ B tarafı (192.168.2.1)

```bash
ip addr add 192.168.2.1/24 dev eth1
```



**Yönlendirme Tablolarını Yapılandırma:**

Cihaz A ve Cihaz B için Router'a Yönlendirme Ekleme

```bash
ip route add 192.168.2.0/24 via 192.168.1.1
```

Cihaz C ve Cihaz D için Router'a Yönlendirme Ekleme

```bash
ip route add 192.168.1.0/24 via 192.168.2.1
```

Bu yapılandırma ile, Ağ A'daki cihazlar (192.168.1.0/24) ve Ağ B'deki cihazlar (192.168.2.0/24) bir router aracılığıyla birbirleriyle iletişim kurabilirler. Router, bu iki ağ arasında veri paketlerini yönlendirir ve doğru hedefe ulaşmalarını sağlar. Bu sayede, iki farklı subnet'teki cihazlar arasında veri alışverişi yapılabilir.



### Gateway Nedir?

**Gateway**, bir ağdan, diğerine veri paketlerinin geçişini sağlayan ağ geçididir. İki farklı ağ arasındaki veri trafiğini yönetir.

### Default Gateway Nedir?

**Default Gateway**, bir cihazın hedefe ulaşamayan veri paketlerini yönlendirdiği varsayılan ağ geçididir. Başka bir deyişle, bir cihaz başka bir ağa veri göndermek istediğinde, IP yönlendirme tablosunda bir yol bulamazsa, veri paketlerini default gateway'e gönderir.&#x20;



**Cihaz A'da Default Gateway Ayarlama:**

```bash
ip route add default via 192.168.1.1
# Açıklama: Cihaz A için default gateway'i 192.168.1.1 olarak ayarlar.
```

**Cihaz B'de Default Gateway Ayarlama:**

```bash
ip route add default via 192.168.2.1
# Açıklama: Cihaz B için default gateway'i 192.168.2.1 olarak ayarlar.
```

* Cihaz A, 192.168.1.0/24 ağında olmayan bir hedefe veri göndermek istediğinde (örneğin, 192.168.2.10), bu veri paketi default gateway (192.168.1.1) üzerinden yönlendirilir.
* Router (192.168.1.1), bu paketi alır ve uygun arayüze (192.168.2.1) yönlendirir.
* Paket, sonunda 192.168.2.10 adresine ulaşır.



#### Routing Examples,

<figure><img src="../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

**Durum,**

* **Cihaz A**: IP 192.168.1.5
* **Cihaz B**:
  * **eth0**: 192.168.1.6
  * **eth1**: 192.168.2.6
* **Cihaz C**: IP 192.168.2.5

Cihaz A ve C farklı subnet'lerde bulunuyor ve Cihaz B bu iki subnet arasında bir router görevi görüyor.



**Sorun:**

* **Cihaz A'dan Cihaz C'ye Ping**: İlk denemede `Network is unreachable` hatası alınıyor.

```bash
ping 192.168.2.5
# Çıktı: Connect: Network is unreachable
```



**Çözüm:**

* Cihaz A'ya 192.168.2.0/24 ağına nasıl ulaşacağını söyleyen bir route ekliyoruz.

```bash
ip route add 192.168.2.0/24 via 192.168.1.6
```

Cihaz B'nin iki farklı subnet'i (192.168.1.0/24 ve 192.168.2.0/24) birbirine bağlaması ve paket yönlendirmesi için IP forwarding'in etkinleştirilmesi gerekmektedir. IP forwarding, bir cihazın iki farklı ağ arasındaki trafiği yönlendirmesini sağlar. Bu özellik etkinleştirildiğinde, cihaz bir router gibi davranır ve gelen paketleri uygun ağa iletir.

**Cihaz B'de, IP Forwarding Durumunu Kontrol Etme,**

```bash
cat /proc/sys/net/ipv4/ip_forward
# Çıktı: 0 (IP forwarding kapalı)
```

**Cihaz B'de, IP Forwarding Etkinleştirme,**

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
# IP forwarding etkinleştirilir
```

**Cihaz B'de, Kalıcı Olarak IP Forwarding Ayarlama,**

`/etc/sysctl.conf` dosyasına aşağıdaki satırı ekleyerek IP forwarding'in kalıcı olmasını sağlayabilirsiniz.

```bash
net.ipv4.ip_forward = 1
```

**Cihaz A'dan Cihaz C'ye ping atarak bağlantının başarılı olduğunu kontrol edebiliriz.**

```bash
ping 192.168.2.5
# Çıktı: Reply from 192.168.2.5: bytes=32 time=4ms TTL=117
```



***

Basic Networking Cheat Sheets,

```bash
# Ağ arayüzlerinin durumunu gösterir
ip link
# Çıktı: 
# 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
#    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
#    link/ether 00:15:5d:22:25:84 brd ff:ff:ff:ff:ff:ff

# Ağ arayüzlerine atanmış IP adreslerini gösterir
ip addr
# Çıktı:
# 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
#    inet 127.0.0.1/8 scope host lo
#       valid_lft forever preferred_lft forever
#    inet6 ::1/128 scope host
#       valid_lft forever preferred_lft forever
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
#    inet 192.168.1.10/24 brd 192.168.1.255 scope global dynamic eth0
#       valid_lft 172737sec preferred_lft 172737sec
#    inet6 fe80::215:5dff:fe22:2584/64 scope link
#       valid_lft forever preferred_lft forever

# eth0 arayüzüne 192.168.1.10/24 IP adresini ekler
ip addr add 192.168.1.10/24 dev eth0

# Yönlendirme tablosunu görüntüler
ip route
# Çıktı:
# default via 192.168.1.1 dev eth0
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10

# 192.168.1.0/24 ağına erişimi 192.168.2.1 IP adresi üzerinden sağlar
ip route add 192.168.1.0/24 via 192.168.2.1

# IP forwarding'in etkin olup olmadığını kontrol eder
cat /proc/sys/net/ipv4/ip_forward
# Çıktı: 
# 0 (IP forwarding kapalı)

# IP forwarding'i etkinleştirir
echo 1 > /proc/sys/net/ipv4/ip_forward

# IP forwarding'in etkinleştirildiğini kontrol eder
cat /proc/sys/net/ipv4/ip_forward
# Çıktı:
# 1 (IP forwarding etkin)

```

