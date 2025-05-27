---
icon: arrow-up-left-from-circle
---

# Variable Scope

Ansible değişken modeli, bir değişkenin hangi parçalar tarafından görülebildiğini (yani “scope” veya kapsamını) belirler. Değişkenlerin nerede tanımlandıkları, o değişkenin hangi Ansible görevlerinde ve hangi hedef sunucularda kullanılabileceğini etkiler. Ansible’da üç temel kapsam (scope) vardır:

1. **Host Kapsamı**
2. **Play Kapsamı**
3. **Global Kapsam**

### 1. Host Scope

#### Tanım

“Host scope”, belirli bir host (sunucu) için tanımlanmış değişkenlerin yalnızca o host içerisinde erişilebilir olduğu anlamına gelir. Başka bir deyişle, bir değişken belirli bir makineye atandıysa, o değişkeni playbook içerisinde yalnızca o makineye görev gönderildiğinde kullanabilirsiniz.

#### Örnek

Bir envanter (inventory) dosyası düşünelim:

```ini
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101 dns_server=10.5.5.4
web3 ansible_host=172.20.1.102
```

Burada `dns_server` değişkeni **yalnızca** `web2` host’u için tanımlanmıştır. Eğer bir playbook yazıp bütün host’lara `dns_server` değerini bastırmak istersek:

```yaml
---
- name: Print DNS Server
  hosts: all
  tasks:
    - debug:
        msg: "{{ dns_server }}"
```

Çıktıda, `web2` için `dns_server` değeri görüntülenirken diğer host’lar için “VARIABLE IS NOT DEFINED!” hatası görünür. Çünkü host kapsamındaki bir değişken, tanımlı olmadığı diğer host’larda erişilebilir değildir.

### 2. Play Scope

#### Tanım

“Play scope”, bir değişkenin tanımlandığı play içinde geçerli olması demektir. Eğer bir play’de değişken tanımlarsanız, o değişken yalnızca o play’de kullanılabilir. Bir sonraki play’e otomatik olarak **taşınmaz**.

#### Örnek

Aşağıdaki playbook’ta birinci play’de `ntp_server` değişkeni tanımlanmış, ikinci play’de ise tanımlanmamıştır:

```
---
- name: Play1
  hosts: web1
  vars:
    ntp_server: 10.1.1.1
  tasks:
    - debug:
        var: ntp_server


- name: Play2
  hosts: web1
  tasks:
    - debug:
        var: ntp_server
```

Çalıştırdığımızda Ansible:

* **Play1** içerisinde `ntp_server` değişkenini başarıyla basacak (çünkü orada tanımlı).
* **Play2** içerisinde, aynı değişkeni bulamayacak ve “VARIABLE IS NOT DEFINED!” diyecek.

Bu örnek, play’lerin birbirinden bağımsız kapsamları olduğunu gösterir. Eğer bir değişkeni iki farklı play’de de kullanmak istiyorsak, her iki play’de de tanımlamalıyız (ya da global yapmalıyız).

### 3. Global Scope (Genel Kapsam)

#### Tanım

Global değişkenler, playbook’un **tüm** kısımlarında ve hatta tüm play’ler boyunca geçerlidir. Örneğin, komut satırından ekstra değişken (**--extra-vars**) olarak geçtiğiniz değerler global kapsamda değerlendirilir.

#### Örnek

Playbook’u şu şekilde çalıştırdığınızı düşünelim:

```
ansible-playbook playbook.yml --extra-vars "ntp_server=10.1.1.1"
```

Bu durumda `ntp_server` değişkeni, hangi play olursa olsun otomatikman tanımlıdır. Böylece bütün play’leriniz bu değere erişebilir.

```
---
- name: Play1
  hosts: web1
  tasks:
    - debug:
        var: ntp_server

- name: Play2
  hosts: web1
  tasks:
    - debug:
        var: ntp_server
```

Çıktıda, her iki play’de de `ntp_server` değeri rahatlıkla basılır.

Özetle:

* **Host scope**: Değişken yalnızca tanımlandığı host’ta geçerli.
* **Play scope**: Değişken yalnızca tanımlandığı play’de geçerli.
* **Global scope**: Tüm play’ler ve host’lar için geçerli.

