# ▶ Playbook

* Konfigürasyon, deployment, orkestrasyon dosyalarıdır.
* IT işlemleri için uygulanacak adımları tanımlayabiliriz.
* Playbook 'ları, yaml dosyalarına yazacağız.
* Playbook 'lar, aslında ansible 'a adım, adım ne çalıştıracağımızı söyler.

### Playbook Options

{% embed url="https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html#common-options" %}

* \--ask-vault-password : Yetkili kullanıcının şifresini sormak için kullanılan bir opsiyondur.
* \--become-method : SSH yetki için kullanılan yöntemdir.
* \--become-user : Hedef sunucularda, komutların hangi kullanıcı ile çalışmasını istiyorsak bu opsiyon ile belirtebiliriz.
* \--flush-cache : Envanter de bulunan her host için cache temizler.
* \--force-handlers : Bir task fail olsa bile, süreci devam ettirmek için kullanılır.
* \--list-hosts : Hostları listelemeye yarar.
* \--list-tags : Tagları listelemek için kullanılır.
* \--private-key : SSH bağlantısı yapılırken, kullanılacak key dosyasını belirtebiliriz.
* \--ssh-common-args : SSH bağlantısı yapılırken, ek argümanlar verebilmek için kullanılır.
* \--syntax-check : Playbook yazım biçimini kontrol eder.
* \--version : Ansible versiyonunu gösterir.
* \-K, --ask-become-pass : Hangi kullanıcı ile işlem yapılacaksa, onun şifresini bize sorması için kullanılır.
* \-T : Timeout değerini vermek için kullanılır.
* \-c : Bağlantı tipini değiştirmek için kullanılır.
* \-e, --extra-vars : key-value, yml,json ekstra değişkenler eklemek için kullanılır.
* \-f : paralel proccess sayısını değiştirmek için kullanılır.
* \-i : harici envanter dosyamızın var ise, yerini belirtebiliriz.
* \-k : ilgili kullanıcının şifresini sormak için kullanabiliriz.
* \-u : bağlantı kuracağımız farklı bir kullanıcı varsa belirtebiliriz.

### Playbook Keywords

{% embed url="https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html" %}

* become\_method : su - sudo gibi yetkileri belirtmek için kullanılır.
* become\_user : Hangi user ile erişim yapılacak, bunu verebiliriz.
* check\_mode : Task ile nelerin değiştiğini görmek için kullanılır.
* collections : Roller,plugin,modüller için namespace'lerin bulunduğu listedir.
* connections : Taskı çalıştırırken manage node 'a nasıl bağlanacağını belirtmek için kullanırız.
* debugger : Hatalar, geri bildirimleri görmek için debug modu açar.

```
ansible-playbook playbook.yml
```
