---
icon: traffic-cone
---

# Traefik Overview

Kubernetes ortamında dışarıdan gelen trafiği içeriye dağıtmak için bir "Ingress Controller" kurmamız gerektiğini biliyoruz. Piyasada birçok seçenek var, ancak son yıllarda modern DevOps ekiplerinin mimarilerinde en çok parlayan isimlerden biri Traefik.

Traefik, Go diliyle yazılmış, son derece dinamik bir HTTP reverse proxy ve yük dengeleyicidir. Peki onu geleneksel proxy sunucularından ayıran ve Kubernetes için bu kadar mükemmel kılan şey nedir?

### "Hot Reload" ve Dinamik Keşif

Geleneksel proxy sunucularında (klasik Nginx veya HAProxy kurulumlarında) sisteme yeni bir web sitesi veya kural eklediğinizde, konfigürasyon dosyasını güncelleyip proxy servisine bir "Reload" komutu göndermeniz gerekir. Bu durum, saniyede binlerce isteğin aktığı devasa sistemlerde anlık kesintilere veya performans dalgalanmalarına yol açabilir.

Traefik ise Kubernetes'in API'sini 7/24 dinler. Siz sisteme yeni bir uygulama (Pod) eklediğinizde veya bir Ingress kuralı yazdığınızda, Traefik bunu anında fark eder ve hiçbir yeniden başlatma (restart) veya reload işlemine gerek duymadan trafiği saniyesinde yeni hedefe göndermeye başlar.&#x20;

***

### Traefik'in 4 Temel Yapı Taşı

Traefik'in mimarisini anlamak, onunla çalışmayı inanılmaz derecede kolaylaştırır. Sisteme giren bir ağ paketi, hedefine ulaşana kadar şu 4 aşamadan geçer:

#### 1. EntryPoints

Traefik'in dış dünyayı dinlediği fiziksel kapılardır. Genellikle iki tane tanımlanır: `web` (Port 80 - HTTP) ve `websecure` (Port 443 - HTTPS). Paket önce bu kapıdan içeri girer.

#### 2. Routers

İçeri giren paketin kimliğinin sorulduğu yerdir. Router, gelen isteğin adresine (Örn: `api.ornek.com`) veya yoluna (Örn: `/blog`) bakar. Eğer kurallarla eşleşiyorsa, paketi içeri alır.

#### 3. Middleware

İşte Traefik'i rakiplerinden ayıran en harika özellik burasıdır. Paket hedefe gitmeden önce Middleware'e sokulabilir. Burada isteğin üzerinde oynamalar yapılır. Örneğin:

* Rate Limiting: Bir kullanıcı saniyede 10'dan fazla istek atıyorsa burada engellenir.
* Auth: Uygulamanıza şifre ekranı koymayı unuttunuz mu? Sorun değil, Middleware ile ana kapıda Basic-Auth veya Forward-Auth (Google ile giriş vb.) zorunluluğu getirebilirsiniz.
* Redirects: HTTP ile gelenleri otomatik olarak HTTPS'e zorlama işi burada yapılır.

#### 4. Services

Paket tüm güvenlik ve kurallardan geçtikten sonra, nihai olarak teslim edileceği yerdir (Yani sizin arkada çalışan Kubernetes Pod'larınız).

***

### Neden Traefik Kullanmalısınız?

1\. Otomatik SSL/HTTPS (Let's Encrypt Entegrasyonu):

Normalde SSL sertifikası almak ve süresi dolmadan yenilemek ciddi bir operasyonel yüktür. Traefik'e sadece e-posta adresinizi verirsiniz; o Let's Encrypt ile otomatik konuşur, domainleriniz için ücretsiz SSL sertifikalarını alır, kurar ve süresi dolmadan kendi kendine yeniler.

2\. Gözlem ve Yönetim Paneli (Dashboard):

Traefik, sistemin anlık durumunu görebileceğiniz çok şık bir web arayüzü ile birlikte gelir. Hangi EntryPoint ne kadar trafik alıyor, hangi Router başarılı çalışıyor, backend servislerinin sağlığı ne durumda? Hepsini terminale girmeden canlı olarak izleyebilirsiniz.

3\. Çoklu Protokol Desteği:

Sadece HTTP/HTTPS web trafiği değil; TCP ve UDP üzerinden çalışan veritabanları veya WebSocket tabanlı anlık mesajlaşma uygulamalarınızın trafiğini de mükemmel şekilde yönlendirir.

### Özet

Eğer sisteminiz sürekli büyüyüp küçülüyorsa (autoscaling), sürekli yeni mikroservisler eklenip çıkarılıyorsa ve SSL sertifikaları gibi operasyonel işlerle zaman kaybetmek istemiyorsanız; Traefik sizin için tasarlanmış kusursuz bir modern ağ yöneticisidir.

