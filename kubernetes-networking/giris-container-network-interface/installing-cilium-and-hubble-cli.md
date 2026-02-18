---
icon: clipboard
---

# Installing Cilium and Hubble CLI

Elinde hali hazırda kurulmuş, sunucuları (Node'ları) olan bir Kubernetes _cluster_'ı var(varsayıyorum). Ancak bu _cluster_ şu an "kör ve sağır" durumda. İçindeki Pod'lar birbirleriyle iletişim kuramıyor, IP adresleri yok. Çünkü Kubernetes'in içinde standart olarak bir ağ altyapısı (CNI) bulunmaz; bunu senin dışarıdan kurman gerekir.

İşte biz bu boş _cluster_'a, piyasadaki en gelişmiş ağ ve güvenlik aracı olan Cilium'u (ve onun izleme aracı olan Hubble'ı) kuracağız.

Bu işlem iki ana aşamadan oluşur:

***

#### AŞAMA 1: Kendi Bilgisayarına Kurmak

Uzaktaki Kubernetes _cluster_'ına Cilium'u yükleyebilmek ve sonrasında ağı yönetebilmek için, kendi bilgisayarına iki küçük terminal aracı (CLI) kurman gerekiyor. Bunlar senin "uzaktan kumandaların" olacak.

{% embed url="https://docs.cilium.io/en/latest/gettingstarted/k8s-install-default/#install-the-cilium-cli" %}

_(Not: (`curl`, `tar`, `sha256sum`) ile yüklenebiliyor olsa da, MacBook kullanıyorsak bunları `brew install cilium-cli hubble` gibi çok daha kısa yollarla kurabilirsin)_

1. Cilium CLI (`cilium`): Bu araç, uzaktaki _cluster_'a bağlanıp içine Cilium ağ altyapısını kurmanı, durumunu kontrol etmeni ve "Ağ düzgün çalışıyor mu?" diye test etmeni sağlar.
2. Hubble CLI (`hubble`): Bu araç ise, Cilium kurulduktan sonra ağın röntgenini çekmeni sağlar. "Hangi Pod hangi IP'ye gidiyor?", "Bağlantı nerede engellendi?" gibi soruların cevabını canlı olarak terminalinde görmeni sağlayan bir gözlem (observability) aracıdır.

Kendi bilgisayarına bu araçları indirdikten sonra, terminalinde çalıştıklarından emin olmak için versiyonlarını kontrol edebiliriz:

```bash
cilium version --client
hubble version
```

***

#### AŞAMA 2: Kubernetes Cluster'ının İçine Cilium'u Yüklemek

Artık bilgisayarındaki kumandalar hazır. Şimdi hedefimiz olan kurulu Kubernetes _cluster_'ına bağlanıp (bunun için `kubectl` yetkisine sahip olmalısın), boş duran ağ altyapısını Cilium ile dolduracağız.

**1. Cilium'u Enjekte Etmek (Kurulum)**

Kendi terminalinden şu komutu çalıştırırsın:

```bash
cilium install --version 1.15.4 --wait
```

* Ne Yapar? Senin bilgisayarındaki Cilium CLI aracı, uzaktaki Kubernetes _cluster_'ına bağlanır. Oradaki her bir sunucunun (Node'un) içine `cilium-agent` adında bir eBPF işçisi yerleştirir. `--wait` parametresi, bu işçilerin hepsi tam anlamıyla çalışana kadar senin terminalini bekletir.&#x20;

{% hint style="info" %}
Terminaline `cilium install` yazdığında, bu komut sihirli bir şekilde doğru cluster'ı bulmuyor. Senin bilgisayarında çalışan bu Cilium CLI aracı, nereye gideceğini tamamen `kubectl`'in ayarlarından (Kubeconfig dosyasından) öğreniyor.
{% endhint %}

* Sonuç: Bu komut bittiğinde _cluster_'ın artık kör ve sağır değildir. Pod'lar anında IP adresi alır ve birbirleriyle ışık hızında (eBPF sayesinde) konuşmaya başlarlar.
* Kontrol: Kurulumun sağlıklı olduğunu görmek için `cilium status` komutunu kullanırsın. (Her şeyin "OK" olması gerekir).

**2. Ağı Test Etmek (Her Şey Yolunda Mı?)**

Yeni kurduğun ağın gerçekten çalışıp çalışmadığını manuel olarak test etmek yerine, Cilium'un harika bir otomasyon aracı vardır:

```bash
cilium connectivity test
```

* Ne Yapar? _Cluster_'ın içine kendi kendine "Test-Pod"ları yaratır. Bu Pod'ları birbiriyle konuşturmaya çalışır, internete ping atar ve tüm ağ yollarının açık olduğunu sana raporlar.

**3. Hubble'ı Aktif Etmek**

Cilium ağı kurdu ama trafiği izleme özelliği (Hubble) sistemi yormasın diye varsayılan olarak kapalı gelir. Bunu açmak için Helm (Kubernetes'in paket yöneticisi) kullanırsın:

```bash
helm upgrade cilium cilium/cilium --version 1.15.4 \
  --namespace kube-system \
  --reuse-values \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true
```

* Ne Yapar? Mevcut çalışan Cilium kurulumunu bozmadan (`--reuse-values` kısmı çok önemlidir), sadece Hubble'ın terminal izleme (`relay`) ve tarayıcı arayüzü (`ui`) bileşenlerini aktif hale getirir.

**4. Canlı Trafiği (Matrix'i) İzlemek**

Artık ağında neler olup bittiğini kendi bilgisayarından canlı izleyebilirsin. Önce _cluster_ ile kendi bilgisayarın arasında güvenli bir tünel açman gerekir:

```bash
cilium hubble port-forward
```

Bu komut arka planda çalışırken, yeni bir terminal sekmesi açıp sihirli komutu yazarsın:

```bash
hubble observe
```

* Sonuç: Terminal ekranında, cluster içindeki tüm ağ trafiği canlı loglar halinde akmaya başlar. "A Pod'u B Pod'una bağlandı (TCP ACK)" gibi detayları anlık olarak görürsün.

Zaten kurulu ama ağ altyapısı olmayan bir Kubernetes _cluster_'ına kendi bilgisayarımızdaki CLI araçlarını kullanarak bağlandık. İçeriye Cilium'u kurduk, ağın çalıştığını test ettik ve son olarak Hubble ile içerideki trafiği canlı izlemeye başladık.

