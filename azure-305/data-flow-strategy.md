# 📼 Data flow strategy

Data flow strategy, verilerin bir organizasyon veya sistem içinde nasıl toplandığı, işlendiği, saklandığı ve analiz edildiğini tanımlayan bir yaklaşımdır. Verilerin yaşam döngüsünü ve verilerin işlenme şeklini planlamak için kullanılır ve genellikle verilerin işlenme sıklığı ve ihtiyaç duyulan işlem türüne göre sıcak, ılık ve soğuk veri yollarını içerir:

<figure><img src="../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

#### Veri Akışı Stratejileri:

1. **Hot Data Path**:
   * Değişken veri gereksinimleri ve gerçek zamanlı işlem gerektiren senaryolarda kullanılır.
   * Anında reaksiyon veya eylem gerektiren veri akışları için idealdir.
   * Örneğin, gerçek zamanlı izleme veya olay tepki sistemleri buraya örnek verilebilir.
2. **Warm Data Path:**
   * Daha az sık erişilen ancak hala hızlı erişim gerektiren veriler için kullanılır.
   * Küçük ölçekli analitik işlemler ve yığın işleme işleri için kullanılır.
   * Kısa süreli veri saklama veya hızlı erişim gerektiren senaryolarda tercih edilir.
3. **Cold Data Path**:
   * Nadiren erişilen ve uzun vadeli analizler için saklanan veriler için kullanılır.
   * Uygun maliyetli saklama ve yasal uyumluluk gibi gereksinimler için idealdir.
   * Tarihsel veri analizi veya arşivleme bu kategoride yer alır.
