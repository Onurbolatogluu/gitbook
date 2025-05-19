# 🟦 Python Introduction

Python, kullanımı kolay ve oldukça popüler bir programlama dilidir. 1991 yılında Guido van Rossum tarafından yaratılmıştır. Python’un en büyük avantajlarından biri, okunabilirliğe ve basitliğe büyük önem vermesidir. Yani, karmaşık işlemleri bile çok daha az ve anlaşılır kodla gerçekleştirebilirsiniz. Bu nedenle, yeni başlayanlardan deneyimli geliştiricilere kadar herkes tarafından sevilir.

Ubuntu için python ve pip kurulumu: [https://www.cherryservers.com/blog/how-to-install-pip-ubuntu](https://www.cherryservers.com/blog/how-to-install-pip-ubuntu)



#### Sıklıkla kullanılan Python komutları,

```bash
# Python shell başlatmak
python
# Python shell'ini başlatır.

# Belirli bir Python dosyasını çalıştırmak
python script.py
# script.py dosyasını çalıştırır.

# Python versiyonunu kontrol etmek
python --version
# Yüklü olan Python'un versiyonunu gösterir.

# Belirli bir modül hakkında yardım almak
python -m pydoc module_name
# Belirli bir Python modülü hakkında yardım dökümantasyonu sağlar.

# Python'un etkileşimli modunu başlatmak
python -i script.py
# script.py dosyasını çalıştırır ve sonrasında etkileşimli moda geçer.

# Python komutlarını doğrudan komut satırından çalıştırmak
python -c "print('Hello, World!')"
# Komut satırından Python kodu çalıştırır ve "Hello, World!" yazdırır.

# Belirli bir Python modülünü çalıştırmak
python -m module_name
# Belirtilen Python modülünü çalıştırır.

# Bir Python sanal ortamı oluşturmak
python -m venv myenv
# Bir sanal Python ortamı oluşturur.

# Python dosyasını optimize ederek çalıştırmak
python -O script.py
# script.py dosyasını optimize ederek çalıştırır (assertion'ları ve __debug__ bloklarını atlar).

# Python dosyasını derlemek
python -m py_compile script.py
# script.py dosyasını derler ve .pyc bytecode dosyasını oluşturur.

```



### PIP,

Pip, Python'da paket yönetim aracıdır. Python paketlerini ve bağımlılıklarını kolayca yüklemenizi, güncellemenizi ve yönetmenizi sağlar. Pip, Python'un standart kütüphanelerinden biridir ve genellikle Python ile birlikte gelir.

```bash
# Pip versiyonunu kontrol etme
pip --version
# Bu komut, sistemde yüklü olan pip'in versiyonunu gösterir.

# Bir paket yükleme
pip install package_name
# Belirtilen `package_name` adlı paketi PyPI'den indirir ve kurar.

# Belirli bir sürümü yükleme
pip install package_name==1.2.3
# `package_name` adlı paketin 1.2.3 sürümünü indirir ve kurar.

# Paket güncelleme
pip install --upgrade package_name
# `package_name` adlı paketin en son sürümünü indirir ve kurar.

# Yüklü paketleri listeleme
pip list
# Sistemde yüklü olan tüm Python paketlerini listeler.

# Yüklü paketler hakkında daha fazla bilgi edinme
pip show package_name
# `package_name` adlı paketin detaylarını gösterir.

# Bir paketi kaldırma
pip uninstall package_name
# `package_name` adlı paketi sistemden kaldırır.

# Bağımlılıkları belirtilen bir dosyadan yükleme
pip install -r requirements.txt
# `requirements.txt` dosyasında belirtilen tüm paketleri ve sürümleri indirir ve kurar.

# Paketlerin depoda en son sürümlerini listeleme
pip list --outdated
# Güncellenmesi gereken paketleri listeler.

# Bir paket arama
pip search package_name
# `package_name` adlı paketi PyPI'de arar ve eşleşen paketlerin listesini gösterir.

# Kurulu paketlerin yolunu gösterme
pip show --files package_name
# `package_name` adlı paketin sistemde nerede kurulu olduğunu gösterir.

# Tüm bağımlılıkları indirerek paketleri yükleme
pip install package_name[extra]
# `package_name` adlı paketi ve belirtilen ekstra bağımlılıkları indirir ve kurar.

# Pip'i güncelleme
pip install --upgrade pip
# Pip'i en son sürüme günceller.

```



Pip ile kurulan paketler, Python'ın kurulumuna ve işletim sistemine bağlı olarak değişiklik gösterebilir. Paketlerin nereye kurulduğunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz:

`pip show package_name`

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

***

`sys.path`, Python'un modül aramak için kullandığı dizinlerin bir listesidir. Python bir modülü içe aktardığında, önce bu listede yer alan dizinlerde arar. Bu liste, Python'un hangi dizinlerde modül aradığını ve hangi yolların Python tarafından kullanıldığını gösterir.

```bash
python3 -c "import sys; print(sys.path)"
```

<figure><img src="../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

* **Boş Dize (`''`)**: Geçerli çalışma dizinini temsil eder.
* **`/usr/lib/python310.zip`**: Python modüllerinin sıkıştırılmış bir dosyada bulunduğu yeri temsil eder.
* **`/usr/lib/python3.10`**: Python'un kurulu olduğu dizin ve standart kütüphaneler.
* **`/usr/lib/python3.10/lib-dynload`**: Dinamik olarak yüklenebilir modüller.
* **`/usr/local/lib/python3.10/dist-packages`**: Kullanıcı tarafından yüklenen paketlerin bulunduğu yer.
*   **`/usr/lib/python3/dist-packages`**: Sistem paket yöneticisi tarafından yüklenen Python paketleri.





#### requirements.txt,

`requirements.txt` dosyası, Python projelerinde kullanılan ve projeye bağımlı olan tüm paketlerin ve bu paketlerin sürümlerinin listesini içeren bir dosyadır. Bu dosya, projeyi başka bir ortama taşırken veya başka bir geliştiricinin projeyi çalıştırması gerektiğinde, projeye gerekli tüm bağımlılıkların kolayca kurulmasını sağlar.&#x20;

**İçeriği,**

`requirements.txt` dosyasının içeriği basit bir metin formatındadır. Her satır bir paketi ve isteğe bağlı olarak sürümünü belirtir. Örneğin:

```
numpy==1.21.0
pandas>=1.3.0
requests<=2.25.1
flask
```

* `numpy==1.21.0`: NumPy paketinin kesin olarak 1.21.0 sürümünü kurar.
* `pandas>=1.3.0`: Pandas paketinin en az 1.3.0 sürümünü kurar.
* `requests<=2.25.1`: Requests paketinin en fazla 2.25.1 sürümünü kurar.
* `flask`: Flask paketinin en son sürümünü kurar.

**Kurulum,**

`requirements.txt` dosyasındaki bağımlılıkları kurmak için aşağıdaki komutu kullanabilirsiniz:

```bash
pip install -r requirements.txt
```

**Örnek,**

Örneğin, bir Flask uygulamanız varsa ve bu uygulama aşağıdaki paketlere ihtiyaç duyuyorsa:

* Flask
* SQLAlchemy
* Requests

Bu paketleri yükledikten sonra, `pip freeze > requirements.txt` komutunu çalıştırarak aşağıdaki gibi bir `requirements.txt` dosyası oluşturabilirsiniz:

```
Flask==2.0.1
SQLAlchemy==1.4.22
requests==2.25.1
```

Bu dosyayı başka bir geliştiriciye veya sunucuya taşıdığınızda, `pip install -r requirements.txt` komutunu çalıştırarak gerekli tüm bağımlılıkları kolayca kurabilirsiniz.



#### Other Package Managers,

* `easy_install`, Python `setuptools` paketinin bir parçasıdır ve Python Paket Dizini (PyPI) üzerinde barındırılan paketleri indirir ve yükler.
  *   İlk olarak, `easy_install` aracının sisteminizde kurulu olduğundan emin olun. `easy_install`, `setuptools` paketi ile birlikte gelir, bu nedenle `setuptools` paketini kurarak `easy_install` aracını da kurabilirsiniz:

      ```bash
      pip install setuptools
      ```
  *   **Flask Kurulumu:**

      ```bash
      easy_install Flask
      ```



* `Wheels`, Python'da paket dağıtımı için kullanılan modern bir formattır ve `.whl` dosya uzantısına sahiptir. `pip` aracı ile birlikte çalışır ve önceden derlenmiş paketleri hızlı bir şekilde yükler.
  * Diyelim ki flask wheel dosyasını indirdiniz. İndirdiğiniz dosya adının Flask-2.0.2-py3-none-any.whl olduğunu varsayalım.&#x20;
  *   **Flask Wheel Dosyasını İndirin:**

      ```bash
      pip wheel flask
      ```
  * Bu komut, mevcut dizine `flask` için bir wheel dosyası indirir.
  *   **Flask Wheel Dosyasını Kurun:**

      ```bash
      pip install Flask-2.0.2-py3-none-any.whl
      ```

