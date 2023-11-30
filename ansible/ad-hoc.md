# 🏑 ad-hoc

ad-hoc komutları, bir veya daha fazla manage node üzerinde single komut çalıştıran ve /usr/bin/ansible 'i kullanan bir komut yöneticisidir. Daha pratik, daha az öneme sahip işlemlerde kullanılabilir.&#x20;

syntax;

```
ansible [pattern] -m [module] -a "[module options]"
```

Modülleri, ad-hoc task olarak kullanabiliriz. Bir çok işlemi ad-hoc ile yapabiliriz. Ansible 'da default olarak 5 proccess aynı anda çalışır. Dilersek, -f parametresi ile farklı bir sayıya set edebiliriz.

```
ansible nodes -m shell -a "echo hello world"
      (grup ismi)
      
      
ansible nodes -m ansible.builtin.shell -a 'echo $TERM'

ansible nodes -m command -a  "hostname" --become --become-user onur

ansible nodes -m shell -a "sudo /bin/systemctl restart iptables" -u onur
sudoers dosyasında onur kullanıcısına yetki vermemiz gerekiyor.
```

{% embed url="https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html" %}

{% embed url="https://www.middlewareinventory.com/blog/ansible-ad-hoc-commands" %}
