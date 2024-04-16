# 🅰️ Azure Stream Analytics

<figure><img src="../.gitbook/assets/stream-analytics-e2e-pipeline (1).png" alt=""><figcaption></figcaption></figure>

Azure Stream Analytics, gerçek zamanlı veri akışlarını analiz ve işlemek için tasarlanmış, tamamen yönetilen bir hizmettir. Cihazlar, sensörler, web siteleri, sosyal medya ve diğer uygulamalar gibi kaynaklardan gelen çoklu veri akışları üzerinde gerçek zamanlı analizler gerçekleştirmenize olanak tanır.

**Azure Stream Analytics'in temel çalışma mekanizması:**

#### 1. Veri Alımı (Ingestion):

Azure Stream Analytics, bir veri alım hattı aracılığıyla sürekli veri akışlarını kabul eder. Bu veriler IoT cihazlarından, uygulamalardan, web sitelerinden, loglardan ve daha birçok kaynaktan gelebilir. Akış verisi, genellikle Azure IoT Hub veya Azure Event Hubs gibi hizmetler üzerinden toplanır ve Stream Analytics'e aktarılır.

#### 2. Veri İşleme (Processing):

Alınan veri akışı, Azure Stream Analytics tarafından gerçek zamanlı olarak işlenir. İşleme sırasında aşağıdaki gibi çeşitli işlemler gerçekleştirilebilir:

* İstenmeyen veya önemsiz verilerin filtrelenmesi.
* Belirli bir zaman aralığına göre verilerin toplanması ve özetlenmesi (örneğin, her dakika veya her saat için toplam sayım).
* İşlenen verilerin, başka depolanan referans verilerle birleştirilerek zenginleştirilmesi.
* Belirli kalıpların dışında kalan verilerin tespit edilmesi.
* Önceden eğitilmiş Azure Machine Learning modelleri ile gerçek zamanlı veriler üzerinde skorlama yapılması.

#### 3. Veri Teslimatı (Delivery):

İşlenen veriler, çeşitli çıktı noktalarına yönlendirilebilir:

* **Uyarılar ve Eylemler**: Azure Functions, Logic Apps veya diğer hizmetler aracılığıyla eylemler tetiklenebilir veya ilgili taraflara uyarılar gönderilebilir.
* **Görselleştirme**: Power BI gibi araçlara yönlendirilerek, veriler gerçek zamanlı görselleştirmelerle analiz edilebilir.
* **Saklama ve Analiz**: Daha fazla analiz ve sorgulama için Azure Synapse Analytics, Azure Data Lake veya diğer veri depolama çözümlerine kaydedilebilir.

***

#### Azure Stream Analytics'in Avantajları:

1. Azure Stream Analytics, donanım veya alt yapı üzerinde endişe etmeden kullanabileceğiniz tamamen yönetilen bir hizmettir.
2. Kaynaklar iş ihtiyaçlarına göre ölçeklenebilir ve hizmet, kullanılan bellek ve CPU'ya göre Stream Units (SU) cinsinden faturalandırılır. Bakım maliyeti yoktur.
3. Hizmet, gelen veriyi paralel olarak birden çok node üzerinde işleyebilir ve saniyede milyonlarca olayı işleyebilir. Hızlı sonuç elde etmek için tasarlanmıştır.
4. Gelen ve giden tüm veri trafiği TLS 1.2 ile şifrelenir. Hizmet, verileri bellekte işler ve veriler disk üzerinde saklanmaz.&#x20;

#### Azure Stream Analytics'in Kullanıldığı Durumlar:

1. Gerçek zamanlı IoT sensör veri akışları analiz edilebilir.
2. Web logları ve tıklama akışı, e-ticaret analitiklerinde müşterilere yeni ürün önermek için kullanılabilir.
3. Uydu, mobil cihazlar ve sensörlerden gelen gerçek zamanlı veri akışları toplanabilir ve hava durumu, konum kullanımı gibi analizler yapılabilir.
4. Endüstriyel makinelerin ve ekipmanların durumu, bakım ve performans analizleri için izlenebilir.
5. POS makinelerinden elde edilen veriler, sahtekarlık tespiti için analiz edilebilir ve işlemler, konum ve değere göre kredi kartı işlemleri izlenebilir.

{% embed url="https://learn.microsoft.com/en-us/azure/stream-analytics/" %}
