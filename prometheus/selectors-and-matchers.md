# 🦯 Selectors  & Matchers

PromQL'de "selector" ve "matcher" kavramları, zaman serisi verilerinin filtrelenmesinde kullanılır. Selector, etiket adları ve değerleri kullanarak belirli zaman serilerini seçmek için kullanılan bir dizi etiket eşleştirmesidir. Matcher, belirli bir etiket eşleştirmesini ifade eder.

Örneğin, aşağıdaki PromQL sorgusu bir selector ve iki adet matcher içerir:

```promql
process_cpu_seconds_total{instance='10.90.0.144:9100',job='node_exporter'}
```

\
selector;

```promql
process_cpu_seconds_total
```

Bu bölüm, "process\_cpu\_seconds\_total" zaman serisi için bir selector görevi görür.



matchers;

```promql
{instance='10.90.0.144:9100', job='node_exporter'}
```

Bu bölüm, filtrelemek için kullanılan bir matcher veya etiket eşleştiricisidir. Bu örnekte, "instance" etiketi "10.90.0.144:9100" ve "job" etiketi "node\_exporter" olan zaman serileri seçilir.

### Matcher Types

#### Equality Matcher

PromQL'de Equality Matcher, belirli bir etiketin belirli bir değere eşit olmasını sağlar. "=" veya "==" sembolü, eşitlik karşılaştırması için kullanılır.&#x20;

```promql
process_cpu_seconds_total{instance='10.90.0.144:9100',job='node_exporter'}
```

Yukarıdaki sorguda aşağıdaki gibi Equality Matchers kullanılmıştır:

```yaml
instance='10.90.0.144:9100'
job='node_exporter'
```

Yukarıdaki sorgu, "instance" etiketi "10.90.0.144:9100" ve "job" etiketi "node\_exporter" olan zaman serilerini seçer. "=" sembolü, etiketin belirtilen değere eşit olması gerektiğini belirtir. Bu nedenle, bu iki etiket eşitlik karşılaştırması ile sorgulanmaktadır.

#### Negative Equality Matcher

Belirli bir etiketin belirli bir değere eşit olmamasını sağlar. "!=" sembolü, eşit olmama karşılaştırması için kullanılır.

Örneğin, "http\_requests\_total" zaman serisinde, "method" etiketi "POST" olmayan zaman serilerini seçmek istediğimizi varsayalım. Bunun için aşağıdaki sorgu kullanılabilir:

```promql
http_requests_total{method!="POST"}
```

Bu sorgu, "http\_requests\_total" zaman serisi için "method" etiketi "POST" olmayan tüm zaman serilerini seçer. "!=" sembolü, etiketin belirtilen değere eşit olmaması gerektiğini belirtir.

\
Veya diğer bir örnek,

```promql
process_cpu_seconds_total{job!='node_exporter'}
```

Yukarıdaki sorgu, "process\_cpu\_seconds\_total" zaman serisinde "job" etiketi "node\_exporter" olmayan tüm zaman serilerini seçer. "!=" sembolü, etiketin belirtilen değere eşit olmaması gerektiğini belirtir ve bu şekilde Negative Equality Matcher kullanılmış olur.

<figure><img src="../.gitbook/assets/Screen Shot 2023-03-30 at 12.03.46 AM.png" alt=""><figcaption></figcaption></figure>

#### Regular expression matcher

Regular expression matcher, belirli bir etiketi belirli bir regular expression ile eşleştirmek için kullanılır.&#x20;

Aşağıdaki sorgu, "prometheus\_http\_requests\_total" zaman serisinde, "handler" etiketi "/api" ile başlayan herhangi bir karakter dizisiyle devam eden tüm zaman serilerini seçmek için kullanılır. "=\~" sembolü, etiketin belirtilen regular expression ile eşleşmesi gerektiğini belirtir.

```promql
prometheus_http_requests_total{handler=~"/api.*"}
```

<figure><img src="../.gitbook/assets/Screen Shot 2023-03-30 at 12.14.53 AM.png" alt=""><figcaption></figcaption></figure>

"prometheus\_http\_requests\_total" zaman serisi, HTTP isteklerinin toplam sayısını içeren bir zaman serisidir. Bu sorgu, sadece "/api" ile başlayan istekleri sayacak şekilde filtreleme yapar.&#x20;

Yani, bu regular expression "/api" ile başlayan ve sonrasında herhangi bir karakter dizisiyle devam eden tüm değerleri seçer.

Veya başka bir örneğe geçersek,

```promql
node_disk_io_now{device=~"sd(a|b)"}
```

Bu sorgu, "node\_disk\_io\_now" zaman serisinde, "device" etiketi "sda" veya "sdb" olan tüm zaman serilerini seçmek için kullanılır. "=\~" sembolü, etiketin belirtilen regular expression ile eşleşmesi gerektiğini belirtir.



<figure><img src="../.gitbook/assets/Screen Shot 2023-03-30 at 12.25.45 AM.png" alt=""><figcaption></figcaption></figure>

\
Regular expression kısmı, "sd(a|b)" şeklinde tanımlanmıştır. Burada, "(a|b)" parantez içindeki "|" operatörüyle ayrılmış iki seçenekten herhangi birini temsil eder. Yani, bu regular expression "sda" veya "sdb" değerlerini seçer.

Bu sorgu, disk giriş/çıkış işlemlerinin anlık değerlerini tutan "node\_disk\_io\_now" zaman serisi için, "/dev/sda" veya "/dev/sdb" diski için ayrı ayrı olmak yerine, her iki diski de kapsayan bir sonuç verir.

#### &#x20;Negative regular expression matcher

Negative regular expression matcher, belirtilen değeri içermeyen etiket değerlerini seçmek için kullanılır. Matcher, "=\~!" sembolü kullanılarak tanımlanır.

```promql
node_disk_io_now{device!~"sd(a|b)"}
```

Yukarıdaki sorgu, "node\_disk\_io\_now" zaman serisinde, "device" etiketi "sda" veya "sdb" değerlerini içermeyen tüm zaman serilerini seçmek için kullanılır. "=\~!" sembolü, etiketin belirtilen regular expression ile eşleşmeyen değerleri seçmek için kullanılır.

Regular expression kısmı, "sd(a|b)" şeklinde tanımlanmıştır. Burada, "(a|b)" parantez içindeki "|" operatörüyle ayrılmış iki seçenekten herhangi birini temsil eder. Yani, bu regular expression "sda" veya "sdb" değerlerini içeren etiket değerlerini seçer.

<figure><img src="../.gitbook/assets/Screen Shot 2023-03-30 at 12.35.29 AM.png" alt=""><figcaption></figcaption></figure>

Özetle bu sorgu, disk giriş/çıkış işlemlerinin anlık değerlerini tutan "node\_disk\_io\_now" zaman serisi için, "/dev/sda" veya "/dev/sdb" diskleri hariç, herhangi bir diski kapsayan sonuçlar verir.



Veya başka bir örnek verelim,

```promql
node_disk_read_bytes_total{device!~"sd(a|b)"}[2m]
```

Yukarıdaki sorgu "node\_disk\_read\_bytes\_total" metrik adına sahip tüm zaman serilerini seçerken "device" etiketi "sda" veya "sdb" olanları hariç tutar ve son 2 dakikaya ait verileri getirir.

Özetle, bu sorgu "node\_disk\_read\_bytes\_total" metrik adına sahip tüm zaman serilerini seçer ve "device" etiketi "sda" veya "sdb" olan zaman serileri hariç diğer tüm zaman serileri seçilir. Sonrasında son 2 dakikaya ait verileri getirir.



Bonus Sorgu;

```promql
avg_over_time(windows_logical_disk_free_bytes{volume!~"C:"}[1m])/1024/1024/1024
```

Bu sorgu Windows işletim sistemi üzerindeki C sürücüsü harici tüm disklerin son 1 dakika içinde ortalama boş alanını gigabyte cinsinden verir.

* `windows_logical_disk_free_bytes{volume!~"C:"}`: C sürücüsü haricindeki tüm disklerin boş alanı verir.
* `[1m]`: Bu ifade son 1 dakika içindeki verileri sorgulayacağımızı belirtir.
* `avg_over_time()`: Belirtilen zaman aralığı içindeki değerlerin ortalamasını alır.
* `/1024/1024/1024`: Byte cinsinden verilen disk boyutunu gigabyte cinsine çevirir.



\-----------------------------------------------------------------------------------------------------

```promql
prometheus_http_requests_total{handler=~"/a.*"}[1m]
```

Yukarıdaki sorgu, son 1 dakika içinde "/a" ile başlayan handler'lar için yapılan HTTP isteklerinin toplam sayısını döndürür. `=~` operatorü regular expression matcher'ını temsil eder. \[1m] ifadesi son 1 dakika içindeki verileri sorguladığını belirtir.



```promql
windows_service_state{state!~".*d"}
```

Yukarıdaki sorgu, "state" etiketi "d" ile bitmeyen Windows hizmetlerinin durumunu gösterir. `!~` kullanılarak negatif bir regular expression araması yapılır. Bu sorgunun sonucu, "state" etiketi "d" ile bitmeyen tüm Windows hizmetlerinin durumlarını içerecektir.<br>
