# Klew Hub v1 — HTTP key servisi ve Discord yönetimi

Bu sürüm, key doğrulaması için harici HTTP servisi kullanır. **Oyuna eski Klew Hub server scriptini kurmak gerekmiyor.** Key oluşturma, iptal ve kullanım kayıtları Discord komutlarıyla yönetilir; oyun içi Admin sekmesi varsayılan kapalıdır.

**Yayın durumu:** Portal herkese açık yayımlandı. Paket veya önizleme ekranları canlı bot/key akışının çalıştığını kanıtlamaz.

| Bileşen | Doğrulanmış durum |
| --- | --- |
| HTTP API | Canlı anonim Hello yanıtı doğrulandı; key aktivasyonu bununla doğrulanmış olmaz. |
| Discord endpoint | Canlı Discord endpoint doğrulandı; bot komutlarının uçtan uca çalışması ayrıca test edilmelidir. |
| LootLabs | LootLabs kampanyası yapılandırıldı; canlı tamamlanma ve key teslimi henüz doğrulanmadı. |
| LootLabs Start/Poll | Public Start 200 ile tek aşamalı kampanya bağlantısı verdi; Poll 200, pending ve key yok yanıtı verdi. Reklam tamamlanması/key teslimi test edilmedi. |
| LootLabs provider ayarı | LootLabs panelindeki postback ayarları yeniden yükleme sonrasında kalıcı olarak doğrulandı. Gerçek callback teslimi test edilmedi. |
| GitHub bu sürüm | GitHub Raw payload'ı yerel sürümün SHA-256 hash'iyle eşleşerek doğrulandı. |

## İstemci için gereken ortam

Çalışma ortamının HTTP **POST** adaptörü sağlaması gerekir. Okunabilir kaynakta `CONFIG.HttpRequest` bu adaptöre bağlanabilir: `Url`, `Method`, `Headers`, `Body` alanlarını alır; `StatusCode` ve JSON metni içeren `Body` alanlarına sahip bir tablo döndürür.

**Standart Roblox StarterPlayer LocalScript'i dış HTTP isteği gönderemez.** Bu HTTP sürümünü yalnız LocalScript'e yapıştırmak yeterli değildir. `game:HttpGet` ve istemcide `loadstring`, aşağıdaki loader için standart Roblox istemci API'leri değildir. Olağan Studio/oyun ortamında kendi server üzerinden bir HTTP köprüsü ayrıca tasarlanmalıdır. [HttpService kısıtları](https://create.roblox.com/docs/reference/engine/classes/HttpService), [loadstring](https://create.roblox.com/docs/reference/engine/globals/LuaGlobals#loadstring)

Kaynakta yapılandırılan hedef adres:

```lua
CONFIG.KeyServiceURL = "https://klew-hub-key-service.neataphid16.chatgpt.site"
```

Bu adresin yazılı olması canlı API doğrulaması değildir. HTTP adaptörü yoksa veya servis hazır değilse giriş ekranı hatayı gösterir ve test özellikleri açılmaz.

## Aynı Raw loader

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/sahtesahte086-max/klew-hub-v1/main/KlewHubV1.lua"))()
```

Bu satır, yükleme yöntemini ve gereken HTTP POST adaptörünü sağlayan ortamda istemciyi indirir. Repo yolu aynı kaldı. GitHub Raw payload'ı yerel sürümün SHA-256 hash'iyle eşleşerek doğrulandı.

Doğrulanan payload commit'i: `c38147424c4c6ebece7b06a24f808841d407934a`. Sonraki belge commit'leri payload değiştiği anlamına gelmez.

`KlewHubV1.lua` değişken adları kısaltılmış dağıtım dosyasıdır. **Public GitHub Raw bağlantısı kapalı kaynak sağlamaz.** İndiren kişi kodu alabilir; obfuscation yalnız okumayı zorlaştırır. `LICENSE.txt` kullanım koşullarını belirtir, teknik gizlilik sağlamaz. Secrets ve bot tokenı dağıtım dosyalarına konulmaz.

## Key ve Discord yönetimi

Bot, belirlenen sunucuda ve server tarafında doğrulanan Discord kullanıcı allowlist'iyle çalışır. Roblox kullanıcı adı veya istemcinin bildirdiği UserId, Discord admin yetkisi vermez. Bot tokenı ve bootstrap secret yalnız server/yerel kayıt aracı tarafında kalır.

| Komut | İşlem |
| --- | --- |
| `/klew-key duration:24 [target-roblox:123456]` | 1–720 saatlik key; isteğe bağlı Roblox UserId hedefi. |
| `/klew-revoke key:KLEW-...` | Key'i ve bağlı erişimi iptal eder. |
| `/klew-logs` | Son 10 panel açılış kaydını yalnız yetkili kullanıcıya gösterir. |
| `/klew-status` | Key servisi ve LootLabs yapılandırmasını gösterir. |
| `/klew-help` | Kullanımı gösterir. |

Key süre dolmadan aktifleştirilir; erişim süresi ilk doğrulamadan başlar. Hedef verilmediyse ilk kullanan Roblox UserId'ye bağlanır. Bu dış servis sürümünde Roblox kimlikleri istemcinin beyanıdır; OAuth ile doğrulanmış değildir.

Discord yanıtları private'dır. Aynı interaction tekrar gelirse ikinci key üretilmez. Kesintide key yanıtı otomatik kurtarılamayabilir. Mevcut backend admin başına dakikada 10 komut sınırı uygular; yardım komutu bu sınırı kullanmaz. Ayrıntılı kurulum geliştirici paketindeki `work/klew-key-service/DISCORD-KURULUM.md` dosyasındadır; `env.example` yalnız boş/örnek ayarlar içerir.

## ESP ve kamera hedef takibi

- Saydam kırmızı menü, logo `88391726271714`, açılış animasyonu ve kapalı başlayan gelişmiş ayarlar.
- ESP'de takım arkadaşlarını gizleme varsayılan açık; kutu/isim/sağlık/mesafe isteğe bağlı. Yenileme 30–240 Hz, varsayılan 60 Hz; gerçek hız FPS'yi aşmaz.
- Kamera takibi kendi takımına kilitlenmez. Ayarlanabilir kilit tuşu, basılı tutma/aç-kapat modu, KİLİT düğmesi ve gamepad L2.
- Aim açıkken FOV halkası görünür. Kesin/yumuşak takip seçilebilir; aim ve ESP mesafeleri ayrıdır.
- Right Shift menüyü açar; başlıktan sürüklenir; × arayüz ve çizimleri kapatır. İstemciye yüklenmemiş karakterler hedeflenemez.

## Eski Studio dosyaları

`KlewHubV1.server.luau`, `KlewHubV1.server-private.luau`, `KlewHubV1.StudioInstaller.luau` ve GitHub paketindeki server/installer kopyaları **önceki Studio + oyun serveri sürümüne aittir**. Yeni HTTP istemcisinin zorunlu parçaları değildir. Eski installer'ı yeni HTTP istemcisinin kurulumu olarak çalıştırma.

`KlewHubV1.Studio-one-line.luau` ve `KlewHubV1.loadstring.luau` önceki destek/wrapper çıktılarıdır; bu sürümde yeniden doğrulanmadı. `KlewHubV1.Studio-private.luau` adı uyumluluk nedeniyle kalmıştır; güncel payload ile eşitse bile HTTP adaptörü gerektirir. Eski `KURULUM.md` içindeki iki-script/oyun içi Admin açıklamaları önceki sürüme aittir; yeni sürüm için bu belgeyi esas al.

## Doğrulama

Güncel okunabilir istemci ve paketlenmiş `KlewHubV1.lua` üzerinde **91'er test** geçti; taklit Roblox ortamında çalışan mantık testleridir. Kaynak/payload için Luau derlemesi ve yerel adlardan bağımsız AST eşitliği ayrıca kontrol edilir.

HTTP backend için **69 test** geçti: Discord 24, Hub 21, LootLabs 24. Yerel SQLite ve taklit HTTP/Discord yanıtları kullanılır; canlı Discord botu veya LootLabs görev testi değildir.

Eski server/installer bu sürümde yeniden test edilmedi. Manifest mevcut hash'lerini saklar, yeni test başarısı iddia etmez. Gerçek Roblox çok oyunculu oturumu test edilmedi. Canlı HTTP/Discord/LootLabs/GitHub sonuçları yukarıdaki yayın tablosunda ayrı belirtilir; yerel testler bu sonuçları kanıtlamaz. HTML/JPG dosyaları önizlemedir. Güncel hash'ler ve paket kopyalarının eşleşmesi `KlewHubV1-release-manifest.json` içindedir.
