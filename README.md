# Klew Hub v1 — HTTP key servisi ve Discord yönetimi

Bu sürüm, key doğrulaması için harici HTTP servisi kullanır. **Oyuna eski Klew Hub server scriptini kurmak gerekmiyor.** Key oluşturma, iptal ve kullanım kayıtları Discord komutlarıyla yönetilir; oyun içi Admin sekmesi varsayılan kapalıdır.

**Yayın durumu:** Portal herkese açık yayımlandı. Paket veya önizleme ekranları canlı bot/key akışının çalıştığını kanıtlamaz.

| Bileşen | Doğrulanmış durum |
| --- | --- |
| HTTP API | Canlı toplu üretim, aktivasyon, filtre ve detay doğrulandı. Discord iptali iki mevcut oturumu ve key'in yeniden kullanımını reddetti; test keyleri iptal edildi. |
| Discord endpoint | Sabit özel kanal panelinde oluşturma formu, key listesi, ayrıntı, vazgeç ve onaylı iptal gerçek Discord UI ile doğrulandı. |
| Discord bildirimleri | Canlı özel kanal mesaj teslimi doğrulandı; gerçek Roblox ve LootLabs olayları ayrıca denenmelidir. |
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

Doğrulanan payload commit'i: `dfbcf1bead867073465221e96103fd8ed1da2bf7`. Sonraki belge commit'leri payload değiştiği anlamına gelmez.

`KlewHubV1.lua` değişken adları kısaltılmış dağıtım dosyasıdır. **Public GitHub Raw bağlantısı kapalı kaynak sağlamaz.** İndiren kişi kodu alabilir; obfuscation yalnız okumayı zorlaştırır. `LICENSE.txt` kullanım koşullarını belirtir, teknik gizlilik sağlamaz. Secrets ve bot tokenı dağıtım dosyalarına konulmaz.

## Key ve Discord yönetimi

Key almak için [başlangıç sayfasını](https://klew-hub-key-service.neataphid16.chatgpt.site/portal) aç. Key al ilk tıklamada LootLabs sekmesini hazırlar; görev sonunda açılan ödül adresi yeni `/key` sonuç sayfasına yönlenir. Bildirim gecikirse bu sayfa tamamlanmayı kontrol eder. Key, yalnız doğrulanmış görev tamamlanınca sonuç sayfasında görünür ve kopyalanabilir. Cookie, key ve özel takip tokenı URL'ye yazılmaz. Görevi başlattığın tarayıcıyla devam et.

Bot, belirlenen sunucuda ve server tarafında doğrulanan Discord kullanıcı allowlist'iyle çalışır. Roblox kullanıcı adı veya istemcinin bildirdiği UserId, Discord admin yetkisi vermez. Bot tokenı ve bootstrap secret yalnız server/yerel kayıt aracı tarafında kalır.

| Komut | İşlem |
| --- | --- |
| `/klew-panel` | Özel kanalda yönetim kartı; oluşturma formu, key yönetimi, kayıtlar ve durum. |
| `/klew-keys` | Beş kayıtlı sayfalar, durum filtresi, detay ve iptal onayı. |
| `/klew-key duration:24 [quantity:3] [target-roblox:123456]` | 1–720 saatlik 1–10 key; isteğe bağlı Roblox UserId hedefi. |
| `/klew-revoke key:KLEW-...` | Tam key veya 64 karakterlik kimlik ile iptal onayını açar. |
| `/klew-logs` | Son 10 panel açılış kaydını yalnız yetkili kullanıcıya gösterir. |
| `/klew-status` | Key servisi ve LootLabs yapılandırmasını gösterir. |
| `/klew-help` | Kullanımı gösterir. |

Key süre dolmadan aktifleştirilir; erişim süresi ilk doğrulamadan başlar. Hedef verilmediyse ilk kullanan Roblox UserId'ye bağlanır. Bu dış servis sürümünde Roblox kimlikleri istemcinin beyanıdır; OAuth ile doğrulanmış değildir.

Yönetim kartı belirlenen özel kanaldadır; tam key oluşturma, liste ve ayrıntı yanıtlarını yalnız işlemi yapan admin görür. Tam key yalnız oluşturma yanıtında verilir; sonraki listede kimlik ve durum gösterilir. İptal onay ister, vazgeç key'i korur, tekrar iptal ilk iptal tarihini değiştirmez. Aynı interaction tekrar gelirse ikinci key üretilmez. Kesintide key yanıtı otomatik kurtarılamayabilir. Discord yönetimi oyuncu giriş kotasını tüketmez; admin ve işlem başına ayrı kotalar vardır: Hello 120, liste/detay 60, oluşturma/iptal 30 istek/dakika. Ayrıntılı kurulum geliştirici paketindeki `work/klew-key-service/DISCORD-KURULUM.md` dosyasındadır; `env.example` yalnız boş/örnek ayarlar içerir.

Key oluşturma, ilk aktifleştirme ve ayrı panel açılışları kalıcı bir bildirim kuyruğuna kaydedilir. Bot, yapılandırılmış özel kanala saat, bağlantı IP'si ve scriptin bildirdiği Roblox kullanıcı adı/UserId ile mesaj gönderir. Key alınırken Roblox kimliği henüz bilinmeyebilir; aktifleştirmede bildirilir. Discord'dan oluşturulan key'in IP'si bilinmiyor olarak gösterilir. Tam key veya oturum tokenı bildirim mesajına yazılmaz; kısa key izi ve **Key iptal et** düğmesi vardır. Düğme Discord imzası, sunucu, kanal ve admin ID'si doğrulanarak çalışır. Kanalın özel olduğu her gönderimde kontrol edilir; sunucu Administrator yetkisine sahip kişiler Discord gereği yine görebilir.

Bildirim işlemi key mutasyonuyla aynı veritabanı transaction'ında yazılır. Rate limit ve bağlantı kesintisinde kuyruk korunur; sonraki API trafiği veya Discord komutu teslimi yeniden dener. Boşta otomatik zamanlayıcı yoktur. Bu nedenle kesintiden sonra sabit bir saniyede teslim veya tam bir kez teslim garantisi verilmez. Backend key iptalini hemen uygular; açık istemci sonraki 15 saniyelik doğrulama isteğinde iptali görür. Bağlantı kesilirse mevcut yerel erişim lease'i son başarılı doğrulamadan yaklaşık 35 saniye sonra kapanır.

## Oturum ve ayarları hatırlama

Doğrulanmış normal kullanıcı oturumu ve geçerli ayarlar aynı hesap/servis kapsamında hatırlanır. Yeniden çalıştırmada sunucu `Hello` doğrulaması başarılı olmadan erişim açılmaz. Key iptal edilmişse, oturumun süresi dolmuşsa veya **Kayıtlı girişi unut** seçilirse kayıtlı giriş temizlenir. Ham key ve admin bootstrap/oturumu saklanmaz. Dosya okuma/yazma sunan ortamda cihazda JSON kayıt tutulur; bu yetenek yoksa yalnız aynı çalışma ortamının belleği kullanılır. Kalıcı oturum tokenı bu yerel JSON'da bulunur; yerel dosyayı paylaşma.

Ayar değişiklikleri, renk, kilit tuşu ve otomatik tuş tercihi saklanır; geçersiz/bozuk kayıt güvenli varsayılanlara döner. **Varsayılanlara dön** ayarları ve E kilit tuşunu sıfırlar.

## ESP ve kamera hedef takibi

- Önceki 430×580 iOS tarzı koyu menü geri getirildi; yatay pill sekmeler, beyaz aktif sekme, kırmızı anahtarlar ve yuvarlak kartlar. Logo `88391726271714` ve açılış animasyonu korunur; panel kullanılabilir canvas boyutuna göre küçülür.
- ESP'de takım arkadaşlarını gizleme varsayılan açık; kutu/isim/sağlık/mesafe isteğe bağlı. Yenileme 30–240 Hz, varsayılan 60 Hz; gerçek hız FPS'yi aşmaz.
- Kamera takibi kendi takımına kilitlenmez. Ayarlanabilir kilit tuşu, basılı tutma/aç-kapat modu, KİLİT düğmesi ve gamepad L2.
- Aim açıkken FOV halkası görünür. Kesin/yumuşak takip seçilebilir; aim ve ESP mesafeleri ayrıdır.
- **Hedefi koru** gelişmiş ayarlardadır ve varsayılan açıktır; kayıtlı tercih korunur. FOV/ekran sınırı ilk hedef seçiminde kullanılır; mevcut geçerli kilit WASD veya fare hareketinde korunur. Ölüm, takım, seçilen parçanın kaybolması, mesafe, görüş hattı ve odak kaybı kontrolleri sürer. **Basılı tut** modunda kilit tuşu basılı kalmalıdır; bırakıldığında kilit çözülür. **Aç / kapat** modunda bir kez basılır, yeniden basılarak bırakılır.
- İsteğe bağlı otomatik basılı tuş: Sol Shift, Sol Ctrl veya Boşluk; hedef kilitliyken ya da kilitli + sol tık sırasında. Varsayılan kapalıdır. Menü/chat/oyun odağı kaybı, hedef kaybı, yetki iptali ve kapanışta tuş bırakılır; geçici key-up hatası için kapanıştan bağımsız sınırlı tekrar denemesi vardır. Resmî [VirtualInput](https://create.roblox.com/docs/reference/engine/classes/VirtualInput) API'si desteklenmiyorsa ayarda durum gösterilir. Gerçek oyun input desteği test edilmedi. AI modu eklenmedi.
- Right Shift menüyü açar; başlıktan sürüklenir; × arayüz ve çizimleri kapatır. İstemciye yüklenmemiş karakterler hedeflenemez.

## Eski Studio dosyaları

`KlewHubV1.server.luau`, `KlewHubV1.server-private.luau`, `KlewHubV1.StudioInstaller.luau` ve GitHub paketindeki server/installer kopyaları **önceki Studio + oyun serveri sürümüne aittir**. Yeni HTTP istemcisinin zorunlu parçaları değildir. Eski installer'ı yeni HTTP istemcisinin kurulumu olarak çalıştırma.

`KlewHubV1.Studio-one-line.luau` ve `KlewHubV1.loadstring.luau` önceki destek/wrapper çıktılarıdır; bu sürümde yeniden doğrulanmadı. `KlewHubV1.Studio-private.luau` adı uyumluluk nedeniyle kalmıştır; güncel payload ile eşitse bile HTTP adaptörü gerektirir. Eski `KURULUM.md` içindeki iki-script/oyun içi Admin açıklamaları önceki sürüme aittir; yeni sürüm için bu belgeyi esas al.

## Doğrulama

Güncel okunabilir istemci ve paketlenmiş `KlewHubV1.lua` üzerinde **156'er test** geçti; taklit Roblox ortamında çalışan mantık testleridir. Kaynak/payload için Luau derlemesi ve yerel adlardan bağımsız AST eşitliği ayrıca kontrol edilir.

15 hareket/kilit regresyonu mevcut render ve input yolunu kullanır. Eski sürüm fare merkezi sıçraması, WASD ile kamera/karakter hareketi ve hedefin kamera arkasına geçmesi testlerinde başarısızdı; düzeltilen kaynak ve payload hepsini geçer. [Kilit kontrol kaydı](KlewHubV1-client-motion-check.json). Bağlı Studio bulunmadığından canlı oyun testi yapılmadı; HTML/JPG menü önizlemesidir.

HTTP backend için **157 test** geçti: Discord 50, Hub 44, LootLabs 31, bildirim 32. Yerel SQLite ve taklit HTTP/Discord yanıtları kullanılır; canlı Discord botu veya LootLabs görev testi değildir.

12 Ekim canlı Discord kontrolünde sabit panelden 2 test key'i üretildi; liste, detay, vazgeç ve onaylı iptal denendi. İptal edilen key iki açık tokenla reddedildi. Test keyleri temizlendi. Sürüm, kanal ve canlı sonuç kaydı [KlewHubV1-discord-admin-live-check.json](KlewHubV1-discord-admin-live-check.json) içindedir. [Özel yönetim panelini aç](https://discord.com/channels/1492835611692765244/1558728156758933524/1558898250277199893).

Ek **2 gerçek workerd kontrolü**, dış ağ kapalı HTTP taklidiyle Worker fetch çağrılarının receiver ve yönlendirme davranışını doğrular.

Portalın gerçek TSX komponentinde **39 taklit ortam kontrolü** geçti; açılış, otomatik sekme, eski yapılandırmadan kurtarma ve takip/iptal akışlarını kapsar. Canlı tarayıcıda Key al düğmesinin tek tıklamayla LootLabs sekmesini açtığı ayrıca doğrulandı.

Yeni sonuç sayfasında **26 taklit ortam kontrolü** çalıştırıldı. Görev sonrası yönlendirme ve cookie doğrulaması backend testlerine dahildir. Bu kontroller gerçek LootLabs reklam tamamlamasının yerine geçmez.

Eski server/installer bu sürümde yeniden test edilmedi. Manifest mevcut hash'lerini saklar, yeni test başarısı iddia etmez. Gerçek Roblox çok oyunculu oturumu test edilmedi. Canlı HTTP/Discord/LootLabs/GitHub sonuçları yukarıdaki yayın tablosunda ayrı belirtilir; yerel testler bu sonuçları kanıtlamaz. HTML/JPG dosyaları önizlemedir. Güncel hash'ler ve paket kopyalarının eşleşmesi `KlewHubV1-release-manifest.json` içindedir.
