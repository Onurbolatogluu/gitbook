---
icon: align-justify
---

# Araçlar ve Komutlar

**1. Yönetim Katmanı: Kubernetes Yerel Araçları (kubectl)**

Bu, kuş bakışı (high-level) inceleme katmanıdır. Kubernetes API'sine "Durum nedir?" diye sorarız.

* `kubectl describe pod`: Pod'un hangi Node üzerinde çalıştığını ve hangi IP adresini (Pod IP) aldığını gösterir. "IP-per-Pod" kuralının ilk kontrol noktasıdır.
* `kubectl get pods -o wide`: Kümedeki tüm Pod'ların IP'lerini ve Node dağılımını toplu halde görmek için kullanılır. İletişim testleri için hedef IP'leri buradan öğreniriz.
* `kubectl exec`: Bu komut, ağın çalışıp çalışmadığını içeriden test etmek için en kritik araçtır.
  * _Pod İçi Test:_ `ip addr` ile Pod'un kendi IP'sini ve `lo` (loopback) arayüzünü görüp görmediği kontrol edilir.
  * _Bağlantı Testi:_ `curl <hedef-ip>` komutuyla, ağın trafiğe izin verip vermediği (Connectivity) doğrulanır.

**2. İzolasyon Katmanı: Namespace Analizi (`lsns`, `ip netns`)**

* `lsns -t pid`:  Node üzerindeki tüm süreçlerin (Process) namespace'lerini listeler. Burada, her Pod için bir \*\*"Pause Container"\*\*ın ağı ayakta tuttuğunu görebilirsiniz.
* `ip netns identify <PID>`: Bir process'in hangi network namespace'ine ait olduğunu söyler. (örneğin `cni-1234abcd...`), ağın CNI Plugin tarafından yönetildiğinin kanıtıdır.
* `ip netns exec`: Konteynerin içine girmeden (SSH veya `kubectl exec` yapmadan), doğrudan Node üzerinden o ağ alanına girip komut çalıştırmayı sağlar. Bu, "Dışarıdan içeriye bakış" imkanı sunar.

**3. Fiziksel/Sanal Katman: Yapılandırma ve Test (`ip addr`, `curl`)**

Bu araçlar, sanal kabloları ve trafiğin akışını gösterir.

* `ip addr` (Host Düzeyinde): Node üzerinde çalıştırıldığında, CNI tarafından oluşturulan veth (Virtual Ethernet) çiftlerini listeler. Bu sanal kablolar, Pod'ları ana makineye bağlayan köprülerdir.
* `ip addr` (Pod Düzeyinde): Pod'un içindeki `eth0` arayüzünü ve aldığı IP'yi doğrular.
* `curl`:  "Herkes herkesle konuşabilir" kuralının sağlamasını yapar. Hem `localhost` (Pod içi) hem de uzak IP (Pod-to-Pod) erişimini test eder.
