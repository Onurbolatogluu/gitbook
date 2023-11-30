---
description: Elastic Load Balancer
---

# ⚖ ELB

ELB kullanmak için, Load Balancers > create > Application Load Balancer > İsim giriyoruz.

internet-facing : Dış dünyadan gelen trafiği sunucularımıza aktarır.\
internal : Sadece iç ağ'da (VPC) load balancing işlemini yapar.\
\
Dış ağ üzerinden trafiği iç ağımıza dağıtmak için internet-facing seçeneğini seçiyoruz. Dualstack seçeneğini seçersek IPv6 desteğini de aktif etmiş oluruz. Protocol olarak HTTP ve HTTPS (web site için) seçiyoruz. Availability zone seçeğini LB istekleri hangi VPC 'lere dağıtacaksa bunu seçiyoruz. Security gruplarımızı dahil edebiliriz.\
Routing seçeneği ile gelen trafiği içeride nasıl dağıtacağımızın ayarlarını yapabiliyoruz. İlk önce target grup oluşturmalıyız. Target type kısmında instance kısmına ec2 sunucuları seçebiliyoruz. Eğer istersek IP olarak da ekleyebiliriz. Sunucular farklı bir cloud ortamında olsa dahi Load Balancing yapabiliriz. Lambda seçeneği ile lambda servislerini balancing edebiliriz.

Protocol : Dış dünyadan aldığımız trafiği içeride sunucuların hangi portuna, hangi protokol ile trafiği göndereceğimizi seçiyoruz.

Health Checks : Trafiğin gönderildiği web sunucuların düzgün çalışıp çalışmadığını kontrol edebiliriz. Düzgün çalışıyorsa trafiği o sunucuya gönderebiliriz. Sorunluysa eğer, trafiği o sunucuya göndermeyecektir.

Path : Sunucular da sağlık kontrolü yaparken istekler hangi path'e gitmeliyse bunu seçiyoruz. Siteye erişimi direkt IP üzerinden yapabiliyorsak /index.html yazıp gitmiyorsak, Default olarak / kalabilir.

Advanced Health Check settings : Health check için kontrol kuralları uygulayabiliriz.

Port : Trafiğin yönlendirileceği port.

Health Threshold :  Kaç defa istek gönderilip, sunucunun sağlıklı olduğuna karar verilecekse buradan belirtebiliyoruz.

Unhealty Threshold : Gönderilen isteklerden kaç tane, fail, hata alırsak sunucunun problemli olduğuna karar vereceksek bunu burada belirtmemiz gerekmektedir.

Timeout : Bağlantı kurmaya çalışırken, kaç saniye de bağlantı kurmasına izin verilecekse, burada belirtmemiz gerekmektedir.

Interval : Her deneme arasında kaç saniye beklememiz gerekiyorsa burada belirtmeliyiz.

Success Codes : Arkada bulunan web sunucular hangi http kodunu döndürüyorsa sağlıklı kabul edeceğimizi burada belirtiyoruz. 200 durum kodunu yazabiliriz.

Register Targets : İstekleri dağıtacağımız sunucular ( add to registered ) seçiyoruz.  Ve LB hazır durumdadır.
