# 📌 ECS - EKS

## EKS - Elastic Container Services

![](../.gitbook/assets/product-page-diagram\_Amazon-ECS@2x.0d872eb6fb782ddc733a27d2bb9db795fed71185.png)

ECS container yönetim servisidir. Ancak bu servisi bir sunucuya kurmuyoruz. Yönetim konsoluna gidip, ECS seçeriz. Ardından ECS bize 2 seçenek sunar.

1 - EC2 : Container yönetim servisi olan ECS çalışma esnasında, container'ların çalışacağı container engine yüklü sunucu olduğu sunucuların özellikleri nasıl olmalı? Hangi tip EC2 sunucuları istenilmekte? Tüm bu soruların cevaplarına göre seçimleri yapmalıyız.  AWS gerekli sunucuları oluşturur Daha sonra ECS üzerinden container çalıştırmaya başlarız. Containerlar bizim seçtiğimiz adet ve tipteki EC2 sunucuları üzerinde çalışır.

2 - Fargate : Alt yapıda bulunan sunucuların yönetim işini bizden almaktadır. ECS servisine gidip, Fargate seçeriz, alt yapı olarak Fargate kullanılır. Buna göre ram-cpu ihtiyacımızı belirleriz.  Alt yapı ile işimiz olmaz. Container Engine tamamıyla AWS tarafından yönetilir.

## EKS - Elastic Kubernetes Service

![](../.gitbook/assets/product-page-diagram\_Amazon-EKS@2x.ddc48a43756bff3baead68406d3cac88b4151a7e.ddc48a43756bff3baead68406d3cac88b4151a7e.png)

Aslında temel olarak ECS'den farklı bir şey değildir. Aynen ECS gibi container yönetimi servisidir. ECS gibi yönetilen bir servistir. Ana farkı ise ECS AWS 'nin kendi container yönetim servisidir. EKS ise Kubernetes ile oluşturulan bir servistir. EKS 'de ECS gibi ECS2 detaylarını sorar ve buna uygun bir cluster oluşturur.

Tüm bunların yanında destekleyici servis olan Elastic Container Registry var. Docker Hub benzeridir. Ama AWS dünyasına özgüdür. Yarattığımız imajları güvenli bir şekilde  saklayabileceğimiz kayıt deposudur. ECS servisi ile tamamen entegre durumdadır.
