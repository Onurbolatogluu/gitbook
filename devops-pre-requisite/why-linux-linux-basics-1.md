# ❤️ Why Linux? Linux Basics #1

<figure><img src="../.gitbook/assets/Basic-Linux-Commands.webp" alt=""><figcaption></figcaption></figure>

Linux, DevOps dünyasında Docker, Kubernetes, Ansible gibi temel araçlarla uyumluluğu, esnekliği, performansı ve geniş topluluk desteği nedeniyle büyük bir öneme sahiptir. Bu nedenle, DevOps süreçlerinde Linux kullanımı yaygındır.

* **Docker**: Docker, Linux çekirdeğinin özelliklerini (cgroups, namespaces) kullanarak konteyner oluşturur. Bu nedenle, Docker ilk olarak Linux üzerinde geliştirilmiştir ve en iyi performansı burada sağlar.
  * **2013**: Docker'ın ilk sürümü tanıtıldı. O dönemde konteyner teknolojisi yeni bir kavramdı ve Docker, bu teknolojiyi yaygınlaştırarak devrim yarattı.
  * **2016**: Docker for Windows piyasaya sürüldü. Docker for Windows, Windows üzerinde Docker konteynerlerini çalıştırabilmek için bir sanal makine (VM) kullanır. Bu sanal makine genellikle Hyper-V veya WSL 2 (Windows Subsystem for Linux 2) üzerinde çalışır. Bu da Docker'ın altında yine bir Linux çekirdeği kullanıldığı anlamına gelir.
*   **Kubernetes**: Kubernetes'in control plane bileşenleri (API server, scheduler, controller manager) Linux üzerinde çalışır.

    * **Windows desteği**: Kubernetes, worker node'larda Windows desteği sunar. Ancak, control plane bileşenleri Linux üzerinde çalışır. Bu, Kubernetes cluster'ının tam anlamıyla çalışabilmesi için bir Linux tabanlı altyapıya ihtiyaç duyduğunu gösterir.


* **Ansible**, Windows host'larını yönetebilir ancak Ansible kendisi bir Linux veya WSL (Windows Subsystem for Linux) üzerinde çalışmalıdır. Bu, Ansible'ın performans ve uyumluluk açısından en iyi Linux üzerinde çalıştığını gösterir.
* **Açık Kaynak ve Topluluk Desteği,** Linux açık kaynaklı bir işletim sistemi olduğundan geniş bir topluluk ve sürekli gelişim desteğine sahiptir. Bu, sürekli güncellenen ve geliştirilen güvenilir bir altyapı sağlar. Açık kaynaklı olması, kullanıcıların ve şirketlerin ihtiyaçlarına göre özelleştirmeler yapabilmesine imkan tanır.
* **Performans ve Güvenilirlik**: Linux yüksek performanslı ve güvenilir bir işletim sistemidir. Sunucu ortamlarında yaygın olarak tercih edilmesinin sebeplerinden biri de budur. Linux'un sağlam mimarisi ve uzun süreli destek sunan sürümleri, onu güvenilir bir seçenek haline getirir.
* **Esneklik ve Özelleştirilebilirlik**: Linux son derece esnek ve özelleştirilebilir bir yapıya sahiptir. Çeşitli dağıtımlar (distributions) ve minimal kurulum seçenekleri ile kullanıcılar, ihtiyaçlarına uygun bir Linux ortamı oluşturabilirler. Bu esneklik, DevOps araçlarının farklı ortamlarda verimli bir şekilde çalışmasını sağlar.
* **CLI (Command Line Interface)**: Linux'un güçlü bir komut satırı arayüzü (CLI) vardır. CLI, sistem yöneticileri ve DevOps mühendisleri için geniş kontrol ve otomasyon imkanları sunar.&#x20;



### Shell Types:

Shell, kullanıcıların işletim sistemi ile etkileşime geçmesini sağlayan bir programdır. Shell, kullanıcının komutlarını alır, bu komutları çalıştırır ve sonuçları kullanıcıya geri döner. Shell, bir komut satırı arayüzü (CLI) sunarak kullanıcıların metin tabanlı komutlar aracılığıyla sistemle etkileşimde bulunmasını sağlar.

#### Linux Shell Türleri

1. **Bourne Shell (Sh Shell)**:
   * **Yer**: /bin/sh
   * **Özellikler**: Bourne Shell, UNIX sistemlerinde kullanılan ilk shell'lerden biridir. Adını geliştiricisi Stephen Bourne'dan alır. Basit ve hızlıdır, özellikle script yazımı için kullanılır.
   * **Kullanım Alanı**: Tarihsel olarak birçok UNIX ve Linux sisteminde varsayılan shell olarak kullanılmıştır.
2. **C Shell (csh veya tcsh)**:
   * **Yer**: /bin/csh veya /bin/tcsh
   * **Özellikler**: C Shell, C programlama diline benzer bir söz dizimi sunar. Tcsh, csh'nin geliştirilmiş bir versiyonudur ve ek özellikler içerir (komut tamamlama gibi).
   * **Kullanım Alanı**: Kullanıcı dostu komut satırı deneyimi sunar ve genellikle programcılar tarafından tercih edilir.
3. **Z Shell (zsh)**:
   * **Yer**: /bin/zsh
   * **Özellikler**: Z Shell, zengin özelliklere sahip bir shell'dir. Bourne Shell ve diğer shell'lerden birçok özelliği bir araya getirir. Gelişmiş komut tamamlama, işlevsellik ve özelleştirme imkanları sunar.
   * **Kullanım Alanı**: Güçlü özellikleri ve geniş özelleştirme imkanları nedeniyle birçok ileri düzey kullanıcı ve sistem yöneticisi tarafından tercih edilir.
4. **Bourne Again Shell (bash)**:
   * **Yer**: /bin/bash
   * **Özellikler**: Bash, GNU Projesi tarafından geliştirilen bir shell'dir ve Bourne Shell'in birçok geliştirilmiş özelliğini içerir. Kapsamlı komut seti ve script yazımı için gelişmiş yetenekler sunar.
   * **Kullanım Alanı**: Linux dağıtımlarında en yaygın kullanılan shell'dir. Hem günlük kullanım hem de script yazımı için idealdir.

#### Neden Farklı Shell Türleri Vardır?

Her shell, farklı kullanıcı ihtiyaçlarını ve kullanım senaryolarını hedefler. Bazıları daha basit ve hızlı iken, bazıları daha gelişmiş özellikler ve özelleştirme imkanları sunar. Kullanıcılar ve sistem yöneticileri, ihtiyaçlarına ve tercihlerine göre uygun shell'i seçebilirler.

{% hint style="info" %}
Terminal, CLI sağlar; shell ise bu CLI'da çalıştırdığınız programdır.
{% endhint %}



### Basic Commands:

```bash
# Ekrana "Hi" yazdırır
echo Hi
# Çıktı: Hi

# Geçerli dizindeki dosya ve klasörleri listeler
ls
# Çıktı: File.txt my_dir1 file2.conf
# (Geçerli dizinde bulunan dosyalar ve klasörler listelenir)

# "my_dir1" adlı dizine geçiş yapar
cd my_dir1

# Geçerli çalışma dizinini görüntüler
pwd
# Çıktı: /home/my_dir1
# (Mevcut dizinin tam yolunu gösterir)

# "new_directory" adında yeni bir dizin oluşturur
mkdir new_directory

# "new_directory" adlı dizine geçiş yapar, "www" adında bir dizin oluşturur ve geçerli çalışma dizinini görüntüler
cd new_directory; mkdir www; pwd
# Çıktı: /home/my_dir1/new_directory
# (Önce "new_directory" dizinine geçilir, ardından bu dizin içinde "www" dizini oluşturulur ve son olarak geçerli dizinin tam yolu görüntülenir)

# ; (noktalı virgül) işareti ile birden fazla komut aynı satırda ardışık olarak çalıştırılır
# Örneğin: cd new_directory; mkdir www; pwd
# Bu komut satırı, önce "new_directory" dizinine geçiş yapar, ardından bu dizin içinde "www" dizinini oluşturur ve son olarak geçerli dizinin tam yolunu gösterir.

# ; işareti, birden fazla komutun ardışık olarak çalıştırılmasını sağlar. Her bir komut bağımsız olarak çalıştırılır. 
# Örneğin, cd new_directory komutu hata verirse, mkdir www ve pwd komutları yine de çalıştırılır.
# Yani ; işareti, önceki komutun başarılı olup olmamasına bakmaksızın sonraki komutları çalıştırır.

```



#### Directory Commands:

```shell
# Tek tek dizin oluşturma
mkdir /tmp/asia
# /tmp dizininde "asia" adında bir dizin oluşturur.

mkdir /tmp/asia/india
# /tmp/asia dizininde "india" adında bir dizin oluşturur.

mkdir /tmp/asia/india/bangalore
# /tmp/asia/india dizininde "bangalore" adında bir dizin oluşturur.

# -p bayrağı ile hiyerarşik dizin oluşturma
mkdir -p /tmp/asia/india/bangalore
# Yukarıdaki tüm dizinleri tek bir komutla oluşturur. Eğer dizinler zaten varsa, hata vermez ve mevcut olanları kullanır.

# Dizin ve içeriğini silme
rm -r /tmp/my_dir1
# /tmp/my_dir1 dizinini ve içindeki tüm dosya ve alt dizinleri siler.
# -r bayrağı "recursive" anlamına gelir ve dizin içeriğini de siler.

# Dizin ve içeriğini kopyalama
cp -r my_dir1 /tmp/my_dir1
# my_dir1 dizinini ve içindeki tüm dosya ve alt dizinleri /tmp dizinine kopyalar.
# -r bayrağı "recursive" anlamına gelir ve dizin içeriğinin tamamını kopyalar.
```



#### File Commands:

```bash
# Boş bir dosya oluşturur (içerik eklemez)
touch new_file.txt
# new_file.txt adlı boş bir dosya oluşturur.

# Dosyaya içerik ekler
cat > new_file.txt
# Bu komut, new_file.txt dosyasına içerik eklemenizi sağlar.
# Yazmaya başladıktan sonra, dosya içeriğini bitirmek için Ctrl+D tuş kombinasyonunu kullanın.
# Örnek İçerik: "This is some sample contents" yazın ve Ctrl+D ile bitirin.

# Dosya içeriğini görüntüler
cat new_file.txt
# new_file.txt dosyasının içeriğini ekrana yazdırır.
# Çıktı: This is some sample contents

# Dosyayı kopyalar
cp new_file.txt copy_file.txt
# new_file.txt dosyasını copy_file.txt adıyla kopyalar.
# Eğer copy_file.txt adında bir dosya zaten varsa, cp komutu new_file.txt dosyasının içeriğini copy_file.txt dosyasının üzerine yazar. Yani, copy_file.txt dosyasının mevcut içeriği silinir ve new_file.txt dosyasının içeriği ile değiştirilir.

# Dosyayı yeniden adlandırır
mv new_file.txt sample_file.txt
# Eğer 'sample_file.txt' mevcut değilse, 'new_file.txt' dosyasını 'sample_file.txt' olarak yeniden adlandırır. 
# Eğer sample_file.txt mevcut bir dosya ise, mv komutu new_file.txt dosyasını sample_file.txt dosyasının üzerine yazar ve sample_file.txt dosyasının önceki içeriği silinir.

# Dosyayı bir dizine taşır
mv new_file.txt /some_directory/
# Eğer '/some_directory/' mevcut bir dizinse, 'new_file.txt' dosyasını bu dizine taşır.
# Eğer belirtilen dizin mevcut değilse, mv komutu hata verir ve dosyayı taşımayı gerçekleştiremez.

# Dosyayı siler
rm new_file.txt
# new_file.txt dosyasını siler.

```

