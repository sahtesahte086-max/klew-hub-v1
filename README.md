# Klew Hub v1

Kendi Roblox test deneyimin için kırmızı, saydam arayüz; logo `88391726271714`.

**Bu sürüm iki script gerektirir.** Kalıcı kayıt ve key doğrulaması sunucuda yapılır. Admin hesabı yalnız **PinCodez_TR** (UserId `4484679888`); diğer hesaplar key ile girer.

## Studio kurulumu

En kolay yol: `KlewHubV1.StudioInstaller.luau` dosyasının tamamını Studio **Command Bar** içine, **Play kapalıyken** yapıştırıp çalıştır. `ServerScriptService/KlewHubV1_Server` ve `StarterPlayer/StarterPlayerScripts/KlewHubV1_Client` doğru yerlerine eklenir. Önceki Klew Hub LocalScript kopyasını kaldır; oyunu kaydet/yayınla.

Elle kurmak istersen:

1. `KlewHubV1.server.lua` dosyasının tamamını **ServerScriptService > Script** içine yapıştır.
2. `KlewHubV1.lua` dosyasının tamamını **StarterPlayer > StarterPlayerScripts > LocalScript** içine yapıştır.
3. Deneyimini yayınla. Studio testlerinde **Game Settings > Security > Enable Studio Access to API Services** açık olmalı; kalıcı key ve geçmiş için DataStore erişimi gerekir. Canlı deneyimde normal sunucu DataStore erişimi kullanılır.
4. PinCodez_TR ile oyuna gir. **Admin** sekmesinden key oluştur; diğer hesaplar giriş ekranına key'i yazar.

## Özellikler

- Yeni logolu açılış animasyonu, sade kırmızı menü, kapalı gelişmiş ayar grupları.
- ESP başlangıçta yalnız vücut dolgusu ve konturu çizer; kutu/isim/sağlık/mesafe isteğe bağlıdır.
- **Takım arkadaşlarını gizle** varsayılan açık; ESP yenileme aralığı **30–240 Hz**, varsayılan **60 Hz**. Gerçek hız oyun FPS'ini aşmaz.
- Aim kendi takımına hiçbir zaman kilitlenmez. **E** basılı tutma veya aç/kapat modu; yeniden atanabilen tuş, KİLİT düğmesi ve gamepad L2.
- Aim açılınca FOV halkası görünür. **Kesin hedef takibi** varsayılan açıktır; seçili gerçek baş/gövde parçasını takip eder. Kapatılınca yumuşak takip uygulanır.
- Aim mesafesi ESP'den ayrı, varsayılan **4000 stud**, ayarlanabilir **100–10000 stud**. Baş bulunamazsa kafa yerine köke kilitlenmez. İstemciye yüklenmemiş uzak karakterler hedeflenemez.
- Right Shift menüyü açar; başlıktan sürüklenir; × bütün Klew Hub görsellerini kapatır.
- Admin key oluşturabilir (1–720 saat, varsayılan24), isteğe bağlı UserId'ye bağlayabilir ve iptal edebilir. İlk kullanılan key tek hesaba bağlanır; süre ilk doğrulamadan başlar ve tekrar kullanımda uzamaz. Kullanılmamış key'in kullanılabilme son tarihi oluşturulma zamanına göre aynı süre kadardır.
- Başarılı ilk menü açılışı, her Roblox bağlantısı için bir kullanım kaydı üretir. Kullanıcı adı, UserId ve sunucu saati kaydedilir; menüyü gizleyip yeniden açmak aynı bağlantıda yeni kayıt üretmez. Son300 kayıt deneyim sunucularında kalıcı tutulur; menü en yeni100 kaydı Türkiye saatiyle gösterir.
- Admin canlı bildirimleri alır; geçmişi sonradan da açabilir. MessagingService hızlı güncellemeye yardımcı olur; kaçan bildirimler periyodik doğrulama ve geçmiş yenilemeyle tamamlanır.
- Key iptali/süre dolması sunucuda doğrulanır. Sunucu erişimi yenilenemezse normal kullanıcının özellikleri kapanır. İstemci kuyruğu zaman aşımından sonra yeniden denemeye izin verir.

## LootLabs ve oyun entegrasyonu

**LootLabs canlı bağlantısı kurulmadı.** Kampanya, hesap API ayarı ve güvenilir harici doğrulama hizmeti verilmedi. “Key al” düğmesi bunu açıkça belirtir; reklam tamamlandı gibi davranıp sahte key üretmez. Adminin manuel oluşturduğu key bağımsız olarak kullanılabilir. API anahtarını istemciye veya GitHub'a koyma. Tek görevli kampanya için sağlayıcı hesabında `number_of_tasks=1` yapılandırılması ve tamamlanma doğrulaması gerekir. [LootLabs API](https://help.lootlabs.gg/en/article/lootlabs-api-documentation-1k0hn73/), [postback](https://help.lootlabs.gg/en/article/postback-api-1ndz3i2/)

Sunucu oyuncunun `KlewHubAuthorizedTest` attribute'unu doğrulanmış erişimde ayarlar. Kendi oyun sistemin bu işareti ancak sunucuda kontrol ederek test özelliklerini tanıyabilir. Mevcut anti-cheat kodu incelenmedi; kick sorununun giderildiği veya görünmezlik garantisi yoktur. Bu kod anti-cheat/Adonis gizleme içermez.

Otomatik fiziksel Sol Shift basışı eklenmedi. Normal LocalScript için sprint/shift-lock işlevinin oyunun kendi hareket sistemine bağlanması gerekir; kilit tuşunu ayarlardan Sol Shift seçmek mümkündür.

## Tek satırlık bağlantı

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/sahtesahte086-max/klew-hub-v1/main/KlewHubV1.lua"))()
```

Bu biçimi destekleyen ortamda istemci dosyasını indirir; **sunucu scriptinin deneyime önceden kurulması yine gereklidir**. Normal StarterPlayer LocalScript'inde bu HTTP/loadstring biçimi kullanılmaz; dosyanın tamamını doğrudan yapıştır veya Studio kurulum dosyasını kullan. `KlewHubV1.lua` ve server dağıtımı adları kısaltılmış sürümlerdir. Public Raw bağlantısı kodu tamamen gizlemez; lisans `LICENSE.txt` içindedir.

## Doğrulama ve pratik sınırlar

91 istemci ve 39 sunucu kaynak testinin tamamı geçti; aynı testler dağıtım dosyalarında da geçti. Luau derlemesi ve yeniden adlandırma sonrası sözdizimi ağacı eşitliği doğrulandı. Giriş, takım filtresi, 3000 stud baş takibi, kamera sıfırlanması, key yetkileri, süre/iptal, kalıcı veri hataları, canlı kayıtlar, kuyruk/zaman aşımı ve geç cevaplar taklit Roblox ortamında test edildi. Bağlı Studio oturumu bulunmadığı için gerçek çok oyunculu Roblox/DataStore çalışması doğrulanmadı. HTML/JPG görselleri tasarım önizlemesidir.

Son300 kayıt tek DataStore anahtarında tutulur; bu küçük test oyunu içindir. Çok sunuculu yüksek trafik için kayıtları bölmek/toplu yazmak gerekir. [DataStore sınırları](https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits)

Uygulama sunucu kimliğini ve verisini esas alır. [Client-server güvenliği](https://create.roblox.com/docs/scripting/security/client-server-boundary), [DataStores](https://create.roblox.com/docs/cloud-services/data-stores), [Camera](https://create.roblox.com/docs/reference/engine/classes/Camera), [UI animasyonu](https://create.roblox.com/docs/ui/animation)
