---
icon: f
---

# packer fmt

HashiCorp Configuration Language (HCL2), belirli bir sözdizimi ve stil standardına sahiptir. `packer fmt` komutu, mevcut yapılandırma dosyalarını tarayarak bu dosyaları HashiCorp'un resmi stil kılavuzuna (Canonical Style) uygun hale getirecek şekilde otomatik olarak yeniden yazar.

* Indentation, Alignment ve parantez yapılarını standartlaştırır.
* Amaç: Geliştiricilerin kişisel kodlama alışkanlıklarından kaynaklanan stil farklarını ortadan kaldırarak, kod tabanının tek bir elden çıkmış gibi tutarlı olmasını sağlar.

#### 2. Geliştirme Döngüsündeki Konumu

* Genellikle `packer validate`  ve `packer build` komutlarından önce çalıştırılır. Kodun önce görsel/yapısal olarak düzeltilmesi (`fmt`), ardından mantıksal olarak doğrulanması (`validate`) önerilir.
* (`fix` vs `fmt`): `packer fix` komutu eski sürüm şablonları yeni sürüme taşımayı sağlamak (Migration) için kullanılırken; `packer fmt` komutu sadece mevcut kodun stilini (Styling) düzeltir, mantığını değiştirmez.

#### 3.Kod İnceleme Süreçleri

* Eğer `packer fmt` kullanmazsanız, Code Review yapan arkadaşınızın ekranında yüzlerce gereksiz "boşluk düzeltmesi" veya "satır kayması" görünür. Bu gereksiz kalabalık arasında, asıl önemli olan değişikliği (örneğin sunucu boyutunun değişmesini) fark etmek zorlaşır.
* `packer fmt` her şeyi standart hale getirdiği için, Git ekranında sadece gerçek değişiklikler görünür.

#### 4. CI/CD Pipeline'larında "Linting"

Otomasyon süreçlerinde `packer fmt`, bir kalite kontrol kapısı olarak yapılandırılabilir.

* &#x20;CI/CD pipeline'larında genellikle `packer fmt -check` parametresi ile kullanılır. Bu mod, dosyayı değiştirmek yerine, eğer formatı bozuk bir dosya varsa hata kodu (Exit Code) döndürür ve pipeline'ı durdurur.
* Bu yöntem, formatlanmamış veya standart dışı kodun ana depoya (Master/Main Branch) Merge edilmesini teknik olarak engeller.

***

Özetle: `packer fmt`; Packer şablonlarının okunabilirliğini, tutarlılığını ve bakım kolaylığını (Maintainability) artıran, kodun makine tarafından yorumlanmasından ziyade insan tarafından anlaşılmasını kolaylaştıran temel bir statik düzenleme aracıdır.
