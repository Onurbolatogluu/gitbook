---
icon: users-slash
---

# Use variables to retrieve the results of running commands

Ansible’da bir komut veya görev çalıştırdığımızda, çoğu zaman çıktısını (output) görmek veya daha sonrasında bu veriyi başka bir göreve girdi olarak kullanmak isteriz. Bunu yapmak için Ansible bize **register** adlı bir direktif sunar. Bu sayede komut sonuçlarını bir değişkene kaydedip istediğimiz zaman kullanabiliriz.&#x20;

### 1. Neden Register Kullanıyoruz?

* Bir komutun çıktılarını başka bir adımda kullanmak isteyebiliriz. Örneğin, `/etc/hosts` dosyasının içeriğini okuyup sonra dosyayı kontrol etmek veya içindeki veriye göre başka işlemler yapmak isteyebiliriz.
* Başka bir örnekte, bir komutun **dönüş kodunu** (return code) inceleyip, 0 mı değil mi diye bakıp koşullu bir işlem (örneğin `when:` ifadesi ile) gerçekleştirebiliriz.

Kısacası, `register` değişkeni sayesinde Ansible’ın her bir host üzerinde çalıştırdığı komutların detaylarına erişebiliyor ve bu bilgileri sonraki adımlarda (tasks) kullanarak dinamik bir yapı kurabiliyoruz.

### 2. Basit Bir Örnek: `/etc/hosts` Dosyasını Okumak

Aşağıdaki basit playbook örneğinde, her bir host üzerinde `cat /etc/hosts` komutu çalıştırılıp, çıktı bir değişkene kaydedilir. Sonrasında `debug` modülüyle ekrana bastırılır.

```yaml
- name: Check /etc/hosts file
  hosts: all
  tasks:
    - shell: cat /etc/hosts
      register: result  # <--- Komutun çıktısı "result" değişkenine kaydedilir

    - debug:
        var: result
```

**Çalışma sırası:**

1. `shell: cat /etc/hosts` komutu her bir host üzerinde çalışır.
2. Komutun çıktısı `result` adlı bir değişkene kaydedilir.
3. `debug` modülü ile `result` değişkeni ekrana bastırılır.

Bu çalıştığında, Ansible bize `result` altında komutun çıktısını, dönüş kodunu (`rc`), komutun ne kadar sürdüğünü (`delta`) ve benzeri detayları gösterecektir.

### 3. `result` Değişkeninin İçeriğini Anlamak

`register` edilen değişken, genellikle **JSON benzeri** bir sözlük (dictionary) yapısına sahiptir. Aşağıdakiler sıkça kullanılan alanlardır:

* **stdout**: Komutun standart çıktısı (tek bir metin bloğu halinde).
* **stdout\_lines**: Komutun standart çıktısının satır satır dizisi.
* **stderr**: Hata çıktılarını barındırır.
* **stderr\_lines**: Hata çıktılarının satır satır dizisi.
* **rc**: Komutun dönüş kodu. 0 ise başarı, 0 değilse hata/sorun olduğunu gösterir.
* **start**, **end**, **delta**: Komutun ne zaman başlayıp bittiğini ve kaç saniye sürdüğünü gösterir.

Playbook’taki `debug: var: result` ile çıktıyı inceleyip hangi alana ihtiyacınız olduğunu görebilirsiniz.

### 4. İçeriğin Belirli Kısımlarına Erişmek

Bazen bütün bilgiyi değil, sadece belli kısımları almak isteriz. Bu durumda `debug`’ta direkt olarak istediğimiz alana erişebiliriz.

```yaml
- name: Check /etc/hosts file
  hosts: all
  tasks:
    - shell: cat /etc/hosts
      register: result

    - debug:
        var: result.stdout
```

Bu şekilde çıktıda sadece `stdout` bilgisi (yani `/etc/hosts` içeriği) gözükecektir. Aynı mantıkla:

```yaml
- debug:
    var: result.stderr  # Hata mesajlarını ekrana yazdırır
```

veya

```yaml
- debug:
    var: result.rc      # Komutun dönüş kodunu ekrana yazdırır
```

kullanabilirsiniz.

### 5. Kayıtlı Değişkenlerin Kapsamı (Scope)

Önceki derslerden hatırlayacağınız gibi, Ansible’da kayıtlı değişkenler **host kapsamındadır (host scope)**. Yani bir görev, `web1` üzerinde çalışıyorsa `result` sadece `web1` için geçerlidir. Başka bir host (`web2` gibi) üzerinde aynı değişken farklı bir değere sahip olabilir.

Ek olarak, **playbook’un aynı koşum (run) süresince** aynı host üzerinde sonraki görevlerde de `result` değerine erişebilirsiniz. Yeter ki aynı playbook içinde kalınmış olsun.

### 6. Birden Fazla Play’de Kullanım

Eğer playbook’unuzda birden fazla `play` tanımladıysanız, her iki play’de de aynı host’lar yer alıyorsa, Ansible aynı “host scope” değişkenlerini tutmaya devam eder. Dolayısıyla bir play’de kaydettiğiniz `result`, aynı playbook içindeki bir sonraki play’de de kullanılabilir.

Örnek:

```yaml
- name: Check /etc/hosts file
  hosts: all
  tasks:
    - shell: cat /etc/hosts
      register: result

    - debug:
        var: result.rc

- name: Second Play
  hosts: all
  tasks:
    - debug:
        var: result.rc
```

Burada “Second Play” kısmında da `result.rc` değerine erişilebilmekte. Tabii bu, aynı **playbook** içindeki aynı host olduğu için geçerlidir.

Playbook’ları `-v` (verbose) seçeneğiyle çalıştırırsanız (örn. `ansible-playbook playbook.yml -v`), birçok detayı ekstra herhangi bir `debug` satırı yazmaya gerek kalmadan görebilirsiniz. Daha ayrıntılı çıktılar için `-vv` veya `-vvv` gibi ek düzeyler de kullanabilirsiniz.
