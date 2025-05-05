---
icon: apple-core
---

# Introduction to Handlers

### 1. Handlers Nedir, Ne İşe Yarar?

Normalde bir konfigürasyon dosyasında değişiklik yaptığınızda, değişikliğin etkin olması için örneğin web servisinin **yeniden başlatılması (restart)** gerekir. Bunu _manuel_ yapmak yerine Ansible’da **Handlers** tanımlayarak, ilgili “Task” bittiğinde servisin **otomatik** olarak restart edilmesini sağlayabilirsiniz.

**Örnek Durum**

* Bir web sunucusunda `/etc/nginx/nginx.conf` dosyasını güncellediniz.
* Bu dosya değiştiyse “nginx servisini yeniden başlat” demek istersiniz.
* **Handler** dediğimiz “otomatik tetiklenen bir görev” tam da burada devreye girer.

### 2. “notify” Mekanizması

Ansible’da **Handlers** şu mantıkla çalışır:

1. **Task** içinde `notify:` anahtarını kullanırsınız.
2. `notify:` ile bir **handler** adı verirsiniz (örn. “Restart Web Service”).
3. Playbook’un sonunda, `handlers:` bölümünde bu adı taşıyan “handler” görevi tanımlarsınız.
4. Eğer söz konusu Task **changed** (yani gerçekten bir değişiklik yaptıysa), Handler “tetiklenir” ve en sonda çalışır.
   * Bu sayede gereksiz restart’lar olmaz; eğer dosya zaten aynıysa, Handler atlanır.

### 3. Örnek Basit Playbook

```yaml
- name: Deploy Application
  hosts: application_servers
  tasks:
    - name: Copy Application Code
      copy:
        src: app_code/
        dest: /opt/application/
      notify: Restart Application Service

  handlers:
    - name: Restart Application Service
      service:
        name: application_service
        state: restarted
```

1. **Tasks Bölümü**
   * `copy:` modülü ile “app\_code/” dizinindeki dosyaları “/opt/application/” konumuna kopyalıyor.
   * Bu kopyalama gerçekten yeni bir dosya kopyaladıysa, **“changed”** olur, ve `notify: Restart Application Service` ifadesi devreye girer.
2. **Handlers Bölümü**
   * “Restart Application Service” adlı bir handler var.
   * Bu handler, `service:` modülü kullanarak “application\_service” adındaki servisi “restarted” (yeniden başlatılmış) durumda olmasını garanti eder.

**Sonuç**: Uygulama kodu her güncellendiğinde, Ansible otomatik olarak ilgili servisi restart ediyor. Manuel uğraş yok.

### 4. Handlers’ın Avantajları

1. **Otomasyon**: Bir konfig değiştiğinde veya yeni kod deploy edildiğinde servisi _kendiliğinden_ yeniden başlatır.
2. **İdempotent**: Sadece gerçekten “changed” (değişiklik olmuş) durumda tetiklenir, aksi halde boşa restart yapmaz.
3. **Daha Az Hata**: İnsan hatası (restart unutma vb.) riskini düşürür.
4. **Daha Temiz Yapı**: Playbook’larınızda “kodu kopyala → servisi restart et” gibi mantığı net bir şekilde ayırırsınız.

***

### 5. Özet

* **Handlers**, Ansible’ın **“değişiklik sonrasında ek işlem yap”** mekanizmasıdır.
* Task’ınızda `notify:` kullanarak bir handler’ı çağırırsınız, eğer Task gerçekten bir yenilik yaptıysa, handler “changed” sinyali alır ve **Playbook’un sonunda** o işlemi (ör. restart) yapar.
* Böylece konfig/uygulama değiştikçe otomatik işlemler (servis restart gibi) **sistemli** ve **güvenilir** biçimde gerçekleştirilir.
