---
icon: servicestack
---

# Service Discovery & DNS

#### Neden Service Discovery'ye İhtiyacımız Var?

Kubernetes ortamında çalışan uygulamalarımız (_Pod_'lar) sürekli olarak yaratılıp silinebilir ve her seferinde yeni bir IP adresi alırlar. Eğer bir uygulamamızın (örneğin _Frontend_), diğer bir uygulamaya (örneğin _Backend_) bağlanması için kodunun içine sabit bir IP adresi veya hostname yazarsak (hardcoding yaparsak), sistem ilk Pod değişikliğinde çöker.

Amacımız şudur: IP adresleri sürekli değişse bile, uygulamalarımız birbirleriyle iletişim kurarken kodlarında hiçbir değişiklik yapmadan hedefi otomatik olarak bulabilsinler. İşte bu "bulma" işlemine Service Discovery diyoruz.

#### Arka Planda Neler Oluyor? (EndpointSlices)

Peki Kubernetes bu değişen IP'leri nasıl takip ediyor?

* Kubernetes, Service Discovery işlemini EndpointSlices adı verilen yapılarla tamamen otomatik hale getirir.
* Bir _Service_'e bağlı olan Pod'lar eklendiğinde, silindiğinde veya durumları değiştiğinde, _EndpointSlices_ anında kendini günceller.
* Normalde, içeride çalışan bir uygulamamızın gidip doğrudan Kubernetes API'sine "Bana şu servisin güncel IP'lerini ver" diye sorgu atması çok karmaşık ve zahmetli bir iştir.

İşte bu API karmaşıklığıyla uğraşmayalım diye Kubernetes, uygulamalarımızın Service'leri bulabilmesi için bize iki temel yöntem sunar: Environment Variables ve DNS.

***

#### Yöntem 1: Environment Variables

Bu yöntem, Kubernetes'in temel ve ilk mekanizmalarından biridir.

Nasıl Çalışır?

Sistemde `my-app` adında bir _Service_ olduğunu varsayalım. Yeni bir Pod ayağa kalktığı anda, Kubernetes bu Pod'un içine o anki _Service_ bilgilerini otomatik olarak birer environment variable olarak ekler (inject eder).

Örneğin, _Service_ IP'si `10.0.0.11` ve portu `80` ise, Pod'un içinde şu değişkenler oluşur:

```bash
MY_APP_SERVICE_HOST=10.0.0.11
MY_APP_SERVICE_PORT=80
```

Uygulamanın kodu sadece sistemindeki `MY_APP_SERVICE_HOST` değişkenini okuyarak nereye gideceğini bulur.

* Önemli Dezavantajı: Bu yöntemin çalışması için _Service_'in, Pod'dan önce yaratılmış olması zorunludur. Aksi halde Pod, var olmayan bir servisin değişkenlerini alamaz. Bu katılık sebebiyle yerini çoğunlukla DNS yöntemine bırakmıştır.

***

#### Yöntem 2: DNS (En Popüler ve En Basit Yöntem)

Günümüzde standart olarak kullanılan, en esnek ve basit Service Discovery yöntemidir. Aynı zamanda tüm namespace'ler arasında _Service_'lere zahmetsizce erişim imkanı sağlar.

CoreDNS Nasıl Çalışır?

Kubernetes _Cluster_'ının içinde CoreDNS adında bir iç DNS sunucusu sürekli çalışır ve Kubernetes API'sini dinler. Sisteme yeni bir _Service_ eklendiğinde, CoreDNS bunu anında fark eder ve kendi rehberine kaydeder.

Sen `my-app` adında bir _Service_ oluşturduğunda, DNS üzerinde otomatik olarak iki temel kayıt (DNS Records) oluşturulur:

1.  A/AAAA Kaydı: _Service_'in IP adresini verir. Formatı şöyledir:

    `my-app.default.svc.cluster.local`
2.  SRV Kaydı: İlgili port ve protokolü spesifik olarak bulmaya yarar. Formatı şöyledir:

    `_my-app._tcp.my-app.default.svc.cluster.local`

**Namespace İçi ve Dışı İletişim**

DNS sayesinde uygulamalar birbiriyle konuşurken çok kısa isimler kullanabilirler:

* Aynı Namespace İçindeyse: İstek yapan uygulaman ve bağlanacağı _Service_ aynı _namespace_ içindeyse, hedef adresi olarak koduna sadece `my-app` yazman yeterlidir.
* Farklı Namespace'teyse: Eğer farklı bir alandan, örneğin `default` isimli namespace'teki bir servise ulaşmak istersen, ismin yanına _namespace_ adını da ekleyip `my-app.default` şeklinde istek yaparsın.

**Peki "Sadece İsim Yazma" Nasıl Çalışıyor? (`resolv.conf`)**

Biz sadece `my-app` yazdığımızda Kubernetes'in o uzun adresi (`my-app.default.svc.cluster.local`) nasıl bulduğu çok kritik bir detaydır.

Her Pod'un içinde, DNS ayarlarını tutan `resolv.conf` adında bir dosya bulunur. Bu dosyanın içinde iki temel ayar vardır:

```bash
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
```

1. `nameserver`: Pod'a, "Bir adresi çözümlemen gerekirse bu IP'ye (CoreDNS sunucusuna) git" der.
2. `search`: İşte işin sırrı buradadır. Sen uygulamanda hedef olarak sadece `my-app` yazdığında, arka plandaki sistem bu `search` satırına bakar. İsmin sonuna sırasıyla buradaki uzantıları (`.default.svc.cluster.local` vb.) otomatik olarak ekler ve DNS'e öyle sorar.

Bu sayede geliştiriciler olarak biz, her seferinde upuzun adresler yazmak zorunda kalmayız; sistem bizim için adresi tamamlar ve doğru IP'yi bulup iletişimi kurar.
