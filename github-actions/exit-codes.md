---
icon: right-from-bracket
---

# Exit Codes

**Exit Codes** konusunu anlamak, GitHub Actions içinde aksiyonlarının (veya script’lerinin) **başarılı mı yoksa hatalı mı**bittiğini belirlemeni sağlar. Bash veya benzeri bir script, “exit 0” ile başarıyı (status = success), “exit 1” (veya başka 0 olmayan kod) ile hatayı (status = failure) bildirir. Bu da iş akışının (workflow) sonucunu etkiler.

### 1) Bash Script İçinde `exit_status=$?`

#### Ne Anlama Gelir?

* “`make build`” gibi bir komut çalıştırdıktan hemen sonra `exit_status=$?` yazarak **son komutun çıkış kodunu**(exit code) alırsın.
* Eğer `exit_status` “0” ise komut başarıyla bitmiştir. “0 olmayan” herhangi bir kod bir hata veya özel durum ifade eder.

#### Örnek:

```bash
make build
exit_status=$?

if [ $exit_status -ne 0 ]; then
  echo "::error ::Build failed with exit code $exit_status"
  exit $exit_status
fi

echo "::set-output name=status::success"
exit 0
```

1. **Komut Başarısızsa** (`-ne 0`):
   * `echo "::error ::Build failed..."` yazarak log’da “error” satırı oluşturur.
   * `exit $exit_status` ile script’i hatayla sonlandırır → GitHub Actions job da **fail** olur.
2. **Başarılıysa**:
   * Son satırda `echo "::set-output name=status::success"` → “status” adında bir output parametresi “success” olarak ayarlanır.
   * `exit 0` → Script normal şekilde biter, job da **success** olur.

***

### 2) Docker Entrypoint Örneği

```bash
#!/bin/sh -l

echo "API_KEY=abc123" >> $GITHUB_ENV  # Örnek environment değişken ekleme

./run-tests.sh
if [ $? -ne 0 ]; then
  echo "::error ::Tests failed"
  exit 1
fi

echo "test-result=passed" >> $GITHUB_OUTPUT
echo "Action completed successfully"
exit 0
```

1. **`./run-tests.sh`** → Testleri çalıştırır. Dönüş kodu 0 değilse “error” mesajı yazar, `exit 1` der (yani job fail).
2. Aksi takdirde, “`test-result=passed`” gibi bir output parametresi kaydedilir (`$GITHUB_OUTPUT`’a yazılıyor), `exit 0` ile başarı belirlenir.

***

### 3) İş Akışında Sonuç

* **Eğer Script “exit 0”** dönerse → Action (ve job) **başarılı** (success) sayılır.
* **Eğer Script “exit 1” (veya başka 0 olmayan kod)** dönerse → Action (ve job) **failed** görünür.
* “**::error**” satırları logda özel şekilde vurgulanır, ama esas önemli olan “exit code”’dur.

***

#### Neden Önemli?

1. **CI/CD Aşamalarını Durdurma**
   * Örneğin bir test script’i başarısızsa `exit 1` diyerek job’ı fail durumuna getirirsin; sonraki adımlar (deploy) atlanır.
2. **Debug**
   * Log’da “::error” mesajı yazarak, neyin yanlış gittiğini daha görünür hale getirirsin.
3. **Output Ayarlama**
   * Başarılıysa `echo "::set-output name=<key>::<value>"` (ya da `$GITHUB_OUTPUT`’a yazmak), workflow’un sonraki adımlarına veri aktarmak için kullanılır.

**Özetle**, GitHub Actions, “exit code 0 → success” ve “exit code != 0 → failure” mantığıyla script sonucunu okur. Bu nedenle, bash script’ler veya Docker entrypoint’lerde, yaptığın işlemlerin sonucuna göre `exit <code>` komutunu doğru kullanmak, workflow’un genel sonucunu kontrol etmeni sağlar.
