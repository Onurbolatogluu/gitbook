---
icon: trash-can
---

# Disable & Delete Workflows

### 1) Disabling a Workflow (Devre Dışı Bırakma)

* Workflow’un tetiklenmesini (push, PR vb.) **geçici** olarak engellemek istersen bu seçeneği kullanırsın.
* İstediğin zaman tekrar **enable** (etkinleştirme) yapabilirsin, böylece workflow kaldığı yerden yeniden çalışmaya başlar.
* **Kullanım Durumu**:
  * “Workflow çok sık tetikleniyor, biraz bakım/inceleme yapılacak” veya
  * “Bu iş akışı geçici olarak gereksiz, ileride yine lazım olabilir.”
*   **Örnek Komut (GitHub CLI)**:

    ```bash
    gh workflow disable my_workflow
    ```

    Sonra “gh workflow enable my\_workflow” gibi bir komutla yeniden etkinleştirebilirsin.

### 2) Deleting a Workflow (Tamamen Silme)

* Workflow YAML dosyasını repo’dan (ör. `.github/workflows/`) silmek veya GitHub arayüzünde “Delete file” demek.
* Bir kez sildiğinde, dosyayı **yeniden** oluşturman veya versiyon geçmişinden kurtarmışsan geriye dönmen gerekir. Yani “geri alma” opsiyonu yoktur.
* **Kullanım Durumu**:
  * “Bu workflow artık hiç kullanılmayacak.”
  * “Düzenleme değil, doğrudan temizlik yapıyorum; gereksiz akışları repo’dan çıkarıyorum.”

#### Hangisini Ne Zaman?

* **Disabling**:
  * “Kodda bakım yaparken workflow tetiklenmesin”
  * “Fazla tetikleniyor, bir süreliğine kapatayım”
  * Kolay ve geri çevrilebilir.
* **Deleting**:
  * “Artık bu workflow’a ihtiyacım yok, repo’dan tamamen kaldırmak istiyorum.”
  * Dosyayı silersen geri getirmek zor olabilir (version history’ye dönmen gerekir).

Kısacası, **disabling** workflow’u geçici olarak durdururken, **deleting** onu kalıcı biçimde repo’dan siliyor.
