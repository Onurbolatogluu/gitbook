# ⏺ Recording Rules

Prometheus Recording Rules, Prometheus'un önceden hesaplanmış zaman serisi verilerini saklamasını sağlayan bir özelliktir. Bu, belirli sorguların sonuçlarını düzenli aralıklarla otomatik olarak hesaplayarak ve saklayarak, bu verilere daha sonra daha hızlı ve verimli bir şekilde erişilebilir hale getirir.

1. Recording Rules, Prometheus'ta sıkça ihtiyaç duyulan veya hesaplaması pahalı olan ifadeleri önceden hesaplamaya ve sonuçlarını Prometheus depolamasında yeni bir zaman serisi olarak saklamaya olanak tanır. Bu, performansı artırarak sorguları daha hızlı ve verimli bir şekilde gerçekleştirmenize yardımcı olur.
2. Prometheus, düzenli aralıklarla değerlendirilebilen ve yapılandırılabilen iki tür kuralı destekler: Recording Rules (Kayıt Kuralları) ve Alerting Rules (Uyarı Kuralları). Recording Rules, önceden hesaplanmış verileri saklamak için kullanılırken, Alerting Rules, belirli koşulların karşılandığında uyarılar oluşturmak için kullanılır.
3. Recording Rules, YAML formatında yazılır.  Bu sayede, Recording Rules'un yapılandırması ve yönetimi kolaylaşır.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Prometheus Recording Rules, bir `.rules` dosyasında saklanır ve bu dosya Prometheus yapılandırma dosyası (`prometheus.yml`) içinde tanımlanır. İlgili sorguların sonuçlarını, belirli aralıklarla otomatik olarak hesaplayarak ve saklayarak, daha sonra daha hızlı ve daha verimli bir şekilde kullanılabilir hale getirir.



1. Önceden hesaplanmış sonucu sorgulamak, orijinal ifadeyi çalıştırmaktan çok daha hızlıdır. Recording Rules, karmaşık ifadelerin sonuçlarını önceden hesaplayarak ve saklayarak, bu sonuçlara daha sonra çok daha hızlı erişim sağlar. Bu, sorgu sürelerinin önemli ölçüde azalmasına ve sistem performansının artmasına katkıda bulunur.
2. Recording Rules, aynı ifadeyi sürekli olarak sorgulamaları gereken gösterge panoları (dashboards) için oldukça yararlıdır. Gösterge panoları, veri görselleştirmelerini ve özetlerini sunan araçlardır ve genellikle belirli aralıklarla verileri güncellerler. Eğer aynı ifadeler sürekli olarak yeniden sorgulanmak zorundaysa, önceden hesaplanmış veriler kullanarak bu süreç çok daha hızlı ve verimli hale gelir. Bu sayede, gösterge panolarının performansı iyileştirilir ve kullanıcı deneyimi geliştirilir.

Recording rules oluşturmak ve promethues'a bu oluşturduğumuz kuralı tanımlamak için, prometheus yapılandırma dosyasında, oluşturduğumuz dosyanın konumunu belirtiyoruz.

```yaml
# prometheus.yml
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
    - "rules/recording_rules_1.yml"
```

Ardından  "/etc/promethues/rules" altında recording rules dosyamızı oluşturabiliriz.

Recording Rules için isimlendirme şemasını önemlidir. İyi düşünülmüş ve tutarlı bir isimlendirme şeması, Prometheus'taki verilerin daha kolay yönetilmesine ve anlaşılmasına yardımcı olur.  önerilen isimlendirme şemasının bileşenleri:

1. `level`: Bu, kuralın çıktısındaki metrik ve etiketlerin toplama düzeyini temsil eder. Bu, sorgulanan verilerin toplama düzeyini (örneğin, `sum`, `avg` veya `max`) ve hangi etiketlere göre gruplandırıldığını belirtir.
2. `metric`: Bu, değerlendirmeye alınan metriğin adıdır. İsimlendirmede kullanılacak metriğin adı buraya yazılır.
3. `operations`: Bu, değerlendirmeye alınan metrik üzerinde uygulanan işlemlerin listesidir. İşlemler, en yeni işlemden başlayarak sıralanır.

Önerilen genel form şu şekildedir: `level:metric:operations`

```yaml
# rules/recording_rules_1.yml
groups:
  - name: my_rules_1
    rules:
      - record: job:node_cpu_seconds_total:avg_idle
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))
```

Yukarıdaki Recording Rule, belirtilen süre boyunca (5 dakika) işlemcilerin (CPU) ortalama boşta kalma sürelerini hesaplamak için kullanılır. Her bileşenin açıklaması:

1. `groups`: Prometheus'ta kural gruplarını tanımlar. Bu örnekte, `my_rules_1` adında bir kural grubu oluşturulmuştur.
2. `name`: Kural grubunun adıdır. Bu örnekte, kural grubunun adı `my_rules_1` olarak belirlenmiştir.
3. `rules`: Kural grubunun içindeki kuralları tanımlar. Bu örnekte, sadece bir kural bulunmaktadır.
4. `record`: Hesaplanan değerin saklanacağı zaman serisi adıdır. Bu örnekte, zaman serisi adı `job:node_cpu_seconds_total:avg_idle` olarak belirlenmiştir.
5. `expr`: Kuralın hesaplaması için kullanılan PromQL ifadesidir. Bu örnekte, ifade şu şekildedir:

```scss
avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))
```

1. İlk olarak, `node_cpu_seconds_total{mode="idle"}` metriği, işlemcilerin (CPU) boşta geçirdiği süreleri temsil eden zaman serilerini seçiyoruz.
2. Ardından, `rate()` işlevi bu zaman serisindeki artış oranını hesaplar. Bu, 5 dakikalık bir süre boyunca (`[5m]`) her işlemcinin ne kadar süreyle boşta kaldığını ölçmeye yardımcı olur. `rate()` işlevi, bu süre boyunca süre artış oranını hesaplar.
3. Son olarak, `avg without(cpu)` ifadesi, tüm işlemcilerin boşta kalma sürelerinin ortalamasını hesaplar. `without(cpu)` kısmı, işlemcilere göre gruplamadan ortalamayı almak anlamına gelir. Bu, her bir işin CPU'larının ortalama boşta kalma sürelerini hesaplamaya yardımcı olur.

Tüm bu tanımlamalardan sonra, kuralımızın prometheus'a yansıması için prometheus servisimizi restart ediyoruz.

```bash
systemctl restart prometheus
```

<figure><img src="../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

Arayüzde bulunan "rules" sekmesinde, az önce oluşturduğumuz recording rules gözükmektedir. State OK durumdadır.

&#x20;

<figure><img src="../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

#### Multiple Rules,

```yaml
groups:
  - name: my_rules_1
    rules:
      - record: job:node_cpu_seconds_total:avg_idle
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))

      - record: job:node_cpu_seconds_total:avg_not_idle
        expr: avg without(cpu, mode)(rate(node_cpu_seconds_total{mode!="idle"}[5m]))
```

Yukarıdaki örnekte, ikinci Recording Rule olan `job:node_cpu_seconds_total:avg_not_idle` şu işlemleri gerçekleştiriyor:

1. `node_cpu_seconds_total{mode!="idle"}` ifadesiyle, işlemcilerin (CPU) boşta olmayan modlarında geçirdiği süreleri temsil eden zaman serilerini seçer. Burada `mode!="idle"` şartı, boşta olmayan tüm modları (örneğin, `system`, `user`, `iowait` vb.) içerir.
2. Ardından, `rate()` işlevi bu zaman serisindeki artış oranını hesaplar. Bu, 5 dakikalık bir süre boyunca (`[5m]`) her işlemcinin ne kadar süreyle boşta olmayan modlarda çalıştığını ölçmeye yardımcı olur.
3. Son olarak, `avg without(cpu, mode)` ifadesi, tüm işlemcilerin boşta olmayan sürelerinin ortalamasını hesaplar. `without(cpu, mode)` kısmı, işlemcilere (CPU) ve modlara göre gruplamadan ortalamayı almak anlamına gelir. Bu, her bir işin CPU'larının ortalama boşta olmayan sürelerini hesaplamaya yardımcı olur.

<figure><img src="../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

Eğer dilersek, aynı grup altına yeni sorgu eklemek yerine, mevcut dosyaya yeni bir grup ekleyebiliriz.

```yaml
groups:
  - name: my_rules_1
    rules:
      - record: job:node_cpu_seconds_total:avg_idle
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))

      - record: job:node_cpu_seconds_total:avg_not_idle
        expr: avg without(cpu, mode)(rate(node_cpu_seconds_total{mode!="idle"}[5m]))
        
  - name: my_rules_2
    rules:
      - record: job:node_cpu_seconds_total:avg_idle_new
        expr: avg without(cpu)(rate(node_cpu_seconds_total{mode="idle"}[5m]))
```

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}


1. Uzun vektör aralıkları için kurallar kullanmaktan kaçının: Uzun aralıklı sorgular genellikle yorucu olup, düzenli olarak çalıştırılması performans sorunlarına yol açabilir. Bunun yerine, daha kısa aralıklar kullanarak sistemi zorlamadan verileri analiz edin.
2. Uzun vadeli veri depolama için kurallar kullanın: Aylar veya yıllar boyunca saklanacak metrik veriler için kayıt kurallarını kullanın. Bu sayede, uzun vadeli analiz ve raporlama için önemli verileri hızlı ve verimli bir şekilde sorgulayabilirsiniz.
3. İşlere göre farklı gruplarda kuralları tanımlayın: İşlerin (Jobs) farklı özelliklerine ve ihtiyaçlarına göre kuralları ayıran gruplar oluşturun. Bu, yönetimi ve bakımı kolaylaştırır ve kaynakları daha etkin bir şekilde kullanmanıza olanak tanır.
4. Kayıt kurallarını adlandırırken adlandırma kurallarına uyun: İyi bir adlandırma düzeni kullanarak, kuralların anlaşılması, yönetilmesi ve bakımı daha kolay hale gelir.&#x20;


{% endhint %}

