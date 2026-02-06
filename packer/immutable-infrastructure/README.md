---
icon: mask-snorkel
---

# Immutable Infrastructure

#### 1. "Baking"

* İşletim sistemi, uygulama kodu, bağımlılıklar ve sistem ayarları tek bir katmanda birleştirilerek nihai bir Makine İmajı (Golden Image) oluşturulur.
* Yayına alınan sunucular, teorik olarak "Read-Only" (Salt Okunur) kabul edilir. Sunucu başlatıldıktan sonra üzerine SSH ile bağlanıp yama yapmak (Hot-fixing) veya konfigürasyon değiştirmek mimari prensiplere aykırıdır.

#### 2. Configuration Drift Yönetimi

Geleneksel (Mutable) altyapılarda, zamanla sunucular üzerinde yapılan manuel değişiklikler, sistemlerin birbirinden farklılaşmasına (Drift) neden olur. Immutable Infrastructure bu sorunu kökten çözer:

* Prod ortamındaki 100 sunucu da aynı İmaj ID'sinden (örneğin `ami-v1.0`) türetildiği için, tüm sunucuların bit seviyesinde birbirinin kopyası olduğu garanti edilir.
* "Dev ortamında çalışıyordu ama Prod çalışmıyor" sorunu, Ortam Eşitliği sayesinde elimine edilir. Packer, her iki ortam için de aynı şablonu kullanır.

#### 3. Sürüm Kontrolü ve Dağıtım Stratejisi

Bu model, yazılım geliştirme süreçlerindeki versiyonlama mantığını altyapıya uygular.

* Bir güncelleme gerektiğinde (v1 -> v2), mevcut sunuculara yama geçilmez. Yeni bir `v2` imajı derlenir.
* Yeni versiyon (`v2`) sunucular, eski versiyonun (`v1`) yanına kurulur. Trafik yeni sunuculara yönlendirilir ve stabilite doğrulandıktan sonra eski sunucular devreden çıkarılır.
* Yeni versiyonda kritik bir hata tespit edilirse, süreç tersine işletilerek saniyeler içinde eski ve kararlı olan `v1` imajına dönülür.

#### 4. Entegrasyon ve Kod Olarak Altyapı

Packer, bu stratejinin "İnşa" bacağını yönetirken, diğer araçlarla entegre çalışarak süreci tamamlar.

* Orkestrasyon: Terraform veya CloudFormation gibi IaC araçları, Packer'ın ürettiği AMI ID'sini (Artefakt) girdi olarak alır ve altyapı dağıtımını gerçekleştirir.
* CI/CD Pipeline: Her kod değişikliği (Commit), otomatik bir Packer sürecini tetikler. Bu, altyapının da uygulama kodu gibi sürekli test edilebilir ve dağıtılabilir bir ürün olmasını sağlar.
