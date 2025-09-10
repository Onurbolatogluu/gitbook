---
icon: s
---

# Basics of SCM

<figure><img src="../.gitbook/assets/Screenshot 2025-09-10 at 16.40.46.png" alt=""><figcaption></figcaption></figure>

#### SCM Nedir ve Neden İhtiyacımız Var?

SCM, "Source Code Management" yani "Kaynak Kod Yönetimi" sisteminin kısaltmasıdır. Diğer bir popüler adı da VCS'dir ("Version Control System" - Versiyon Kontrol Sistemi).

En basit haliyle SCM'i, kodunuz için süper güçlere sahip bir "zaman makinesi" veya "merkezi bir kütüphane" olarak düşünebilirsin.

Eğer bir SCM sistemi olmasaydı tam bir kaos yaşanırdı:

* Yazılımcı 2, kendi değişikliklerini kaydettiğinde Yazılımcı 1'in yaptığı işin üzerine yazar ve o değişiklikler kaybolur.
* Yazılımcı 3'ün yaptığı hata yüzünden program çalışmaz hale geldiğinde, hatanın kimden kaynaklandığını ve programın en son ne zaman düzgün çalıştığını bulmak imkansız hale gelir.

İşte SCM sistemleri tam olarak bu kaosu önlemek için var. Kod üzerinde yapılan her bir değişikliği, kimin yaptığını ve ne zaman yaptığını kaydeder.

#### SCM'in Sağladığı Faydalar Nelerdir?

Paylaştığın görseller bu faydaları çok güzel özetliyor. Gel üzerinden geçelim:

1. Sorunsuz İşbirliği (Seamless Collaboration): Birden fazla yazılımcı aynı proje üzerinde birbirinin işini ezmeden güvenle çalışabilir.
2. Kod Gözden Geçirme (Code Review): Bir yazılımcının yaptığı değişiklikler ana koda eklenmeden önce diğer ekip üyeleri tarafından incelenebilir ve onaylanabilir. Bu, kod kalitesini artırır.
3. Çakışma Çözümü (Conflict Resolution): İki yazılımcı aynı satırı değiştirdiğinde, SCM sistemi bunu fark eder ve bu çakışmayı çözmeleri için onlara yardımcı olur.
4. Kolay Paylaşım (Effortless Sharing): Kod, herkesin erişebileceği merkezi bir yerde durur.
5. Geçmişi Görmek (Unveiling the Past): Projenin geçmişindeki herhangi bir ana gidip o tarihte kodun nasıl göründüğünü inceleyebilirsin. Kimin hangi değişikliği neden yaptığını görmek çok kolaydır.
6. Geri Almaya Hazır (Rollback Ready): Yapılan bir değişiklik büyük bir hataya yol açtıysa, tek bir komutla projenin sorunsuz çalıştığı bir önceki versiyonuna anında geri dönebilirsin. Bu, geliştiriciler için bir "güvenlik ağı" görevi görür.

#### Git, GitHub, GitLab Arasındaki Fark Nedir?

Bu konu genellikle yeni başlayanların kafasını karıştırır, o yüzden netleştirelim. Paylaştığın görseldeki logolar bu konuyu anlatıyor.

* Git: SCM işini yapan teknolojinin kendisidir. Tıpkı bir arabanın motoru gibi, arka planda çalışan asıl sistem budur. Genellikle komut satırından kullanılır.
* GitHub, GitLab, Bitbucket vb.: Bu platformlar, Git teknolojisini kullanan web tabanlı servislerdir. Arabanın motoru Git ise, GitHub ve GitLab arabanın kendisidir. Yani Git motorunu alıp üzerine kullanışlı bir web arayüzü, kullanıcı yönetimi, kod inceleme araçları gibi birçok ek özellik eklerler. Kodunuzu bu platformlarda barındırırsınız (host edersiniz).

Jenkins için bunun anlamı: Jenkins, projenin kodunu çekmek için bu platformlardan birine (örneğin GitHub'a) bağlanır ve arka planda Git komutlarını kullanarak kodu kendi sunucusuna indirir.



