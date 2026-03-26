# ⏲️ Azure Blob Backup and Recovery

### Soft Delete,

<figure><img src="../.gitbook/assets/16169913746061548e330dd.png" alt=""><figcaption></figcaption></figure>

* Bu özellik, blob ve container gibi nesneleri korumak için tasarlanmıştır. Nesneler silinse bile, belirlediğiniz bir süre içerisinde geri getirilebilirler.
* Bir nesne silindiğinde, belirlenen süre boyunca (1 ila 365 gün arası) geri dönüş için kullanılabilir kalır.
* Bu süreçte, silinen nesneler görünmez olur, ancak tamamen silinmeden önce kullanıcılar tarafından kurtarılabilir.

### **Versioning,**

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Azure'da her bir blob için yapılan değişiklikler otomatik olarak yeni bir sürüm olarak kaydedilir.
* Bir blob güncellendiğinde veya üzerine yazıldığında, eski versiyonlar saklanır ve bu eski sürümlere istenildiği zaman erişilebilir.
* Versioning özelliği veri yönetimi ve kurtarma süreçlerini kolaylaştırır ve eski sürümlere dönüş yapmayı mümkün kılar.

***



Özetle, Blob Storage'da Versioning özelliği, bir blob üzerinde yapılan her değişikliği (yeniden yazma, güncelleme veya silme gibi) yeni ve benzersiz bir sürüm olarak saklar.&#x20;

* Bir blob'a herhangi bir değişiklik uygulandığında, bu değişiklik otomatik olarak yeni bir sürüm oluşturur ve bu sürüm benzersiz bir sürüm kimliği (version ID) ile etiketlenir. Bu kimlik, o sürümü diğerlerinden ayırt etmek için kullanılır.
* Yapılan her değişiklik yeni bir sürüm oluşturduğu için, kullanıcılar blob'un önceki durumlarına erişebilir ve gerektiğinde önceki sürümlere dönebilirler. Bu, verinin zaman içindeki evrimini gözlemlemek ve gerektiğinde eski verilere dönmek için çok faydalıdır.
* Sadece en yeni sürüm varsayılan olarak erişilebilir durumdadır. Önceki sürümler listelendiğinde veya özel olarak istendiğinde erişilebilir olurlar. Misal,

```
İstek şablonu : https://<your-storage-account>.blob.core.windows.net/<your-container>/<your-blob>?versionId=<your-version-id>&<your-SAS-token>

Örnek : https://example.blob.core.windows.net/demo/soft-delete-historical-version.png?versionid=2024-04-19T12:02:50.3098983Z&sp=rl&st=2024-04-19T12:16:10Z&se=2024-04-26T20:16:10Z&sv=2022-11-02&sr=c&sig=L3PQgkvck02dGgYEguHL4VFxFo3r56hrfbOFcIhTY1s%3D
```



{% hint style="info" %}
Eğer bir blob üzerinde `Versioning` özelliği açıksa ve `Soft Delete` kapalı olsa bile, blob silinse dahi o blob'a ait önceki sürümler saklanmaya devam eder. Bu sürümler, silme işleminden bağımsız olarak korunur ve istediğiniz zaman bu sürümlere erişebilirsiniz.\
\
`Versioning` özelliği, blobların her bir değişikliğini yeni bir sürüm olarak saklar. Bu sayede, Soft Delete özelliğinin açık olmadığı durumda bile herhangi bir blob silinse bile, onun önceki sürümleri tamamen silinmediği sürece veriye erişim mümkün olur. Ancak, bu özellik depolama kullanımını artırabileceği için, eski sürümleri düzenli olarak gözden geçirmek ve gereksiz olanları silmek gerekir.
{% endhint %}



### PITR for blobs,

<figure><img src="../.gitbook/assets/restore-all-containers-portal.png" alt=""><figcaption></figcaption></figure>

Point-in-Time Restore (PITR) özelliği, Azure Blob Storage'daki verileri, belli bir geçmiş noktasına göre otomatik olarak geri yükleyebilmeniz için bir veri kurtarma çözümüdür. Bu özellik, özellikle bloblar yanlışlıkla silindiğinde, zarar gördüğünde veya üzerinde istenmeyen değişiklik yapıldığında kullanılır.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. **T-3 Gün:**
   * Ekranda sol tarafta gösterilen ilk durum, belirli bir zamandaki (T-3 gün) storage account'unuzun ve içerdiği container'ların durumunu temsil ediyor. Burada her container, içinde bir dizi blob (dosya veya veri parçası) barındırıyor.
2. **Blobların Silinmesi (T-2 Gün):**
   * Zamanda ilerlendiğinde, ortada gösterilen durumda bazı blob'ların silindiğini görüyoruz. Bu, yanlışlıkla silme veya veri bozulması gibi durumları temsil edebilir.
3. **Blobların Geri Yüklenmesi (T):**
   * Sağ tarafta ise, silinen blob'ların "Point-in-Time Restore" kullanılarak geri yüklendiği gösteriliyor. Burada, silinmeden önceki duruma (T-3 gün) geri dönmüş olarak, blob'lar tekrar yerlerinde ve erişilebilir durumda.

#### Nasıl Çalışır?

1. Azure, otomatik olarak blob depolama hesabınızda snapshotlar alır. Bu snapshotlar, belirlenen bir saklama politikasına göre tutulur.
2. Kullanıcılar, verilerini geri yüklemek istedikleri özel bir tarih ve saati seçebilirler. Bu, silinen veya bozulan verileri, seçilen zaman noktasına kadar olan en son durumuna geri yükleme imkanı verir.
3. Seçilen zaman noktasına ait snapshot üzerinden, bloblar eski haline geri yüklenir. Bu işlem, genellikle Azure portalı üzerinden veya Azure CLI aracılığıyla gerçekleştirilir.



{% embed url="https://learn.microsoft.com/en-us/azure/storage/blobs/point-in-time-restore-manage?tabs=portal" %}
