# Linux I/O redirection

Linux komut satırında **I/O yönlendirme (input/output redirection)**, komutların standart giriş ve çıkış akışlarını yeniden yönlendirme işlemidir. Varsayılan olarak her program üç özel akış kullanır: **stdin**, **stdout** ve **stderr**.&#x20;

### stdin, stdout, stderr Kavramları ve File Descriptor (FD) Yapısı

Linux ve Unix sistemlerde her **proces** (çalışan program), varsayılan olarak üç standart akış ile başlar:

* **stdin (Standard Input)** – Programın girdi aldığı akıştır (varsayılan olarak klavye veya terminal girişi). File descriptor numarası **0** olarak tanımlıdır.
* **stdout (Standard Output)** – Programın normal çıktıları yazdığı akıştır (varsayılan olarak terminal ekranı). File descriptor numarası **1**'dir.
* **stderr (Standard Error)** – Programın hata mesajlarını yazdığı akıştır (varsayılan olarak terminal ekranı). File descriptor numarası **2**'dir.

Her akış aslında işletim sistemi düzeyinde bir **file descriptor (FD)** ile temsil edilir. File descriptor, açık bir dosyayı veya I/O kanalını tanımlayan bir tamsayı kimliktir. Örneğin, yukarıda belirtildiği gibi `stdin` için FD 0, `stdout` için FD 1 ve `stderr` için FD 2 kullanılır. Bir program çalıştığında bu FD'ler genellikle terminale bağlıdır (yani `stdin` terminal girdisini, `stdout` ve `stderr` terminal çıktısını temsil eder). Programlar gerektiğinde başka dosyalar açtığında 3, 4, 5 gibi sonraki FD numaraları kullanılır, ancak biz bu rehberde temel olarak 0, 1, 2 numaralı standart akışlar ve bunların yönlendirilmesine odaklanacağız.

### Temel Redirect Operatörleri: `>`, `>>`, `<`

Komut satırında çıktı ve girdi yönlendirmek için özel operatörler kullanılır. Temel yönlendirme operatörleri şunlardır:

* **`>` (büyüktür işareti)** – **Stdout yönlendirme (çıktıyı dosyaya yazma)** operatörüdür. Bir komutun normal çıktısını bir dosyaya yönlendirir. Örneğin, `ls > liste.txt` komutu, `ls` çıktısını _liste.txt_ adlı dosyaya yazar. Dosya mevcut değilse oluşturulur; mevcutsa dosyanın önceki içeriği silinir ve üzerine yazılır. Not: `>` operatörü aslında `1>` anlamına gelir; yani eğer bir rakam belirtilmezse shell bunu FD 1 (stdout) için yönlendirme olarak varsayar.
* **`>>` (çift büyüktür işareti)** – **Append (ekleyerek yazma)** operatörüdür. Stdout çıktısını bir dosyanın sonuna ekler. Örneğin, `date >> gunluk.log` komutu, `date` çıktısını _gunluk.log_ dosyasının sonuna ekleyecektir. Dosya yoksa oluşturulur; varsa mevcut içerik korunarak yeni çıktı en sona eklenir. Bu operatör, tek `>` operatörünün aksine dosyanın içeriğini sıfırlamaz (üzerine yazmaz), yalnızca ekleme yapar.
* **`<` (küçüktür işareti)** – **Stdin yönlendirme (girdiyi dosyadan okuma)** operatörüdür. Bir komutun girdi kaynağını klavye yerine bir dosya olarak ayarlar. Örneğin, `wc -l < belge.txt` komutu, `wc` (word count) programına `belge.txt` dosyasını standart girdi olarak verir; böylece `wc` komutu dosyadaki satır sayısını ekrana yazacaktır (klavyeden girdi beklemek yerine doğrudan dosyadan okur). Benzer şekilde, herhangi bir komutu `< dosya` şeklinde çalıştırmak, o komutun `stdin` akışını belirtilen dosyaya bağlar. Shell, sayı belirtilmediğinde `<` operatörünü FD 0 (stdin) için uygular.

Bu temel operatörlerle, komutların girişini ve çıkışını esnek bir şekilde yönlendirebiliriz. Örneğin, bir dosyanın içeriğini başka bir dosyaya kopyalamak için `cat kaynak.txt > hedef.txt` kullanılabilir. Ya da bir programın çıktısını hem ekrana hem dosyaya göndermek için `tee` komutunu kullanmadan önce temel `>` operatörü ile çıktı dosyaya yönlendirilebilir.

### stderr Redirect İşlemleri: `2>`, `2>>`, `2>&1`

Varsayılan durumda, bir komut hem normal çıktısını (**stdout**) hem de hata çıktısını (**stderr**) aynı ekrana (terminal) yazar. Ancak stdout ve stderr ayrı akışlar olduğu için, istersek bunları ayrı ayrı yönlendirebiliriz:

* **`2>`** – Hata çıktısını (stderr) bir dosyaya yönlendirir (üzerine yazarak). `2` rakamı, FD 2 (stderr) anlamına gelir. Örneğin, `grep aranan dosya.txt 2> hata.txt` komutu, `grep` çalışırken oluşabilecek hata mesajlarını _hata.txt_ dosyasına yönlendirecektir. Dosya mevcutsa eski içerik silinir ve yerine yeni hata mesajı yazılır (tıpkı `>` operatöründe olduğu gibi). Normal çıktılar (eşleşen satırlar) ise yönlendirilmediği için terminalde görüntülenmeye devam eder. Benzer şekilde `komut 2> /dev/null` yazarak bir komutun tüm hata mesajlarını "yok edebiliriz" (_/dev/null_ özel bir "çöp" dosyasıdır, bkz. Pro İpuçları). **Dikkat:** `2 > dosya` şeklinde araya boşluk koymak farklı anlama gelir; `2>dosya` şeklinde bitişik yazılmalıdır.
* **`2>>`** – Hata çıktısını bir dosyaya **ekleyerek** yönlendirir. Yani `2>>` operatörü, `2>` ile aynı şekilde stderr akışını hedef dosyaya yollar fakat mevcut dosya içeriğini korur, yeni hata mesajlarını dosya sonuna ekler. Örneğin, `ls /root 2>> hatalar.log` komutunu her çalıştırdığınızda, _hatalar.log_ dosyasına erişim hatası mesajı (varsa) eklenir ve önceki kayıtlar korunur.
* **`2>&1`** – Stderr akışını, stdout akışının şu anki hedefine yönlendirir. Buradaki `&1`, "file descriptor 1'ı kastediyorum" demenin yoludur; yani `2>&1`, "FD 2'yi, FD 1 nereye gidiyorsa oraya yönlendir" anlamına gelir. Tek başına kullanıldığında bir çıktı üretmez, ancak diğer yönlendirme operatörleriyle kombinasyon halinde sıkça kullanılır (bir sonraki bölümde ayrıntılı ele alınacak). Örneğin `komut > output.txt 2>&1` şeklinde kullanıldığında, önce stdout _output.txt_ dosyasına yönlendirilir, ardından `2>&1` ile stderr de stdout ile aynı yere (artık _output.txt_ dosyasına) yönlendirilir – böylece her iki akışın çıktıları da aynı dosyada toplanır. **Önemli:** `2>&1` yazarken `&` karakterini unutmayın; eğer `2>1` yazarsanız shell, "stderr çıktısını adlı '1' olan bir dosyaya yönlendir" şeklinde yorumlayacaktır ki bu genelde istenmeyen bir durumdur. Doğru kullanım `2>&1` şeklindedir.

Yukarıdaki operatörler sayesinde, bir komutun hata mesajlarını ayrı bir dosyada tutabilir veya tamamen yok sayabiliriz. Örneğin, `find / -name "*.conf" 2> hatalar.txt` komutu, root olarak çalışmadığınızda oluşacak "İzin engellendi" türü hata mesajlarını _hatalar.txt_'ye kaydederken, normal çıktıyı (bulunan dosyaları) ekrana yazmaya devam edecektir. Benzer şekilde, `find / -name "*.conf" 2> /dev/null` yaparak istenmeyen tüm hata mesajlarını _dev/null_ çöp dosyasına atabilirsiniz.

### stdout ve stderr için Birleşik Redirect Örnekleri: `> file 2>&1`, `&> file`, `&>> file`

Çoğu zaman, bir komutun **hem standart çıktısını hem de hata çıktısını birlikte** aynı yere yönlendirmek isteriz. Bunu yapmanın birkaç farklı yöntemi vardır:

* **`komut > file 2>&1`** – Bu kullanım, stdout çıktısını _file_ adlı dosyaya yönlendirir **ve** `2>&1` sayesinde stderr çıktısını da aynı dosyaya gönderir. Yani komutun tüm çıktıları (başarılı çıktı + hata mesajları) tek bir dosyada toplanmış olur. Örneğin: `ls -l /etc > liste.txt 2>&1` komutu, `/etc` içeriğini listeler ve çıktı ile birlikte olası hataları da _liste.txt_ dosyasına yazar. Bu yöntem tüm POSIX uyumlu shell'lerde çalışır. Ekleme yapmak istersek `>>` ile kombinasyon da mümkündür (ör. `komut >> file 2>&1` mevcut dosyaya ekleme yapacaktır).
* **`komut &> file`** – Bu, Bash shell'ine özel bir **kısayol** olup, stdout ve stderr akışlarının her ikisini birden _file_ dosyasına yönlendirir. Yani `&>` operatörü, yukarıdaki `> file 2>&1` kombinasyonunun tek bir operatörle yazılmış halidir. Örneğin `gcc program.c &> derle.log` komutu derleme sırasında çıkan tüm mesajları (hata dahil) _derle.log_ dosyasına tek seferde yönlendirir. (Not: `&>` kullanımı Bash 4+ içindir ve POSIX standardında yoktur; en geniş uyumluluk için scriptlerde klasik `> ... 2>&1` yöntemini kullanmak tercih edilir.)
* **`komut &>> file`** – Bu da Bash'e özel bir kısayol olup, stdout ve stderr çıktılarının **her ikisini birden append modunda** (dosya sonuna ekleyerek) _file_ dosyasına yönlendirir. Yani `&>>` operatörü, `>> file 2>&1` işlemini tek adımda yapar. Örneğin `./script.sh &>> log.txt` komutu, _script.sh_ çalışmasının tüm çıktısını _log.txt_ dosyasının sonuna ekleyecektir (dosya yoksa oluşturulur).

Yukarıdaki birleşik yönlendirme yöntemleri, sık kullanılan pratiklerdir. Örneğin, bir cron job içinde `mysqldump` komutunu çalıştırıp hem normal çıktıyı hem hataları bir log dosyasına yönlendirmek için `mysqldump ... &> backup.log` şeklinde kullanılabilir. Yine de, eğer Bash dışında bir shell kullanma ihtimaliniz varsa `&>` yerine her zaman `> file 2>&1` şeklindeki taşınabilir sözdizimini kullanmak güvenli olacaktır.

### Redirect Sıralamasının Önemi (Örneklerle)

Yönlendirme operatörlerinin **yazılış sırası**, özellikle stderr'in stdout'a yönlendirilmesi durumunda çok kritiktir. Shell, komutu çalıştırmadan önce komut satırındaki yönlendirmeleri soldan sağa değerlendirir. Dolayısıyla stdout ve stderr'i aynı dosyaya yönlendirmek için doğru sıra kullanılmazsa beklenmedik sonuçlar doğabilir.

**Doğru Sıra –** Örneğin `ls > çıktı.txt 2>&1` komutunu ele alalım. Burada shell önce `> çıktı.txt` kısmını işler, bu adımda stdout akışı _çıktı.txt_ dosyasına bağlanır. Ardından `2>&1` kısmı işlenir, bu da stderr akışını "FD 1 nereye gidiyorsa oraya" bağlar. Bu noktada FD 1 _çıktı.txt_ dosyasına yönlenmiş olduğundan, FD 2 de aynı dosyaya yönlenecektir. Sonuç olarak hem normal listleme çıktıları hem de hata mesajları _çıktı.txt_ dosyasında toplanır.

**Yanlış Sıra –** Şimdi `ls 2>&1 > çıktı.txt` komutunu düşünelim. Shell soldan sağa giderken ilk olarak `2>&1` ifadesini yorumlar. Bu an itibarıyla stdout (FD 1) henüz yeniden yönlendirilmemiş olduğundan, FD 1 hala terminale işaret etmektedir. Dolayısıyla `2>&1` işlemi, stderr'i mevcut stdout'un yönüne (terminale) yönlendirecektir. Sonrasında `> çıktı.txt` kısmı geldiğinde stdout _çıktı.txt_'ye yönlenir, fakat stderr çoktan terminale sabitlenmiştir. Sonuç: stdout çıktıları dosyaya giderken, stderr çıktıları terminalde kalır. Yani komutun hata mesajları dosyaya yazılmaz, ekranda görünür. Bu örnek, operatör sıralamasının önemini net bir şekilde göstermektedir.

Özetle, **her iki akışı aynı hedefe yönlendirmek istediğinizde** `> file 2>&1` biçimini kullanmalısınız. `2>&1` ifadesi, stdout yönlendirmesinden **sonra** yazılmalıdır. Tersi durumda stderr, stdout yönlendirilmeden önce kopyalandığı için yönlenmez.

> 🔹 **Not:** Stdout ve stderr farklı hedeflere (örneğin farklı dosyalara) yönlendiriliyorsa, sıralama hayati değildir – hangi sırayla yazıldığı çıktının bütünlüğünü etkilemez; shell her ikisini de uygulayacaktır. Örneğin `komut >out.txt 2>err.txt` ve `komut 2>err.txt >out.txt` aynı sonucu üretir (stdout _out.txt_ dosyasına, stderr _err.txt_ dosyasına). Ancak stderr'i stdout ile **birleştirirken** (aynı yere yönlendirirken) yukarıdaki sıralama kuralına uymak gerekir.
>
>

### Here Document (`<<`) ve Here String (`<<<`) Kullanımı

Belirli durumlarda komutların standard input'una bir dosyadan değil, komut satırı içinde tanımladığımız **metni** veya **çok satırlı bir metin bloğunu** beslemek isteriz. Bu amaçla Bash ve benzeri shell'ler iki özel yönlendirme sunar: **here document** ve **here string**.

*   **Here Document (`<<`)** – Bir here-doc, komuta birden fazla satırlık girdi beslemenin özel bir yoludur. `<<` operatöründen sonra bir _delimiter_ (sınırlandırıcı) tanımlanır ve bu delimiter ne ise, o satıra kadar yazılan tüm satırlar komuta girdi olarak verilir. Söz dizimi genellikle şu şekildedir:

    ```shell
    komut <<DELIM
    çok satırlı
    girdi metni...
    DELIM
    ```

    Yukarıdaki yapıda, `DELIM` yerine çoğunlukla `EOF` veya `END` gibi bir anahtar kelime kullanılır (ama aslında herhangi bir kelime olabilir). Komut çalıştığında, `<<EOF` gördüğünde shell, _EOF_ satırı gelene dek yazılan tüm satırları alır ve ilgili komuta standart girdi olarak sunar. **Örnek:** Aşağıdaki kullanım, `cat` komutuna üç satırlık bir metni girdi olarak verip aynı çıktıyı oluşturur:

    ```shell
    cat <<EOF
    Birinci satır
    İkinci satır
    Üçüncü satır
    EOF
    ```

    Bu komut çalıştığında `cat`, EOF ile işaretlenmiş burada yazılı üç satırı standart girdi olarak alır ve ekrana yazdırır. Here-doc özellikle _çok satırlı yapılandırma_ veya _script_ parçacıklarını bir komuta beslemek için kullanışlıdır. Örneğin bir sunucuya SSH ile bağlanıp birkaç komutu uzaktan çalıştırmak için:

    ```shell
    ssh user@host <<EOF
    ls /var/log
    df -h /
    EOF
    ```

    Bu sayede `ssh` komutu, EOF ile sınırlanmış bloğu uzak sunucuda yürütülecek komutlar olarak alır. Here-doc içerisinde değişken kullanımı, delimiter'ın tırnaklı veya tırnaksız olmasına göre genişletilebilir; delimiter eğer tırnak içine alınmazsa içindeki `$VAR` gibi ifadeler yerine değerleri konur, ancak delimiter'ı örneğin `<<'EOF'` şeklinde tek tırnakla yazarsanız burada yazdığınız metin _çıplak_ olarak (değişkenleri değerlendirmeden) iletilir. Bu detaylar ileri seviye olsa da, temel olarak here document ile bir komuta etkileşimli çok satırlı giriş verebileceğinizi bilmek önemlidir.
* **Here String (`<<<`)** – Here-string, here-doc yapısının tek satırlı (dizgisel) versiyonudur. Söz dizimi `komut <<< "metin"` şeklindedir. Yani `<<<` operatöründen sonra gelen bir string (genellikle tırnak içinde) komuta **tek bir satırlık girdi** olarak verilir. Shell bu stringin sonuna bir newline (satırsonu) ekleyerek komutun stdin'ine aktarır. **Örnek:** `bc <<< "5+5"` komutu, _5+5_ ifadesini `bc` (Basic Calculator) programına girdi olarak verip hesaplamasını sağlar. Benzer şekilde `grep "ariza" <<< "$LOG_VARS"`, bir değişken içeriğini grep komutuna arama girdisi olarak göndermeye yarar. Here-string kullanımı, bir metni echo ile pipe etmeye (`echo "metin" | komut`) göre daha kısa bir sözdizimi sağlar. Unutmayın ki here-string de Bash'e özgüdür.

Here document ve here string, özellikle shell script yazarken kullanıcıdan girdi bekleyen komutları otomatikleştirmek için çok işe yarar. Örneğin bir **here-doc**, `ftp` veya `mysql` gibi interaktif oturum açan programlara komut listesi sağlamakta kullanılabilir. Here-string ise tek seferlik basit girdiler (mesela bir parolanın script içinde bir komuta verilmesi gibi) için pratik bir yol sunar.

### Pro İpuçları ve Sık Yapılan Hatalar

Son olarak, I/O yönlendirme konusunda işinize yarayacak bazı ipuçları ve yeni başlayanların düştüğü yaygın hatalar:

* **Çıktıyı /dev/null ile Yok Etme:** İstenmeyen çıktıları tamamen gözardı etmek için çıktıyı _/dev/null_ adlı özel dosyaya yönlendirebilirsiniz. Örneğin `komut > /dev/null 2>&1` yazmak, ilgili komutun hem stdout hem stderr çıktısını çöpe atacak, hiçbir şey göstermeyecektir. _/dev/null_, genellikle "bit bucket" ya da kara delik olarak anılır; yazdığınız her şey kaybolur. Bu yöntem, özellikle bir komutun çıktılarıyla ilgilenmiyorsanız veya bir servis kontrol edilirken hata mesajlarını baskılamak istiyorsanız kullanışlıdır.
* **`noclobber` Opsiyonu ile Dosya Ezilmeyi Önleme:** Bash shell'inde `set -o noclobber` komutunu etkinleştirerek `>` operatörünün var olan dosyaları ezmesini önleyebilirsiniz. Bu özellik açıkken, `>` ile yönlendirme yapmaya çalıştığınız dosya zaten varsa shell hata verir. Bu durumda gerçekten üzerine yazmak isterseniz `>|` operatörünü kullanmanız gerekir. Yeni başlayanlar için bu opsiyon, kazara önemli bir dosyanın üzerine yazmayı engellemek açısından yararlı olabilir.
* **Yönlendirme Sırasında Boşluk Hatası:** Yönlendirme operatörlerini kullanırken doğru sözdizimine dikkat edin. Özellikle FD numarası ile `>` arasında boşluk bırakmayın. Örneğin `2> hata.txt` doğru kullanım iken `2 > hata.txt` yanlış anlamlara gelebilir (shell bu durumda `2` adında bir komut çalıştırmaya ve stdout'unu _hata.txt_'ye yönlendirmeye çalışabilir). Benzer şekilde `&>` operatöründeki `&` işareti bitişik olmalıdır.
* **Bash'e Özgü Kısayollar ve Taşınabilirlik:** `&>` ve `&>>` gibi birleşik yönlendirme kısayollarının Bash ve Zsh gibi gelişmiş shell'lerde çalıştığını, ancak daha eski Bourne shell veya dash gibi POSIX shell'lerde desteklenmediğini unutmayın. Eğer yazdığınız script'in farklı sistemlerde `/bin/sh` altında çalışmasını bekliyorsanız, bu kısayollar yerine daima klasik sözdizimini (`> file 2>&1` gibi) tercih edin. Script'inizin başına `#!/bin/bash` _shebang_ koymak, Bash'e özgü özellikleri kullanacağınızı belirtmenin bir yoludur.
* **Pipe (`|`) ile Hata Akışını Yönlendirme:** Normalde pipe operatörü (`|`), bir komutun **sadece stdout** çıktısını bir sonraki komutun stdin'ine bağlar. Eğer bir komutun stderr çıktısını da pipelene dahil etmek istiyorsanız, bunu açıkça belirtmelisiniz. Bir yöntem, pipe öncesi `2>&1` yazarak stderr'i de stdout'a katmaktır: `komut1 2>&1 | komut2`. Alternatif olarak Bash'de bunun için özel bir kısayol vardır: **`|&`** operatörü. Örneğin `grep -r "aranan" . |& less` komutu, grep'in hem normal çıktısını hem hata çıktısını (ör. "izin yok" hatalarını) _less_ üzerinden sayfalayacaktır. Bu kısayol, Bash 4 ve üstünde `2>&1 |` yerine kullanılabilir ve okunabilirliği artırır.
* **Aynı Dosyayı Hem Girdi Hem Çıktı Olarak Kullanma:** Yeni başlayanların yaptığı hatalardan biri, aynı dosyayı bir komuta hem girdi hem çıktı olarak yönlendirmektir. Örneğin `sort < veriler.txt > veriler.txt` gibi bir komut, beklediğiniz gibi çalışmaz; çünkü shell önce `> veriler.txt` işlemini yapıp dosyayı boşaltır, sonra `sort` komutu artık boşalmış dosyayı okumaya çalışır. Bu nedenle bir dosyayı kendisine yine çıktıyla değiştirecek işlemlerde ya **geçici bir dosya** kullanın, ya da `sort veriler.txt -o veriler.txt` gibi komutun kendi sağladığı imkân varsa onu tercih edin. Aksi halde veri kaybına uğrayabilirsiniz.
* **Heredoc Delimiter Hataları:** Here document kullanırken, bitiş delimiter sözcüğünü satırda tek başına ve başta kullandığınızdan emin olun. Eğer `EOF` gibi bir delimiter tanımladıysanız, bu kelimeyi bitiş satırında hiçbir ekstra boşluk veya karakter olmadan yazmalısınız; aksi takdirde shell, here-doc'un bitmediğini düşünerek girdi beklemeye devam edecektir. Ayrıca here-doc içinde değişken değerlendirmelerini kontrol etmek için delimiter'ı tırnaklı/tırnaksız kullanma konusuna dikkat edin (örn. `<<EOF` vs `<<"EOF"` farkı).

### Her Yönlendirme Tipi için Örnekler ve Açıklamalar (Tablo)

Aşağıdaki tabloda yaygın I/O yönlendirme operatörleri, örnek kullanımları ve kısa açıklamaları özetlenmiştir:

| Yönlendirme Tipi         | Örnek Kullanım                                                                                                                          | Açıklama                                                                                                                                                      |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `>` (stdout overwrite)   | `komut > dosya.txt`                                                                                                                     | Stdout çıktısını **dosya.txt** adlı dosyaya yönlendirir. Dosya varsa içeriğini siler, yoksa oluşturur.                                                        |
| `>>` (stdout append)     | `komut >> dosya.txt`                                                                                                                    | Stdout çıktısını **dosya.txt** sonuna ekler. Dosya yoksa oluşturulur, mevcut içerik korunur.                                                                  |
| `<` (stdin from file)    | `komut < girdi.txt`                                                                                                                     | Komutun stdin (girdi akışı) kaynağını **girdi.txt** dosyası yapar. Yani klavye yerine dosyadan okur.                                                          |
| `2>` (stderr overwrite)  | `komut 2> hata.log`                                                                                                                     | Stderr (hata) çıktısını **hata.log** dosyasına yönlendirir. Dosya varsa üzerine yazar (mevcut içerik silinir).                                                |
| `2>>` (stderr append)    | `komut 2>> hata.log`                                                                                                                    | Stderr çıktısını **hata.log** sonuna ekler. Dosya yoksa oluşturulur, mevcut içerik korunur.                                                                   |
| `2>&1` (combine streams) | `komut > dosya 2>&1`                                                                                                                    | Stderr akışını, stdout'un yönlendiği **aynı yere** gönderir. Bu örnekte her iki çıktı da **dosya**'ya gider.                                                  |
| `&>` (both to file)      | `komut &> tüm çıktı.log`                                                                                                                | **Bash** kısayolu: Stdout ve stderr çıktılarının her ikisini **tüm çıktı.log** dosyasına yönlendirir.                                                         |
| `&>>` (both append file) | `komut &>> tüm çıktı.log`                                                                                                               | **Bash** kısayolu: Stdout ve stderr çıktılarının her ikisini **tüm çıktı.log** dosyasına **ekler** (append).                                                  |
| `<<` (here document)     | <p><em>Bakınız:</em><br><code>komut &#x3C;&#x3C;EOF</code><br><code>çok satırlı</code><br><code>metin...</code><br><code>EOF</code></p> | Komuta bir _here document_ bloğu sağlar: `EOF` (veya belirlenen delimiter) satırına kadar yazılan çok satırlı metin, komutun stdin girdisi olarak kullanılır. |
| `<<<` (here string)      | `komut <<< "metin"`                                                                                                                     | Bir _here string_ kullanır: **"metin"** dizgesini tek seferlik stdin girdisi olarak komuta verir (stringin sonuna otomatik newline eklenir).                  |
