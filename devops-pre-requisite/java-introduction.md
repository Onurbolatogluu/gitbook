# 🟨 Java introduction

Java, 1995 yılında Sun Microsystems (şu anda Oracle Corporation tarafından sahiplenilmiştir) tarafından geliştirilen yüksek seviyeli, sınıf tabanlı, nesne yönelimli bir programlama dilidir.&#x20;

* Java, ücretsiz olarak sunulmaktadır. Geliştiriciler, Java Development Kit (JDK) ve diğer araçları ücretsiz olarak indirip kullanabilirler.
* Java, açık kaynaklı bir dildir. Bu, geliştiricilerin Java'nın kaynak koduna erişebileceği ve kendi ihtiyaçlarına göre değiştirebileceği anlamına gelir. Açık kaynak topluluğu, Java'nın gelişimine aktif olarak katkıda bulunur.
* Java, dünya çapında geniş bir topluluğa sahiptir. Bu topluluk, Java'nın sürekli gelişimine ve desteklenmesine katkıda bulunur. Çeşitli forumlar, kullanıcı grupları ve etkinlikler, Java geliştiricilerinin bilgi ve deneyim paylaşmasını sağlar.

#### Java'nın çeşitli sürümleri,

* Java 13: 2019
* Java 12: 2019
* Java 11: 2018
* Java 10: 2018
* Java 9: 2017
* Java 8: 2014
* Java 7: 2011
* Java 6: 2006
* Java 5: 2004

Bu sürümler, Java'nın sürekli olarak güncellenen ve geliştirilen bir dil olduğunu gösterir. Her yeni sürüm, dildeki iyileştirmeleri, yeni özellikleri ve performans artışlarını içerir. Java'nın sürekli güncellenmesi, onu güvenilir ve sürdürülebilir bir dil yapar.

Java, platform bağımsızlığı (bir kez yaz, her yerde çalıştır), güvenlik ve taşınabilirlik gibi özellikleriyle bilinir. Bu özellikler, Java'nın geniş bir yelpazede uygulamalar geliştirmek için kullanılmasını sağlar, örneğin web uygulamaları, mobil uygulamalar, masaüstü uygulamaları ve büyük ölçekli kurumsal sistemler.



Java Kurulumu hakkında => [https://hostman.com/tutorials/installing-java-on-ubuntu-22-04/](https://hostman.com/tutorials/installing-java-on-ubuntu-22-04/)

### Java Development Kit (JDK) Nedir?

JDK, Java uygulamaları geliştirip çalıştırmak için gereken araçları içeren bir yazılım geliştirme kitidir. Üç ana bileşeni vardır: **Develop (Geliştirme)**, **Build (Derleme)** ve **Run (Çalıştırma)**.

**Geliştirme (Develop):**

* **jdb**: Java Debugger, Java programlarını hata ayıklamak için kullanılır.
* **javadoc**: Java kodu için otomatik olarak API dokümantasyonu oluşturur.

**Derleme (Build):**

* **javac**: Java Compiler, Java kaynak kodunu (source code) derleyerek bytecode'a dönüştürür.
* **jar**: Java Archive, bir veya daha fazla dosyayı tek bir dosyada paketler ve sıkıştırır.

**Çalıştırma (Run):**

* **JRE (Java Runtime Environment)**: Java uygulamalarını çalıştırmak için gereken runtime ortamıdır.
* **java**: Java uygulamalarını çalıştırmak için kullanılan komuttur.

#### Ek Araçlar:

JDK'nın içinde birçok ek araç ve komut bulunmaktadır. Örneğin:

<figure><img src="../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

Bu araçlar, Java uygulamaları geliştirirken, derlerken ve çalıştırırken ihtiyaç duyulabilecek çeşitli işlevleri yerine getirir.

<figure><img src="../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Java 9'da yapılan büyük değişikliklerden biri, JDK (Java Development Kit) ve JRE (Java Runtime Environment) ayrımının kaldırılmasıdır. Önceki sürümlerde, JDK ve JRE ayrı paketler olarak sunuluyordu ve kullanıcılar ihtiyacına göre bunlardan birini indirip kurabiliyordu. Ancak Java 9'dan itibaren, JDK indirildiğinde JRE de bu paketin içinde yer alıyor. Bu değişiklik, kullanıcıların daha az dosya indirmesini sağlamakta ve kurulum sürecini basitleştirmektedir.
{% endhint %}

***

### Build,

Java'da "compile" süreci, Java kodlarının çalıştırılabilir hale getirilmesi sürecidir.

<figure><img src="../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

**1 - Kaynak Kodun Geliştirilmesi (Develop Source Code)**

* Java programcısı, Java programını yazdığı bir kaynak kod dosyası (örneğin `MyClass.java`) oluşturur.

```java
public class MyClass {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

**2 - Derleme (Compile)**

* Kaynak kod dosyası (`MyClass.java`), Java derleyicisi (`javac`) kullanılarak derlenir. Bu işlem, kaynak kodun makine tarafından çalıştırılabilir bayt kodlarına dönüştürülmesi sürecidir. Derleme sonucunda `.class` uzantılı bir dosya (örneğin `MyClass.class`) oluşturulur.

**3 - Çalıştırma (Run)**

* Oluşturulan bayt kodu dosyası (`MyClass.class`), Java Runtime Environment (JRE) kullanılarak çalıştırılır. Bu işlem, bayt kodlarının Java Sanal Makinesi (JVM) tarafından çalıştırılmasıyla gerçekleştirilir.

```sh
java MyClass
```

* Bu komut, `MyClass.class` dosyasını çalıştırarak "Hello World" mesajını ekrana yazdırır.

Özetle, Java'da bir program yazıldığında, önce kaynak kod dosyası oluşturulur, ardından bu dosya derlenir ve son olarak derlenen dosya çalıştırılarak programın çıktısı alınır. Bu süreç sayesinde, Java kodları farklı platformlarda çalıştırılabilir hale gelir.



### Java Virtual Machine (JVM),

JVM, Java programlarının çalıştırılmasını sağlar. JVM'in temel çalışma adımları:



**Kaynak Kod (Source Code)**:

* İnsan tarafından okunabilir Java kaynak kodu, örneğin `MyClass.java`.

**Ara Kod (Byte Code)**:

* `javac` komutu ile derlenen bytecode, bir ara kod formatıdır ve `.class` uzantılı dosyalarda saklanır. Bu bytecode, JVM tarafından yorumlanarak çalıştırılabilir.

```sh
javac MyClass.java
```

**JVM ve Makine Kodu**:

* JVM, bytecode'u alır ve çalıştırmak için makine koduna çevirir. Bu, JVM'in çalıştığı platforma bağımlıdır ve bytecode'un platformdan bağımsız çalışmasını sağlar.&#x20;

```sh
java MyClass
```



### Package,

Java'da paketleme, bir veya birden fazla sınıf dosyasını (class files) ve bunlara bağlı diğer dosyaları bir araya getirerek, dağıtımı ve kullanımını kolaylaştıran bir dosya oluşturma sürecidir. Bu süreçte, Java Archive (JAR) veya Web Archive (WAR) dosyaları kullanılır.

JAR dosyası, birçok sınıf dosyasını, bağımlılıklarını, resim ve diğer kaynak dosyalarını tek bir arşivde birleştirir. JAR dosyaları, genellikle uygulamaların daha kolay dağıtılmasını ve çalıştırılmasını sağlar. Örneğin, tüm uygulamayı tek bir JAR dosyasına paketleyerek, kullanıcıların uygulamayı çalıştırmak için sadece bu dosyayı kullanması yeterli olur.

WAR dosyası, web uygulamaları için kullanılan bir arşiv formatıdır. Java Servlet, JSP ve ilgili sınıf dosyaları, kaynaklar ve konfigürasyon dosyaları bir WAR dosyasında birleştirilir ve bir Java web sunucusunda dağıtılır.

**META-INF/MANIFEST.MF**

Bu dosya, JAR dosyasının meta bilgilerinin bulunduğu yerdir. Manifest dosyasında, JAR dosyasının versiyonu, yaratıcı bilgileri ve ana sınıf (main class) gibi bilgiler yer alır. Örneğin:

```vbnet
Manifest-Version: 1.0
Created-By: 1.8.0_242 (Private Build)
Main-Class: MyClass
```

#### Paketleme Örneği

1. Gerekli Sınıf Dosyalarının Derlenmesi:

```sh
javac MyClass.java Service1.java Service2.java
```

2. JAR Dosyasının Oluşturulması:

```sh
jar cf MyApp.jar MyClass.class Service1.class Service2.class
```

* Bu komut, `MyApp.jar` adında bir JAR dosyası oluşturur ve içine `MyClass.class`, `Service1.class`, `Service2.class` dosyalarını ekler.

3. JAR Dosyasının Çalıştırılması:

```bash
java -jar MyApp.jar
```

Bu komut, `MyApp.jar` dosyasını çalıştırır ve içindeki ana sınıfın (main class) `main` metodunu başlatır.



* **JAR Dosyası**: Bir JAR dosyası oluşturmak için `jar` komutunu kullanırız. `cf` parametresi, yeni bir arşiv oluşturulacağını (`c`) ve dosya isimlerinin takip ettiğini (`f`) belirtir.
* **Manifest Dosyası**: Eğer `MyApp.jar` içindeki `META-INF/MANIFEST.MF` dosyasını inceleyecek olursak, burada JAR dosyasının versiyon bilgisi, yaratıcı bilgileri ve ana sınıf bilgisi bulunur.
* **Çalıştırma**: `java -jar MyApp.jar` komutu, JAR dosyasının içindeki `Main-Class` olarak belirtilen sınıfın `main` metodunu çalıştırır.



Java projelerinde, çeşitli sınıfları ve kaynakları tek bir dosyada paketlemek için Java Archive (JAR) dosyaları kullanılır. JAR dosyaları, tüm sınıf dosyalarını ve diğer kaynakları içeren sıkıştırılmış bir arşivdir. Bu dosyalar, projeyi dağıtmayı ve çalıştırmayı kolaylaştırır. JAR dosyası oluşturduktan sonra, ana sınıfın main metodunu çalıştırmak için `java -jar` komutu kullanılır. Örneğin:

```shell
java -jar MyApp.jar
```

Ancak, JAR dosyasının içindeki belirli bir sınıfı çalıştırmak için, JAR dosyasını classpath'e ekleyerek `java -cp` komutunu kullanabilirsiniz. Bu yöntemle, ana sınıf olarak belirtilmemiş olan bir sınıfı doğrudan çalıştırabilirsiniz. Örneğin:

```shell
java -cp MyApp.jar Service2
```

Bu komut, MyApp.jar dosyasının içinde bulunan ve main metoduna sahip Service2 sınıfını çalıştırır. Örneğin, `java -cp MyApp.jar Service2` komutu, `Service2.class` dosyasını çalıştırır, eğer bu sınıf bir `main` metoduna sahipse.

```java
public class Service2 {
    public static void main(String[] args) {
        System.out.println("Service2 is running");
    }
}
```

Özetle, `java -jar` komutu JAR dosyasının ana sınıfını çalıştırırken, `java -cp` komutu ile JAR dosyasındaki belirli bir sınıfı çalıştırabilirsiniz. Bu, projedeki farklı sınıfları çalıştırmayı ve test etmeyi kolaylaştırır.



***

### Document,

"javadoc" komutu, Java sınıflarından belgeler oluşturmak için kullanılır.

```bash
javadoc -d doc MyClass.java
```

* **`javadoc`**: Bu komut, Java kaynak kodu dosyalarından API belgeleri oluşturur.
* **`-d doc`**: Bu seçenek, oluşturulan belgelerin hangi dizine kaydedileceğini belirtir. Burada, belgeler "doc" adlı bir dizine kaydedilecek.
* **`MyClass.java`**: Bu, belgelerin oluşturulacağı Java kaynak kodu dosyasıdır.

Bu komut çalıştırıldığında, "MyClass.java" dosyasındaki sınıf ve metodlara ait açıklamalar, HTML formatında belgeler olarak "doc" dizinine oluşturulur. Bu belgeler, geliştiricilere kodun nasıl çalıştığını ve nasıl kullanılacağını anlamalarına yardımcı olur.

***

Java programlama diliyle bir yazılım geliştirme sürecinin ana adımları:

1. **Develop (Geliştirme)**:
   * Bu aşamada, geliştiriciler kaynak kodu yazar. Örneğin, `MyClass.java` dosyası.
2. **Compile (Derleme)**:
   * `javac MyClass.java` komutuyla, Java kaynak kodu derlenir. Derleyici (`javac`), yazılan Java kodunu makine diline daha yakın olan bayt koduna (bytecode) dönüştürür ve bu kod `.class` uzantılı dosyalarda saklanır. Örneğin, `MyClass.class`.
3. **Package (Paketleme)**:
   * `jar cf MyClass.jar MyClass.class` komutuyla, derlenmiş `.class` dosyası veya dosyaları ve gerekli diğer dosyalar tek bir arşiv dosyasında (JAR dosyası) paketlenir. JAR (Java Archive) dosyaları, Java uygulamalarının dağıtımı ve çalıştırılması için kullanılır. Bu aşamada tüm gerekli sınıf dosyaları ve bağımlılıklar bu arşivde toplanır.
4. **Document (Belgelendirme)**:
   * `javadoc MyClass.java` komutuyla, yazılan Java kodu için dokümantasyon oluşturulur. `javadoc`, Java kodu içinde yer alan yorumları kullanarak, HTML formatında dokümanlar oluşturur. Bu dokümanlar, sınıfların, metodların ve alanların ne işe yaradığını açıklar.

Özetle,

* **Develop**: Kod yazma aşaması.
* **Compile**: Yazılan kodun bayt koduna çevrilmesi.
* **Package**: Bayt kodunun bir arşiv dosyasına (JAR) paketlenmesi.
* **Document**: Kodun dokümantasyonunun oluşturulması.

***

**Build Tools**: Build araçları, yazılım projelerinin otomatikleştirilmiş derleme, test, paketleme ve dağıtım süreçlerini yönetmek için kullanılır. Üç popüler build aracı şunlardır:

1. **Maven**: Maven, özellikle Java projeleri için yaygın olarak kullanılan bir build aracıdır. Proje yapılandırma, bağımlılık yönetimi ve proje yönetimi için standart bir yaklaşım sağlar. XML tabanlı POM (Project Object Model) dosyaları kullanılarak yapılandırılır.
2. **Gradle**: Gradle, Maven ve Ant gibi build araçlarının özelliklerini birleştiren modern bir build aracıdır. Groovy veya Kotlin DSL (Domain-Specific Language) kullanılarak yapılandırılır ve yüksek performanslı build süreçleri sağlar.
3. **ANT**: Apache Ant, XML tabanlı yapılandırma dosyaları kullanarak build süreçlerini otomatikleştiren eski bir build aracıdır. Esneklik sağlar ancak daha yeni araçlar kadar yüksek düzeyde bağımlılık yönetimi sunmaz.

