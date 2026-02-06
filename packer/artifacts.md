---
icon: newspaper
---

# Artifacts

#### 1. Artifact Nedir?&#x20;

Packer fabrikasını kurdun (Builder), içini döşedin (Provisioner), paketledin (Post-Processor). Banttan düşen o son ürüne Artifact denir.

* Önemli Ayrım: Artifact her zaman elle tutulur bir dosya olmak zorunda değildir.
  * VMware/VirtualBox kullanıyorsan: Artifact, bilgisayarında oluşan `.ova` veya `.vmdk` dosyasıdır (Klasör dolusu dosya).
  * AWS/Google Cloud kullanıyorsan: Artifact, sadece bir Kimlik Numarasıdır (ID). (Örn: `ami-05x...`). Çünkü "Image" aslında Amazon'un veri merkezinde durur, sana sadece onun referans numarasını verir.

#### 2. Değişmez Altyapı (Immutable Infrastructure) ve "Save Game" Mantığı

* Artifact, bir sunucunun en mükemmel, en ayarlı halinin fotoğrafını çekmek gibidir.
* Rollback (Geri Alma): Diyelim ki bugün `v2` versiyonlu bir Artifact (Image) ürettin ve canlı sisteme aldın. Ama sistem patladı. Hiç panik yapmazsın. Hemen `v1` versiyonlu Artifact'i devreye alırsın.
  * Sunucuyu tamir etmeye çalışmazsın, bozuk olanı atıp çalışan yedeği (Artifact'i) koyarsın.

#### 3. Görev Dağılımı: Packer vs. Terraform

* Packer (Mimar): Planı çizer, evi (Image/Artifact) oluşturur ve anahtarı (AMI ID) masaya bırakır. "Benim işim bitti, bu ID'yi kullanın" der. Packer sunucuyu canlıya alıp trafiği yönlendirmez.
* Terraform (İşletmeci): Masadaki anahtarı (Artifact ID'sini) alır. "Tamam, şimdi bu ID'yi kullanarak bana 50 tane sunucu aç" der ve sistemi canlıya alır.

#### 4. İş Akışındaki Yeri

Önceki parçalarla birleştirirsek tam resim şöyledir:

1. Builder: Boş makineyi açtı.
2. Provisioner: İçine yazılımları kurdu.
3. Artifact (ANLIK DURUM): Makine donduruldu ve bir çıktı (ID veya Dosya) oluştu.
4. Post-Processor: Bu çıktı (Artifact) sıkıştırıldı veya kaydedildi.

***

Özetle: Artifact, Packer işleminin sonucunda elde ettiğin \*\*"Altın Kopya"\*\*dır (Golden Image). İster bir dosya olsun, ister bir ami id numarası; gelecekte kuracağın tüm sunucuların atası ve şablonu budur.
