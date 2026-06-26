---
icon: message
---

# Benefits of Having More vCenter Users

Bir sistem yöneticisinin yaşayabileceği en stresli anlardan birini hayal edin: Mesai saatinin tam ortasında ortamdaki kimlik doğrulama mekanizması çöküyor, kullanıcılar sistemlere giremiyor ve alarmlar çalmaya başlıyor. Sorunu araştırdığınızda, her şeyin kalbi olan "Domain Controller 2012" sanal makinesinin aniden kapatıldığını (Powered Off) fark ediyorsunuz.

Sistem çöktüğü için oluşan hasar bir yana, yöneticinin aklında yankılanan asıl soru şudur: "Bunu kim yaptı?"

Eğer sanallaştırma altyapınızda görev yapan tüm yöneticiler, operatörler ve uzmanlar sisteme `administrator@vsphere.local` veya `root` gibi ortak bir süper kullanıcı hesabıyla giriyorsa, bu sorunun cevabını asla bulamazsınız. Bu makalede, VMware vCenter ortamlarında bireysel kullanıcı hesapları açmanın operasyonel faydalarını, log izleme stratejilerini ve "Hesap Verilebilirlik" kavramını inceleyeceğiz.

**1. Ortak Hesap Kullanımının Yarattığı Operasyonel Körlük**

Küçük veya yeni kurulan BT altyapılarında, kolaylık olması adına tek bir yönetici parolasının tüm ekibe dağıtılması sık karşılaşılan bir hatadır. Ancak sistemler büyüdükçe bu "kolaylık", yerini tam bir operasyonel körlüğe bırakır.

Yanlışlıkla silinen bir makine, hatalı yapılandırılan bir ağ kartı veya aniden kapatılan kritik bir sunucu sonrasında vCenter kayıtlarına baktığınızda, eylemi gerçekleştiren kişi olarak sadece "Administrator" ibaresini görürsünüz. Ortada bir insan hatası (veya kötü niyetli bir eylem) vardır, ancak eylemin kime ait olduğu karanlıktadır. Bireysel kullanıcı hesapları (Örn: `jack@vsphere.local` veya Active Directory üzerinden gelen `ahmet.yilmaz`), bu anonimlik perdesini ortadan kaldırır.

**2. Kriz Anında Olay Yeri İncelemesi: "Monitor > Tasks" Mimarisi**

Bir felaket anında veya beklenmeyen bir değişiklikte, vCenter arayüzü aslında size harika bir dijital ayak izi sunar. Ancak bu izleri doğru menülerde aramak gerekir.

Çoğu zaman alt kısımdaki "Recent Tasks" paneline bakılır, ancak oturum kapatılıp açıldığında bu panel temizlenebilir. Gerçek ve kalıcı loglara ulaşmak için doğrudan vCenter veya ilgili sanal makine üzerinden `Monitor > Tasks and Events > Tasks` sekmesine gidilmelidir.

Bu ekranda sistemdeki tüm hareketlerin kronolojik bir dökümü bulunur. Örneğimizdeki Domain Controller kesintisine geri dönersek, bu listede "Power Off Virtual Machine" (Sanal Makineyi Kapat) eylemini bulduğunuzda, satırın detaylarında sihirli bir sütun dikkatinizi çekecektir: Initiator (Başlatan). Eğer mimarinizi doğru kurguladıysanız, bu sütunda anonim bir "Administrator" yazısı yerine eylemi bizzat gerçekleştiren kullanıcının kimliğini (Örn: `jack@vsphere.local`) ve işlemin yapıldığı milisaniyesine kadar zaman damgasını görürsünüz. Sorunun kaynağı anında tespit edilmiş olur.

**3. İleri Seviye Mimari: Olay Günlüklerini (Log) Merkezileştirmek**

vCenter arayüzünden log okumak işin sadece temel düzeyidir. Gelişmiş tehditleri algılamak veya kurumsal uyumluluğu (Compliance) sağlamak için sadece kimin hangi makineyi kapattığını bilmek yetmez; aynı zamanda başarısız kimlik doğrulama denemelerini (Authentication Failures) veya anomali yaratan hareketleri de proaktif olarak izlemek gerekir.

> 💡 İpucu: Merkezi Güvenlik ve Log İzleme (SIEM) Entegrasyonu
>
> Gerçek bir kurumsal altyapıda, vCenter logları kendi arayüzüne hapsedilmez. Profesyonel sistem yöneticileri, Active Directory sunucuları ile vCenter altyapısını eşzamanlı olarak izleyebilmek için ortamdaki tüm olay günlüklerini (Syslog) Wazuh gibi kapsamlı güvenlik ve log izleme mimarilerine yönlendirirler.
>
> Agent tabanlı veya Syslog yönlendirmesiyle kurulan bu yapılar sayesinde; Windows, Active Directory ve vCenter sunucularınızdaki tüm kimlik doğrulama hataları ve kritik "Power Off" işlemleri tek bir merkezden izlenebilir. Hatta kritik bir makine kapatıldığında (örneğin birisi Domain Controller'ı kapattığında), merkezi log altyapınızın Telegram gibi kanallar üzerinden size anında otomatik uyarı mesajları göndermesini sağlayabilirsiniz.

Özetle; Bireysel kullanıcı hesapları tanımlamak ve bu hesaplara sadece işlerini yapabilecekleri kadar yetki (RBAC) vermek, sadece bir yönetim prosedürü değil; aynı zamanda veri merkezinizin en temel savunma hattıdır. Doğru yetkilendirme ve merkezi log izleme stratejileriyle, altyapınızda hiçbir olay karanlıkta kalmaz.
