---
icon: invision
---

# Giriş : Container Network Interface

#### CNI (Container Network Interface) Nedir?

Basitçe düşünelim: Bir Node üzerinde bir Pod oluşturduğunda, bu Pod'un dış dünyayla veya diğer Pod'larla konuşabilmesi için bir IP adresine ve bir ağ kablosuna ihtiyacı vardır.

Eskiden her Container teknolojisi (Docker, rkt vb.) bunu kendi kafasına göre yapardı. Bu kaos yarattı. CNCF (Cloud Native Computing Foundation) dedi ki: _"Arkadaşlar, bir standart belirleyelim. Kimse tekerleği yeniden icat etmesin. Biz bir interface yazalım, plugin geliştiricileri bu standarta uysun."_

İşte CNI, bu standarttır. Container Runtime ile Network Plugin arasındaki "ortak dil"dir.

***

#### Nasıl Çalışır?

Sen Kubernetes'e "Bana bir Pod yarat" dediğinde arka planda şunlar olur:

1. Pod Oluşumu: Kubernetes (Kubelet), Container Runtime'a (örneğin `containerd` veya `CRI-O`) emri verir.
2. Namespace Hazırlığı: Runtime, Pod için izole bir alan (Network Namespace) oluşturur. Ama şu an bu alanın içinde ne bir IP var ne de bir kablo.
3. CNI Çağrısı: Runtime, hemen diskteki CNI konfigürasyonuna bakar (genelde `/etc/cni/net.d/` altındadır). Sonra CNI Plugin'ini (bu bir binary dosyadır, `/opt/cni/bin/` altındadır) çağırır.
4. İşlem: Runtime, CNI Plugin'e der ki: _"Hey, bu Namespace'e bir IP ver ve onu ağa bağla (ADD komutu)."_
5. Sonuç: Plugin işini yapar, Pod'a IP'yi atar ve sonucu JSON formatında Runtime'a geri döner (stdout üzerinden).

Artık Pod'un interneti var!

***

#### Temel CNI Operasyonları

CNI Plugin'leri aslında çok basit komutlarla çalışır. Runtime, Plugin'i çağırırken Environment Variables kullanır.

1. ADD:&#x20;
   * Pod oluşturulurken çalışır. Interface'i oluşturur, IP atar.
2. DEL:
   * Pod silinirken çalışır. Kaynakları temizler, IP'yi havuza geri iade eder.
3. CHECK:
   * Ağ durumunun sağlıklı olup olmadığını kontrol eder.
4. VERSION:&#x20;
   * Plugin'in desteklediği CNI versiyonunu sorar.

***

#### Örnek: Bir CNI Konfigürasyonu

Bir sistem yöneticisi olarak `/etc/cni/net.d/` klasörüne girdiğinde şöyle bir dosya görebilirsin. Bu dosya Runtime'a ne yapması gerektiğini söyler.

```json
{
  "cniVersion": "1.0.0",
  "name": "my-k8s-net",
  "type": "bridge",         // Hangi binary kullanılacak? (Burada 'bridge' plugin'i)
  "bridge": "cni0",         // Bridge ismi
  "isGateway": true,
  "ipMasq": true,
  "ipam": {                 // IP Address Management (IP'yi kim dağıtacak?)
    "type": "host-local",
    "subnet": "10.244.0.0/16",
    "routes": [
      { "dst": "0.0.0.0/0" }
    ]
  }
}
```

_Burada `type: bridge` dediğimiz için Runtime, `/opt/cni/bin/bridge` binary dosyasını çalıştıracaktır._

***

Bazen tek bir işlem yetmez. Tıpkı bir fabrikadaki üretim bandı gibi düşün.

* Senaryo: Pod'a önce bir IP verilsin, sonra port yönlendirmesi yapılsın, en son da bant genişliği sınırlansın.

CNI, bu işlemleri sıraya koyar (Chaining).

* ADD işlemi: Liste sırasıyla çalışır (1 -> 2 -> 3).
* DEL işlemi: Tam tersi sırayla çalışır (3 -> 2 -> 1). Böylece temizlik düzgün yapılır.
* Eğer 3. adımda hata olursa, sistem otomatik olarak geriye dönük (2 ve 1 için) DEL komutunu çalıştırır.

***

#### Popüler CNI Plugin'leri (Hangisini Ne Zaman Seçmelisin?)

Kaynaklarında geçen eklentileri bir SysAdmin gözüyle yorumlayalım:

1. Flannel:
   * _Özellik:_ Çok basittir. Sadece Pod'lara IP verir (L3). Karmaşık Network Policy (güvenlik kuralı) desteklemez.
   * _Kullanım:_ Küçük Cluster'lar veya öğrenme ortamları (k3s, minikube) için ideal.
2. Calico:
   * _Özellik:_ Hem ağ sağlar hem de gelişmiş Network Policy özelliklerine sahiptir. BGP protokolünü kullanır.
   * _Kullanım:_ Production ortamlarının %80'inde bu kullanılır. Standarttır.
3. Weave Net:
   * _Özellik:_ Kurulumu çok kolaydır ve encryption özelliği yerleşiktir.
   * _Kullanım:_ Karmaşık ayar yapmadan şifreli ağ isteyenler için.
4. Cilium:
   * _Özellik:_ eBPF teknolojisini kullanır (Linux çekirdeği seviyesinde çalışır). İnanılmaz hızlıdır ve çok detaylı observability sağlar.
   * _Kullanım:_ Yüksek performans, büyük ölçekli sistemler ve modern güvenlik ihtiyaçları için.

