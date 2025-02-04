---
icon: calendar-days
---

# Manuel Events

Normalde GitHub Actions, genelde push/pull request gibi olaylar ya da cron zamanlamasıyla otomatik tetiklenir. Ancak bazı durumlarda, üç farklı şekilde manuel olarak tetikleme yapabiliriz:

* **GitHub UI** (GitHub arayüzünden “Run workflow” butonu vb.)
* **GitHub CLI** (Terminalde `gh` komut satırı aracıyla)
* **GitHub REST API** (API çağrısı yaparak)

```bash
gh workflow run greet.yml \
  -f name=mona \
  -f greeting=hello \
  -F data=@myfile.txt
```

* `gh workflow run greet.yml` komutu, `greet.yml` adlı bir GitHub Actions iş akışını başlatıyor.
* `-f name=mona` ve `-f greeting=hello` gibi parametreler, iş akışında `inputs` veya benzeri değişkenleri varsa bunları doldurmak için kullanılıyor.
* `-F data=@myfile.txt` ile de bir dosya (örneğin `myfile.txt`) doğrudan payload olarak aktarılabiliyor. (Buradaki fark: `-f` parametreyi düz metin gönderirken, `-F` form-data veya dosya gönderimi için kullanılır.)

{% embed url="https://cli.github.com/manual/gh_workflow" %}
