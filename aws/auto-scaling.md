# 🤝 Auto Scaling

Sunucularımızda problem olduğunda bunun önüne geçebiliriz.

Launch Config : Sunucu kaynakları ve özelliklerini belirttiğimiz bir dosya yaratıp, sunucular otomatik oluşturulurken bu dosya üzerinden baz alınarak kurulacaktır.

Auto Scaling > Launch conf > create > Sunucu kurarken yapılan adımları giriyoruz > AMI seçiyoruz > Sunucu tipi > Launch ismi giriyoruz > IAM rolü seçiyoruz > Disk seçiyoruz. > Security group seçiyoruz > Create  > İlk launch configuration oluşturuldu.

Autoscaling group > Create > Launch configuration > yukarıda yarattığımız Launch configuration dosyasını seçiyoruz >grup ismini giriyoruz > grup size =auto scaling ile kaç tane sunucu yaratmak istiyoruz bunu belirtmeliyiz. > VPC  'mizin tüm subnetlerini seçiyoruz.

#### Advanced Details

Load Balancing : Eğer bu sunucurı Load balancer arkasına alıp yük dağıtımı yapmak istiyorsak burayı işaretleyip, oluşturduğumuz target grubunu seçiyoruz.

Health Checks : Sağlık kontrolünü ELB mi, yoksa EC2 üzerinden mi yapacağız. Bunu seçiyoruz. ELB üzerinden sağlık kontrolü yapıyorsak ELB seçeneğini seçiyoruz. Geri kalanlar default kalabilir.

#### Configure Scaling Policies

Keep this group at its initial size : Bunu seçersek autoscaling yapılandırmasında kaç adet sunucu oluşturmasını istediysek, onları oluşturur. Bu sunucuları silersek tekrar oluşturur. Birinde bir problem olursa silip yenisi oluşturacak.

Use scaling policies to adjust the capacity of this group : Bu seçeneği seçiyoruz çünkü ihtiyaç halinde sunucuları yaratıp auto-scaling yapmış olacağız.

Scale beetwen : 2 > Burada her türlü 2 sunucu bulunabilir şekilde ayarla demiş oluyoruz.\
And 4 instance : Aşağıda gireceğimiz ayara göre aşağıdaki belirttiğimiz durum oluşursa 4 sunucuya kadar sunucu ekle. Aşağıdaki durum sona ererse yaptığın değişiklikleri geri al demektedir.

#### Scale Group Size

Metric type : Application LB request count per target : LBye gelen istek sayısına göre ayarla.

Avarage Cpu util : Cpu değerine göre ayarla.

Target value : 30 > cpu 30un üstüne çıkarsa kuralı uygula, Ortamdaki sunucuların ortalama cpu değerlerini baz alır.

#### Configure Notifications

Auto scaling servisi hakkında bilgilendirme maili almak istiyorsak, buradan bildirimleri açabiliriz.

Tag ekleyebiliriz. Next diyip autoscaling grubunu oluşturabiliriz.

Mevcut sunucuları seçip, actions kısmından autoscaling grubuna dahil edebiliriz. Fakat autoscaling grubunda 2 sunucu istediğimiz söylemiştik. Fakat autoscaling ayarlarında yeni sunucularımızı dahil ettiğimizde bu ayar değişecektir. Kaç sunucuyu sonradan eklediysek o kadar sayı artacaktır. Bunu değiştirip 2ye alabiliriz. ve autoscaling fazla sunucuları cpu %30un altındaysa sunucularımızın fazlalarını kapatacaktır.&#x20;

{% embed url="https://www.youtube.com/watch?ab_channel=Simplilearn&v=4EOaAkY4pNE" %}
