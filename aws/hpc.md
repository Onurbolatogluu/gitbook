# 💪 HPC

* Yüksek kaynaklara sahip bilgisayarla oluşturabildiğimiz sistemlerin adı. Matematiksel işlemler, Veri inceleme ve analiz. Yüksek işlem gücü gerektiren işler için kullanılır.
* Bu sunucuları EC2 sunucu tipleri kısmında, uygulamamızın ihtiyacına göre seçebiliyoruz(Sunucu kurulum ekranında).&#x20;
* Aslında ayrı bir hizmet değil, EC2 içerisinde High Per computing işleri için bir sunucu kuruyoruz.&#x20;
* EFA (Elastic Fabric Adapter) : EC2 üzerinde, bazı sunucularda destekleniyor. AWS tarafından yazılmış özel bir network interface 'dir. Böylelikle HPC sunuculara bu adaptörü takarsak, gecikme minimum seviyelere inmektedir. (Sunucular arası gecikme)
* HPC Paralel Cluster : Bu sistemi kendi lokalimize indirip, AWS üzerinde bu tool yani orchestration sayesinde HPC cluster 'mızı rahat bir şekilde ayağa kaldırabiliyoruz.  Tek bir ortamdan kurup, yönetmemizi sağlıyor.
