# 💙 Basics Commands | Linux Basics #3

```bash
# Hangi kullanıcı ile oturum açıldığını gösterir
whoami
# Çıktı: matthew
# Bu örnekte, matthew kullanıcısı ile oturum açıldığını gösterir.

# Kullanıcının UID, GID ve grup üyeliklerini gösterir
id
# Çıktı: uid=1001(matthew) gid=1001(matthew) groups=1001(matthew)
# Bu örnekte, matthew kullanıcısının UID'si 1001, GID'si 1001 ve matthew grubunun bir üyesi olduğunu gösterir.

# Belirtilen kullanıcıya geçiş yapar
su aparna
# Şifreyi girdikten sonra kullanıcı aparna'ya geçer.
# Örnek: aparna kullanıcısına geçiş yapılır ve aparna kullanıcısının şifresi girilmelidir.

# Belirtilen kullanıcı adı ve sunucu adresi ile uzak bir sunucuya SSH üzerinden bağlanmayı sağlar
ssh aparna@192.168.1.2
# 192.168.1.2 IP adresindeki sunucuya aparna kullanıcısı olarak bağlanır.
# Örnek: aparna kullanıcısı olarak 192.168.1.2 IP adresindeki sunucuya bağlanılır. SSH bağlantısı yapıldıktan sonra aparna kullanıcısının şifresi girilmelidir.

```

Bir kullanıcının `sudo` komutunu kullanabilmesi için `sudo` grubuna dahil olması gerekmektedir. Kullanıcının `sudo` yetkilerine sahip olması, `sudo` komutunu kullanarak geçici olarak süper kullanıcı yetkilerine sahip olmasını sağlar. Bu yetkiler, `sudo` grubuna üyelik veya `/etc/sudoers` dosyasındaki uygun yetkilendirme ile verilebilir. Ancak, bu durum dağıtıma ve yapılandırmaya göre değişebilir. Genellikle Ubuntu gibi dağıtımlarda, `sudo` grubuna üye kullanıcılar `sudo` komutunu kullanabilir. Diğer bazı dağıtımlarda ise bu grup `wheel` olabilir.

```bash
# Normal kullanıcı olarak /root dizinindeki dosyaları listelemeye çalışır
ls /root
# Çıktı: 
# ls: cannot open directory /root: Permission denied
# Açıklama: Normal bir kullanıcı olarak /root dizinine erişmeye çalışırken "Permission denied" hatası alırsınız. Bu, /root dizininin yalnızca root veya yetkili kullanıcılar tarafından erişilebilir olduğunu gösterir.

# Süper kullanıcı yetkileri ile /root dizinindeki dosyaları listelemeye çalışır
sudo ls /root
# Çıktı:
# anaconda-ks.cfg  initial-setup-ks.cfg
# Açıklama: `sudo` komutunu kullanarak geçici olarak süper kullanıcı yetkilerine sahip olursunuz ve /root dizinindeki dosyaları listeleyebilirsiniz. `sudo` komutu, belirli işlemleri root yetkileri ile gerçekleştirmek için kullanılır ve genellikle şifre gerektirir.
```



```bash
# Dosya İndirme Komutları

# curl komutu ile dosya indirme
curl http://www.some-site.com/some-file.txt -O
# Açıklama: Belirtilen URL'deki dosyayı indirir ve dosyayı aynı isimle (some-file.txt) kaydeder.

# wget komutu ile dosya indirme
wget http://www.some-site.com/some-file.txt -O some-file.txt
# Açıklama: Belirtilen URL'deki dosyayı indirir ve 'some-file.txt' adıyla kaydeder. '-O' seçeneği, indirilen dosyanın adını belirler.

# İşletim Sistemi Sürümünü Kontrol Etme

# /etc dizininde ismi 'release' kelimesini içeren dosyaları listelemek
ls /etc/*release*
# Açıklama: /etc dizininde ismi 'release' kelimesini içeren tüm dosyaları listelemek için kullanılır.
# Örnek Çıktı:
# /etc/centos-release /etc/os-release /etc/system-release
# /etc/centos-release-upstream /etc/redhat-release /etc/system-release-cpe

# /etc dizininde ismi 'release' kelimesini içeren dosyaların içeriğini görüntülemek
cat /etc/*release*
# Açıklama: /etc dizininde ismi 'release' kelimesini içeren tüm dosyaların içeriğini görüntülemek için kullanılır.
# Bu komut, sistemin işletim sistemi sürümünü ve diğer önemli bilgilerini kontrol etmek için kullanılır.
# Örnek Çıktı:
# CentOS Linux release 7.7.1908 (Core)
# Derived from Red Hat Enterprise Linux 7.7 (Source)
# NAME="CentOS Linux"
# VERSION="7 (Core)"
# ID="centos"
# ID_LIKE="rhel fedora"
# VERSION_ID="7"
# PRETTY_NAME="CentOS Linux 7 (Core)"
# ANSI_COLOR="0;31"
# CPE_NAME="cpe:/o:centos:centos:7"
# HOME_URL="https://www.centos.org/"
# BUG_REPORT_URL="https://bugs.centos.org/"

```

