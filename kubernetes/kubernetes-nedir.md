# 🍏 Kubernetes: Nedir ?

Kubernetes, hem beyan temelli yapılandırmayı, hem de otomasyonu kolaylaştıran, container iş yüklerini  ve hizmetlerini yönetmek için oluşturulmuş taşınabilir ve genişletilebilir açık kaynaklı bir platformdur.&#x20;

* Container iş yüklerini yönetmek için tercih edilen en önemli container orchestration platformudur.
* İsteyen herkes tarafından ücretsiz bir şekilde kurulur.
* Tamamen moduler yapıda tasarlanmıştır. istenirse bu modüler yapı genişletilebilir.
* Her kubernetes komponenti ayrı bir major modül olarak ayrılmıştır.
* Kubernetes'den istediklerimizi declarative olarak belirtmemiz yeterlidir. Şunu yap, sonra şunu yap olarak değil de, Şunu istiyorum gibi..
* Kubernetes genel olarak ondan istediklerimizi belirtmemizi sağlıyor.

<table><thead><tr><th width="417.20226843100187">Desired State ( Deklare edilen durum )</th><th>Current State ( Mevcut durum )</th></tr></thead><tbody><tr><td>abcnet/k8s:latest isimli imajı kullan.</td><td>Belirtilen özelliklerde 10 container çalışıyor.</td></tr><tr><td>Sistem genelinde 10 container çalışacak.</td><td></td></tr><tr><td>Eğer bu servisi güncellersem her güncellemede aynı anda 2 task üzerinde yürütülecek ve her güncelleme arasında 10sn bekleyecek.</td><td></td></tr><tr><td>Dış dünyaya 80 portundan yayınlanacak.</td><td></td></tr></tbody></table>

10 yerine 9 containera düşerse kubernetes bunu otomatik olarak algılayıp düzeltecek.
