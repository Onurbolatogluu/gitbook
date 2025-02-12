---
icon: down-right
---

# Workflow Commands

**GitHub Actions Workflow Commands,** GitHub Actions içinde **özel environment değişkenleri ayarlamak**, **PATH’e dizin eklemek**, **step’ler arasında çıktı paylaşmak** veya **log’ları düzenlemek** gibi işleri yapmanı sağlar.

### 1) Ortam Değişkenleri (Environment Variables) Ayarlama

* **Amaç**: Bir step’te (komutta) oluşturduğun değişkeni, **sonraki adımlarda** da kullanılabilir hale getirmek.
* **Nasıl Yapılır?**
  * `echo "KEY=VALUE" >> $GITHUB_ENV` komutunu kullanarak bir adımda `GITHUB_ENV` dosyasına yazarsın.
  * Örnek;

```yaml
steps:
  - name: Set environment variable
    run: echo "ACTION_ENV=production" >> $GITHUB_ENV

  - name: Print environment variable
    run: echo "$ACTION_ENV"
```

* İkinci adımdan itibaren `ACTION_ENV` ortam değişkeni `production` değeriyle kullanılabilir.



### 2) PATH’e Dizin Ekleme

* **Amaç**: Özel bir script veya binary dizinini `PATH` değişkenine ekleyerek, sonraki adımlarda direkt komut ismiyle çağırabilmek.
* **Nasıl Yapılır?**
  * `echo "/path/to/dir" >> $GITHUB_PATH` şeklinde yazdığında, “/path/to/dir” dizini otomatik olarak `PATH`’e eklenir.

```yaml
steps:
  - name: Add directory to PATH
    run: echo "$GITHUB_WORKSPACE/my_scripts" >> $GITHUB_PATH

  - name: Check new path
    run: echo "$PATH"
```

Bundan sonraki adımlarda “`my_scripts`” içindeki komutları `./komut` yerine doğrudan `komut` ile çalıştırabilirsin.



### 3) Step Çıkışlarını (Outputs) Ayarlama ve Kullanma

* **Amaç**: Bir adımda üretilen değeri, başka adımlarda ya da başka job’larda (eğer “needs” ilişkisi varsa) kullanmak.
* **Nasıl Yapılır?**
  * `echo "result=output_value" >> $GITHUB_OUTPUT` diyerek bir step’in `outputs.result` değerini belirtebilirsin.
  * Sonra aynı job’da sonraki adımlarda `steps.<step_id>.outputs.result` şeklinde erişirsin.

```yaml
steps:
  - name: Set output
    id: example_step
    run: echo "result=output_value" >> $GITHUB_OUTPUT

  - name: Use output
    run: echo "The output was ${{ steps.example_step.outputs.result }}"

```



### 4) Debug/Log Komutları

* **Grup Oluşturma (`::group::` / `::endgroup::`)**
  * Log’larda mesajları **gruplamak**, okunabilirliği artırmak için kullanılır.

```yaml
steps:
  - name: Group log messages
    run: |
      echo "::group::My Grouped Messages"
      echo "Message 1"
      echo "Message 2"
      echo "::endgroup::"
```

* GitHub Actions log ekranında bu mesajlar “My Grouped Messages” altında katlanabilir (collapsible) hale gelir.

**Debug Mesajı (`::debug::`)**

* Sadece debug mod açıkken (Settings→Actions→Enable Debug Logging) görünecek mesajlar.

```yaml
run: echo "::debug::This is a debug message"
```

**Uyarı Mesajı (`::warning::`)**

* Log’da “Warning” olarak görünür, workflow başarısız olmaz ama dikkat çeker.

```yaml
run: echo "::warning::This is a warning"
```

**Hata Mesajı (`::error::`)**

* Log’da “Error” olarak görünür, workflow’u fail yapabilir.

```yaml
run: echo "::error::Something went wrong!"
```



### 5) Değerleri Maskeleme (Secrets Gibi)

* **Amaç**: Log’larda hassas bilgileri gösterme yerine `***` ile gizlemek.
* **Nasıl Yapılır?**
  * `echo "::add-mask::<value>"` komutunu kullanarak, log’larda `<value>` göründüğü yerde `***` göstermesini sağlarsın.

```yaml
run: echo "::add-mask::${{ secrets.SECRET_VALUE }}"
```

* Bu komuttan sonra, SECRET\_VALUE log satırlarında görünse de yıldızlı halde gösterilir.

### 6) Workflow’u Durdurma / Hata Verme

* **Amaç**: Belli bir koşul sağlanmazsa ya da kritik bir hata oluşursa, workflow’u manuel olarak durdurup “fail” et.
* **Nasıl Yapılır?**
  * `echo "::error::This is an error message"` → Step hata verip workflow fail alır.
  * Ya da bash exit kodu ile `exit 1` diyerek yine step’i fail’e düşürebilirsin.

### Özet

* **`echo "KEY=VALUE" >> $GITHUB_ENV`** → Ortam değişkeni ekleme
* **`echo "/path/to/dir" >> $GITHUB_PATH`** → PATH’e dizin ekleme
* **`echo "result=something" >> $GITHUB_OUTPUT`** → Step output oluşturma
* **`echo "::debug::message"`, `::warning::`, `::error::`** → Özel log mesajları
* **`echo "::group::Title"` / `echo "::endgroup::"`** → Log gruplama
* **`echo "::add-mask::<value>"`** → Hassas verileri maskeleme

Bu komutlar, GitHub Actions’ın “run” adımları içinde **shell üzerinden** “echo” komutlarıyla iletişim kurmak için sağladığı bir arabirimdir. Böylece **daha gelişmiş** script mantıkları, debug çıktıları, step’ler arası veri paylaşımı gibi özellikleri kullanabilirsin.
