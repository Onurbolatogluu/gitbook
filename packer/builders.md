---
icon: trowel-bricks
---

# Builders

#### 1. Builder Nedir? (Köprü Görevi)

En basit tabiriyle Builder, Packer ile hedef platformun (AWS, VMware, Azure vb.) konuşmasını sağlayan bileşendir.

* Görevi: Sen Packer'a "Bana bir sunucu lazım" dersin. Builder, gider AWS'ye veya VMware'e bağlanır, API'leri kullanarak boş bir sanal makineyi (VM) ayağa kaldırır.

#### 2. Konfigürasyon (Source Blocks & HCL2)

Packer'a ne yapacağını söylediğimiz reçetelere Template diyoruz (HCL2 formatında yazılır). Burada `source` dediğimiz bloklar Builder'ı tanımlar.

* Builder'a şunu söylersin: "Bana Ubuntu bazlı (Base Image), 2 CPU'lu, 4GB RAM'li bir makine aç."
* Packer'ın en güzel yanı şudur; aynı anda hem AWS hem de VMware için Image üretebilirsin. İkisi için ayrı ayrı Builder tanımlarsın, Packer ikisini paralel çalıştırır.

#### 3. İş Akışı: Pasta Nasıl Yapılır?

1. Builder (Kek Kalıbı ve Fırın): Önce Builder devreye girer ve hamurdan (Base Image) boş bir kek tabanı oluşturur. Yani makineyi Boot eder (ayağa kaldırır).
2. Provisioner (Süsleme Ekibi): Makine açılınca Builder, sahneyi Provisionerlara bırakır. Provisioner'lar makineye bağlanır; içine yazılımları kurar, ayarları yapar (Kekin kremasını ve süsünü koyar).
3. Artifact (Paketlenmiş Ürün): İş bitince makine kapatılır ve paketlenir. Elinde kalan o nihai çıktıya Artifact denir.
   * AWS için bu bir AMI ID'dir.
   * VMware için bir OVF/OVA dosyasıdır.

#### 4. Immutable Infrastructure (Değişmez Altyapı) ve "Baking"

* Eski Yöntem: Bir sunucu kurulur, yıllarca o sunucuya bağlanıp (SSH) güncellemeler elle yapılırdı.
* Packer Yöntemi (Immutable): Uygulamanı ve tüm ayarlarını Builder ile Image'ın içine dahil ederiz (Bake edersin).
* Mantık: Sunucuyu güncellemekle uğraşmazsın. Yeni bir Image oluşturur, eskisini çöpe atar, yenisini Deploy edersin. Tıpkı çizilmiş bir CD'yi tamir etmeye çalışmak yerine yeni bir CD yazmak gibi.

#### 5. Debugging (Hata Ayıklama)

Bazen işler yolunda gitmez. Packer sana şunu sunar:

* Debug Mode: Eğer `-debug` parametresiyle çalıştırırsan, Packer her adımda durur ve sana sorar: "Devam edeyim mi?".
* Hata aldığında makineyi hemen silmez. Sana geçici bir anahtar (Key) verir, makineye SSH ile bağlanıp "Nerede patladı bu?" diye bakmana izin verir.

***

Özetle:

Builder, hedef platformda (AWS, Azure vs.) geçici bir sanal makine açan, işi bittiğinde onu dondurup bir Image (Kalıp) haline getiren ve sana bu kalıbın ID'sini veren işçidir.
