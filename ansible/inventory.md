# 🎒 Inventory

Ansible inventory olarak bilinen, liste ya da liste gruplarını kullanan, aynı anda aynı altyapıda bulunan bir veya birden fazla node'ları yönetebilir. Default olarak, /etc/ansible/hosts dosyasında bulunur. Farklı path ve dosya verebiliriz. Aynı anda birden fazla inventory dosyası kullanabiliriz. Cloud,dynamic source'lardan, inventory 'e pull edebiliriz. Inventory dosyası, farklı formatlarda .ini ve .yml olarak yazılabilir.

### INI

```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

### YAML

```
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
```

* Her host birden fazla gruba sahip olabilir.

### Envanter Parametreleri

```
192.168.1.5 web ansible_connection=ssh ansible_user=root

web ansible_port=223 ansible_host=192.168.1.5
```

* ansible\_hosts : Ansible 'ın bağlanacağı node adı. (Farklı bir isim de yazılabilir)
* ansible\_connection : Node 'a bağlanma türü. (SSH vb)
* ansible\_port : Node 'a bağlantı kurarken, kullanacağı port numarası. (ssh için 22)
* ansible\_user : Node 'a bağlanacağımız zaman kullanılacak olan user bilgisidir.
* ansible\_password : Node 'a bağlanacağımız kullanıcının password bilgisidir.
* ansible\_ssh\_private\_key\_file : SSH tarafından kullanılan özel anahtar dosyası. Birden fazla anahtar kullanıyorsanız ve SSH aracısını kullanmak istemiyorsanız kullanışlıdır.
* ansible\_become : sudo yetkisi verir.

{% embed url="https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters" %}

