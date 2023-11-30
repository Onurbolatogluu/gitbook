# ⚙ Prometheus with HTTP API

Prometheus, HTTP API'yi kullanarak bir dizi işlem gerçekleştirebilirsiniz. İşte Prometheus'un HTTP API'sini kullanarak yapılabilecek bazı işlevler:

1. Metrik Verileri Almak: HTTP API, Prometheus tarafından toplanan metrik verilerine erişim sağlar. Bir GET isteği kullanarak, mevcut metriklerin listesini, belirli bir metriğin değerini veya belirli bir zaman aralığındaki metrik verilerini alabilirsiniz.
2. Metriklerde Filtreleme ve Sorgulama: Prometheus, metrik verileri için etkili bir sorgu diline sahiptir. HTTP API'yi kullanarak metrikler üzerinde filtrelemeler uygulayabilir ve istediğiniz metrik verilerini alabilirsiniz. Örneğin, belirli bir etikete sahip metrikleri alabilir veya belirli bir değeri içeren metrikleri filtreleyebilirsiniz.
3. Alarm ve Uyarıları Yönetmek: Prometheus, metriklerin değerleri üzerinde uyarılar ve alarm durumları oluşturmanıza olanak tanır. HTTP API'yi kullanarak, mevcut uyarıları listeleyebilir, yeni bir uyarı oluşturabilir veya mevcut bir uyarının durumunu değiştirebilirsiniz.
4. Metriklerin Sunucudan Kaldırılması: HTTP API aracılığıyla Prometheus'tan belirli bir metriği veya metrikleri kaldırabilirsiniz. Bu, gereksiz veya eski metrikleri temizlemek için kullanışlı olabilir.
5. Hedefleri ve Konfigürasyonu Yönetmek: Prometheus, hedeflerin ve hedefleri izlemek için kullanılan metriklerin konfigürasyonunu yönetmenize olanak tanır. HTTP API'yi kullanarak hedefleri ekleyebilir, kaldırabilir veya düzenleyebilirsiniz. Ayrıca, scrape\_interval gibi yapılandırma ayarlarını güncelleyebilirsiniz.

Bu, Prometheus'un HTTP API'si ile yapılabilecek bazı temel işlevleri içeren bir örnektir. Prometheus'un kapsamlı bir API'si vardır ve daha pek çok işlem gerçekleştirmenize olanak tanır.&#x20;

#### Örnekler,

```
http://prometheusIP:9090/api/v1/query?query=up
```

\
Bu sorgu, Prometheus HTTP API'sini kullanarak `up` metriği için bir sorgu yapar. `up` metriği, Prometheus tarafından izlenen hedeflerin sağlık durumunu temsil eder. Bu sorgu, tüm hedeflerin mevcut durumlarını almanıza olanak tanır.

Örneğin, http://10.90.0.138:9090/api/v1/query?query=up sorgusunu çağırdığınızda, Prometheus sunucusu üzerinde çalışan tüm hedeflerin sağlık durumunu elde edebilirsiniz. Bu durumda, sunucudaki tüm hedeflerin "up" durumunu kontrol edebilir ve bu durumun 1 (sağlıklı) veya 0 (hedef çalışmıyor) olarak döndüğünü görebilirsiniz.

<figure><img src="../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

#### Diğer Örnekler;

1.  Metriklerin listesini almak:

    ```
    curl http://10.90.0.138:9090/api/v1/label/__name__/values
    ```
2.  Belirli bir metriği sorgulamak:

    ```
    curl http://10.90.0.138:9090/api/v1/query?query=<metrik_adı>
    ```
3.  Metrik değerlerini belirli bir zaman aralığıyla sorgulamak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=<metrik_adı>&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```
4.  Belirli bir etikete sahip metrikleri listelemek:

    ```
    curl http://10.90.0.138:9090/api/v1/series?match[]=<etiket_adı>="<etiket_değeri>"
    ```
5.  Aktif hedeflerin listesini almak:

    ```
    curl http://10.90.0.138:9090/api/v1/targets
    ```
6.  Prometheus'un sağlık durumunu kontrol etmek:

    ```
    curl http://10.90.0.138:9090/api/v1/status/config
    ```
7.  Gözetim hedeflerinin yapılandırmasını almak:

    ```
    curl http://10.90.0.138:9090/api/v1/targets/metadata
    ```
8.  Kuralların listesini almak:

    ```
    curl http://10.90.0.138:9090/api/v1/rules
    ```
9.  Dış işleyicilerin (alertmanager vb.) listesini almak:

    ```
    curl http://10.90.0.138:9090/api/v1/alertmanagers
    ```
10. Grafiklerin listesini almak:

    ```
    curl http://10.90.0.138:9090/api/v1/graphs
    ```
11. Hedefin belirli bir metriği için anlık verileri almak:

    ```
    curl http://10.90.0.138:9090/api/v1/series?match[]={__name__="<metrik_adı>"}&start=<başlangıç_zamanı>&end=<bitiş_zamanı>
    ```
12. Hedefin belirli bir metriği için toplam veri noktalarını almak:

    ```
    curl http://10.90.0.138:9090/api/v1/query?query=count(<metrik_adı>)
    ```
13. Belirli bir zaman dilimi içinde belirli bir metriğin ortalama değerini almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=avg_over_time(<metrik_adı>[<zaman_dilimi>])'
    ```
14. Belirli bir zaman aralığındaki metrik değerlerinin toplamını almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=sum(<metrik_adı>)&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```
15. Belirli bir zaman aralığındaki metrik değerlerinin minimumunu almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=min(<metrik_adı>)&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```
16. Belirli bir zaman aralığındaki metrik değerlerinin maksimumunu almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=max(<metrik_adı>)&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```
17. Belirli bir zaman aralığındaki metrik değerlerinin standart sapmasını almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=stddev(<metrik_adı>)&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```
18. Belirli bir zaman aralığındaki metrik değerlerinin yüzdesini almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=quantile(<yüzde>, <metrik_adı>)&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```
19. Belirli bir zaman aralığındaki metrik değerlerinin artışını almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=increase(<metrik_adı>)&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```
20. Belirli bir zaman aralığındaki metrik değerlerinin hızını almak:

    ```
    curl 'http://10.90.0.138:9090/api/v1/query_range?query=rate(<metrik_adı>)&start=<başlangıç_zamanı>&end=<bitiş_zamanı>&step=<zaman_adımı>'
    ```



