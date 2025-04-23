---
icon: clipboard-list-check
---

# Ansible lint

### 1. Ansible Lint Nedir ve Neden Önemli?

* **Ansible Lint**, Ansible playbook’larınızı (ayrıca roller, koleksiyonlar vb.) **özel kurallara** göre inceleyen bir **komut satırı aracıdır**.
* Playbook’larınızda olası **hataları**, **uygunsuz stil** (indentation, adlandırma vb.), **yanlış modül kullanımları**, hatta **güvenlik** veya **depreceated (kullanımı önerilmeyen)** yapıların kullanımını tespit eder.
* **Check mode** veya **diff mode** Ansible’ın kendi doğrulamasıyken (gerçekte ne değişeceğini görmek), **Ansible Lint** daha çok “**kod kalitesi**, **stil**, **en iyi pratikler**” yönünden bir rehber görevi görür.

**Örnek Senaryo**:\
Büyük bir organizasyonda, zamanla yüzlerce playbook yazıyorsunuz. Kod kalitesini korumak için, her playbook’u yazdıktan/değiştirdikten sonra Ansible Lint çalıştırarak **tutarsızlıkları** erkenden yakalayabilirsiniz.

### 2. Nasıl Kurulur ve Nasıl Çalıştırılır?

1. **Kurulum**
   *   Genellikle `pip` üzerinden kurabilirsiniz:

       ```bash
       pip install ansible-lint
       ```
   * Veya sistem paket yöneticisinden (dağıtıma göre `apt-get install ansible-lint`, vb.).
2. **Çalıştırma**
   *   Temel kullanım:

       ```bash
       ansible-lint <playbook_dosyası>.yml
       ```
   * Komut, playbook içindeki potansiyel hataları ve uyarıları listeleyecektir.

### 3. Lint Yaparken Karşılaşabileceğiniz Uyarılara Örnekler

Örneğin `style_example.yml` adında bir playbook’unuz var ve içerisinde **stilde** veya **modül kullanımında** bazı sorunlar var. `ansible-lint style_example.yml` dediğinizde şu tip uyarılar alabilirsiniz:

```bash
[WARNING]: incorrect indentation: expected 2 but found 4 (syntax/indentation)
style_example.yml:6

[WARNING]: command should not contain whitespace (blacklisted: ['apt']) (commands)
style_example.yml:6

[WARNING]: Use shell only when shell functionality is required (deprecated in favor of 'cmd') (commands)
style_example.yml:6

[WARNING]: command should not contain whitespace (blacklisted: ['service']) (commands)
style_example.yml:12

[WARNING]: 'name' should be present for all tasks (task-name-missing) (tasks)
style_example.yml:14
```

#### Uyarıların Anlamı

1. **incorrect indentation**: Satır 6’da girinti hatası (2 boşluk yerine 4 boşluk).
2. **command should not contain whitespace**: Bazı modüller (`apt`, `service`) komut modülüyle değil, kendi özel modülleriyle kullanılmalı.
3. **Use shell only when shell functionality is required**: `shell:` modülünü gereksiz kullanmak yerine `command:` veya ilgili modülü kullanmanız önerilir.
4. **task-name-missing**: Bütün task’lerin `name:` parametresi olmalı (kolay okunabilirlik ve hata ayıklama için).

### 4. Ortaya Çıkan Uyarıları Nasıl Düzeltiriz?

* **Girinti Hatası**: YAML formatında en yaygın hatalardan biri, her task’in 2 boşlukla (veya 4 ama tutarlı şekilde) girintilenmesi. Ansible Lint “2 bekliyorum, 4 buldum” diyorsa, tüm satırları tutarlı hale getirin.
* **Kullandığınız modülü gözden geçirin**: Örneğin bir paket kuracaksanız `apt` veya `yum` gibi ilgili modülleri kullanın, shell/command ile paket kurmaktan kaçının.
* [**Görev isimleri**: Her task’te `name:` olmasını ve anlaşılır bir ifade yazılmasını önerir (playbook okuyanın anlaması kolay olsun).](#user-content-fn-1)[^1]
* **Ek parametreler**: Mesela “update\_cache: yes” gibi en iyi pratikleri eklemeniz de tavsiye edilebilir.

Uyarılar bazen “hata” değil “öneri” olabilir. Kendi kod standartlarınıza göre karar verebilirsiniz. Ama genelde Ansible Lint uyarıları **en iyi uygulamalara** işaret eder.

### 5. Eğer Sessiz Kalırsa?

* **“No issues found”** veya hiçbir çıktı yoksa, demek ki Ansible Lint playbook’unuzu hatasız/stil olarak düzgün bulmuş. Tebrikler!
* Düzenli olarak lint işlemini **CI/CD** pipeline 'nıza ekleyerek, kod kalitesini her değişiklikte otomatik kontrol edebilirsiniz.

### 6. Sonuç ve Özet

* **Check/ Diff mode**: “Bu playbook çalışırsa, sistemde ne değişecek/ gerçekte nasıl davranacak?”
* **Ansible Lint**: “Bu playbook **kod kalitesi** ve **en iyi pratikler** açısından doğru mu? Girintiler, modül kullanımı, isimlendirme tutarlı mı?”
* Lint’i kullanmak, playbook’larınız büyüdükçe **tutarlılık** ve **kalite** sağlar, hataları erken farkettirir.





[^1]: _varsayılan olarak_ bir göreve (task) isim vermeseniz bile Ansible bir hata vermez; görev ismi olmadan da çalışır. Ancak **iyi bir pratik** olarak her göreve `name:` eklemeniz önerilir. Bunun sebebi, playbook’un daha **okunur**, **anlaşılır** ve **hata ayıklaması kolay** olmasıdır.
