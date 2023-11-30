# 📼 Tape Gateway

![](../.gitbook/assets/Replace\_Tape\_Backup\_Diagram.8e2372fac68648f79dd0684026244d50fbd1a10d.png)

* ISCSI VTI protokolü ile bağlanabileceğimiz sanal tape kütüphanesi sunar. ve Fiziksel olarak lTO kartuşlar yerine bulut da oluşturacağınız sanal kartuşlara yedekleme yapmaya imkan tanır.
* Sanal kartuşlar S3 de tutulur.
* Glacier desteği sayesinde daha uzun süre saklamamız gereken sanal kartuşları uygun maliyet de saklayabiliriz.
* Storage GW başına 1 PB kadar sıkıştırılmış yedekleme dosyası saklayabiliriz.
