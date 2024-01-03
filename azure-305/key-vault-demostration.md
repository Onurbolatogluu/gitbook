# 🔐 Key Vault Demostration

Azure Key Vault, Microsoft Azure tarafından sunulan bir bulut hizmetidir. Bu hizmet, şifreleme anahtarlarını (encryption keys), sertifikaları, API anahtarlarını ve diğer hassas bilgileri güvenli bir şekilde saklamak ve yönetmek için kullanılır. Key Vault'un temel amacı, bu tür hassas bilgileri merkezi bir yerde saklayarak güvenliğini sağlamak ve kolay erişim ile yönetimi mümkün kılmaktır. Bu, uygulama geliştiricilerin ve sistem yöneticilerinin güvenlik yönetimini kolaylaştırır ve geliştirme süreçlerinde verimliliği artırır. Ayrıca, hassas verilerin korunmasına yardımcı olur ve uyumluluk gerekliliklerini karşılamak için gereken denetim ve izleme özelliklerini sunar.

1. **Encryption Keys Yönetimi**:
   * Azure Key Vault, veritabanlarındaki hassas verilerin şifrelenmesi ve güvenli iletişim için kullanılan encryption keys (şifreleme anahtarları) saklamak ve yönetmek için kullanılır.
   * Bu anahtarlar, Azure'un güvenli veri merkezlerinde saklanır ve kullanıcıların erişim düzeylerine göre yönetilebilir.
2. **Sertifika Yönetimi**:
   * SSL/TLS gibi güvenlik sertifikalarını saklamak ve yönetmek için kullanılır.&#x20;
   * Key Vault, sertifikaların ömrünü yönetebilir ve otomatik yenileme gibi işlemleri destekleyebilir.
3. **API Anahtarları ve Diğer Gizli Bilgilerin Saklanması**:
   * Uygulamaların harici hizmetlere erişiminde kullanılan API keys ve benzeri gizli bilgileri saklamak için kullanılır.
   * Bu yaklaşım, hassas bilgilerin uygulama kodu içinde sabitlenmesi riskini azaltır ve güvenlik risklerini önler.
4. **Merkezi Saklama ve Güvenlik**:
   * Key Vault, tüm hassas bilgileri merkezi bir konumda saklar, bu da güvenlik yönetimini ve erişimi kolaylaştırır.
   * Erişim kontrolleri ile kullanıcıların ve uygulamaların hangi bilgilere erişebileceği detaylı bir şekilde ayarlanabilir.



***

<figure><img src="../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

**Adım 1:** Öncelikle, service principal oluşturuyoruz. Ve oluşturduğumuz service principal overview kısmından, Application (client) ID - Object ID - Directory (tenant) ID bilgilerini bir yere kaydediyoruz.



***

<figure><img src="../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

**Adım 2:** Oluşturduğumuz key vault objesi içerisinde, "Client Secret" yaratmalıyız. Ve oluşturduktan sonra ekranda çıkan client secret id ve value değerlerini not etmeliyiz.



***

<figure><img src="../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

**Adım 3:** Ardından, Key Vault objesi oluşturalım ve Access policies kısmından service principal için gerekli izinleri verelim. Bizim örneğimizde get-list izinleri verdik.



***

<figure><img src="../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

**Adım 4:** Key vault içerisinde, test için erişip okuyacağımız bir secret oluşturuyoruz.



***

<figure><img src="../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

```bash
pip install azure-identity azure-keyvault-secrets
```

**Adım 5:** Testi yapacağımız cihazımıza, python key vault paketini ve identity paketini pip ile kurmalıyız.



***

{% code title="list_secrets.py" %}
```python
from azure.identity import ClientSecretCredential
from azure.keyvault.secrets import SecretClient

# Service Principal ve Key Vault Bilgileri
tenant_id = "<your-tenant-id>"
client_id = "<your-client-id>"
client_secret = "<your-client-secret>"
key_vault_url = "https://<your-key-vault-name>.vault.azure.net/"

# Service Principal Kimlik Doğrulaması
credential = ClientSecretCredential(tenant_id, client_id, client_secret)
client = SecretClient(vault_url=key_vault_url, credential=credential)

# Key Vault'taki Secret'leri Listele ve Değerlerini Yazdır
print("Key Vault'taki Secret ve Değerleri:")
secrets = client.list_properties_of_secrets()
for secret_properties in secrets:
    secret = client.get_secret(secret_properties.name)
    print(f"- {secret.name}: {secret.value}")

```
{% endcode %}

<figure><img src="../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

**Adım 6:** Sıra geldi, scriptimizi çalıştırıp, key vault üzerinde tuttuğumuz secret ve secret value değerlerini almaya, bunun için yukarıdaki scripti çalıştırdık ve göreceğiniz üzere, secret ve secret value değerlerini başarılı bir şekilde alabildik.



***



{% hint style="info" %}
Yukarıdaki işlemlerin tamamını "service principal" yerine, "managed identities" ile yapabilirdik. Burada sonuç değişmezdi. Bu örnekte service principal'ı denedik.
{% endhint %}
