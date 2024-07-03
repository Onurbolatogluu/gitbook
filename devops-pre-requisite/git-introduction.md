# 🟪 GIT Introduction

<figure><img src="../.gitbook/assets/1_irvoqLol7t-EPNzZN6CSnA.png" alt=""><figcaption></figcaption></figure>

#### What is GIT?

Git, açık kaynaklı bir dağıtık versiyon kontrol sistemidir. Yazılım geliştirme süreçlerinde, geliştiricilerin kod değişikliklerini izlemelerine ve işbirliği yapmalarına olanak tanır.

#### Install GIT

Git'i kurmak oldukça basittir. İşletim sistemine göre farklı adımlar izlenir:

* **Windows**: [Git for Windows](https://gitforwindows.org/) sitesinden indirilebilir ve standart bir kurulum süreciyle kurulabilir.
* **macOS**: Homebrew kullanarak `brew install git` komutuyla kurulabilir.
* **Linux**: Paket yöneticisi kullanılarak kurulabilir. Örneğin, Debian tabanlı sistemlerde `sudo apt-get install git` komutuyla kurulabilir.

#### GIT Repositories

Git repository (depo), bir projenin tüm dosyalarını ve geçmişini içeren bir dizindir. Bir Git reposu, `git init` komutuyla oluşturulabilir ve bu komut mevcut bir dizini bir Git reposuna dönüştürür.

#### Remote Repositories

Remote repository, internette veya ağ üzerinde bulunan bir Git reposudur. Geliştiriciler, projelerini bu repolara göndererek (push) veya bu depolardan çekerek (pull) işbirliği yapabilirler. Örnek olarak GitHub, GitLab ve Bitbucket gibi platformlar remote repository hizmeti sunar.

#### Clone, Pull & Push

* **Clone**: `git clone <repository-url>` komutuyla, uzak bir depo yerel bilgisayara kopyalanır.
* **Pull**: `git pull` komutuyla, uzak depodaki değişiklikler yerel depoya çekilir ve birleştirilir.
* **Push**: `git push` komutuyla, yerel depodaki değişiklikler uzak depoya gönderilir.

#### GIT vs GITHUB

* **Git**: Versiyon kontrol sistemidir. Komut satırı araçlarıyla kullanılan, dağıtık bir sistemdir.
* **GitHub**: Git repolarını barındıran ve projelerin paylaşılmasını sağlayan bir platformdur. GitHub, Git'in üzerine ek özellikler sunar, örneğin, sorun takibi, proje yönetimi araçları ve wiki.



