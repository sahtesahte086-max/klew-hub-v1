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

Doğrulanan payload commit'i: `c530ca99aa5e4dafca5869c4038aa1b687a39637`. Sonraki belge commit'leri payload değiştiği anlamına gelmez.

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
| `/klew-reset` | Bütün keyleri, normal kullanıcı oturumlarını ve sayaçları sıfırlamak için yazılı onay formu açar. |
| `/klew-help` | Kullanımı gösterir. |

Key süre dolmadan aktifleştirilir; erişim süresi ilk doğrulamadan başlar. Hedef verilmediyse ilk kullanan Roblox UserId'ye bağlanır. Bu dış servis sürümünde Roblox kimlikleri istemcinin beyanıdır; OAuth ile doğrulanmış değildir.

Key alanında **Enter** ile **Doğrula ve devam et** aynı giriş akışını çalıştırır. Açılıştaki `Hello` kontrolü sürüyorsa gönderilen key bir kez bekletilir; kontrol yanıtından veya 15 saniyelik zaman aşımından sonra doğrulanır. Gönderim anındaki metin kullanılır. Yapıştırma sırasında key'in başına veya sonuna eklenen boşluk ve görünmez kopyalama işaretleri temizlenir; key'in içindeki karakterler değiştirilmez. Boş veya 1024 bayttan uzun giriş gönderilmez. Hata/zaman aşımında key alanda kalır ve tekrar denenebilir; **Kayıtlı girişi unut** bekleyen giriş ve eski yanıtın erişim açmasını da iptal eder.

Yönetim kartı belirlenen özel kanaldadır; tam key oluşturma, liste ve ayrıntı yanıtlarını yalnız işlemi yapan admin görür. Tam key yalnız oluşturma yanıtında verilir; sonraki listede kimlik ve durum gösterilir. İptal onay ister, vazgeç key'i korur, tekrar iptal ilk iptal tarihini değiştirmez. Aynı interaction tekrar gelirse ikinci key üretilmez. Kesintide key yanıtı otomatik kurtarılamayabilir. Discord yönetimi oyuncu giriş kotasını tüketmez; admin ve işlem başına ayrı kotalar vardır: Hello 120, liste/detay 60, oluşturma/iptal 30 istek/dakika. Ayrıntılı kurulum geliştirici paketindeki `work/klew-key-service/DISCORD-KURULUM.md` dosyasındadır; `env.example` yalnız boş/örnek ayarlar içerir.

`/klew-reset` veya paneldeki kırmızı sıfırlama düğmesi, önceden doldurulmamış bir onay formu açar. İşlemi yapmak için tam **TUM KEYLERI SIFIRLA** yazılmalıdır. Form; yetkili Discord hesabına, kanala ve sıfırlama amacına bağlı imzayla korunur, 15 dakika geçerlidir. İşlem bütün keyleri, normal kullanıcı oturumlarını, erişim izinlerini, kullanım kayıtlarını, LootLabs takiplerini, bildirim kayıtlarını ve işlem sayaçlarını temizler; yönetici oturumları ve interaction tekrar işlem koruması korunur. Silinen kayıt sayıları yalnız işlemi yapan adminin yanıtında gösterilir. Bu işlem geri alınamaz.

15 Ekim 2026'daki canlı key/sayaç sıfırlaması ve Discord kanal düzenlemesinin sonucu [KlewHubV1-reset-live-check.json](KlewHubV1-reset-live-check.json) dosyasındadır. Bu kayıt, o işlemin sonrasındaki durumu gösterir; daha sonra oluşturulan keyler ayrıca yönetilir.

Key oluşturma, ilk aktifleştirme ve ayrı panel açılışları kalıcı bir bildirim kuyruğuna kaydedilir. Bot, yapılandırılmış özel kanala saat, bağlantı IP'si ve scriptin bildirdiği Roblox kullanıcı adı/UserId ile mesaj gönderir. Key alınırken Roblox kimliği henüz bilinmeyebilir; aktifleştirmede bildirilir. Discord'dan oluşturulan key'in IP'si bilinmiyor olarak gösterilir. Tam key veya oturum tokenı bildirim mesajına yazılmaz; kısa key izi ve **Key iptal et** düğmesi vardır. Düğme Discord imzası, sunucu, kanal ve admin ID'si doğrulanarak çalışır. Kanalın özel olduğu her gönderimde kontrol edilir; sunucu Administrator yetkisine sahip kişiler Discord gereği yine görebilir.

Bildirim işlemi key mutasyonuyla aynı veritabanı transaction'ında yazılır. Rate limit ve bağlantı kesintisinde kuyruk korunur; sonraki API trafiği veya Discord komutu teslimi yeniden dener. Boşta otomatik zamanlayıcı yoktur. Bu nedenle kesintiden sonra sabit bir saniyede teslim veya tam bir kez teslim garantisi verilmez. Backend key iptalini hemen uygular; açık istemci sonraki 15 saniyelik doğrulama isteğinde iptali görür. Bağlantı kesilirse mevcut yerel erişim lease'i son başarılı doğrulamadan yaklaşık 35 saniye sonra kapanır.

## Oturum ve ayarları hatırlama

Doğrulanmış normal kullanıcı oturumu ve geçerli ayarlar aynı hesap/servis kapsamında hatırlanır. Yeniden çalıştırmada sunucu `Hello` doğrulaması başarılı olmadan erişim açılmaz. Key iptal edilmişse, oturumun süresi dolmuşsa veya **Kayıtlı girişi unut** seçilirse kayıtlı giriş temizlenir. Ham key ve admin bootstrap/oturumu saklanmaz. Dosya okuma/yazma sunan ortamda cihazda JSON kayıt tutulur; bu yetenek yoksa yalnız aynı çalışma ortamının belleği kullanılır. Kalıcı oturum tokenı bu yerel JSON'da bulunur; yerel dosyayı paylaşma.

Ayar değişiklikleri, renk, kilit tuşu ve otomatik tuş tercihi saklanır; geçersiz/bozuk kayıt güvenli varsayılanlara döner. **Varsayılanlara dön** ayarları ve E kilit tuşunu sıfırlar.

Key iptali veya oturum süresinin dolması erişimi kapatır; kayıtlı ESP tercihlerini tek başına sıfırlamaz. Yeni key ile yeniden doğrulandıktan sonra aynı kayıtlı ESP ayarları geri yüklenir. **Varsayılanlara dön** tercihleri bilerek sıfırlamak için kullanılır.

15 Ekim 2026'da yeni key ile yeniden doğrulama sonrasında kayıtlı ESP tercihlerinin geri yüklenmesine ilişkin kontrol sonuçları [KlewHubV1-esp-restore-check.json](KlewHubV1-esp-restore-check.json) dosyasındadır. Taklit Roblox ortamındaki bu kontroller canlı oyun testi yerine geçmez.

## ESP ve kamera hedef takibi

- 13 Ekim referansına göre 482×360 gri/cyan menü, solda gezinme, sade kartlar ve açılır gelişmiş ayar grupları kullanılır. Varsayılan **Main** sayfasında **ESP** ve **Settings** bölümleri bulunur; kamera takibi **Aimbot**, kurallı otomatik takip/Tool girdisi **AI** sekmesindedir. Büyütme düğmesi paneli 680×500 yapar; panel kullanılabilir canvas boyutuna göre küçülür. Logo `88391726271714` ve açılış animasyonu korunur.
- ESP'de takım arkadaşlarını gizleme varsayılan açık; kutu/isim/sağlık/mesafe isteğe bağlı. Yenileme 30–240 Hz, varsayılan 60 Hz; gerçek hız FPS'yi aşmaz.
- Kamera takibi kendi takımına kilitlenmez. Ayarlanabilir kilit tuşu, basılı tutma/aç-kapat modu, KİLİT düğmesi ve gamepad L2.
- Aim açıkken FOV halkası görünür. Kesin/yumuşak takip seçilebilir; aim ve ESP mesafeleri ayrıdır.
- **Hedefi koru** Aimbot sayfasında görünür ve varsayılan açıktır; kayıtlı tercih korunur. FOV/ekran sınırı ilk hedef seçiminde kullanılır; mevcut geçerli kilit WASD veya fare hareketinde korunur. Ölüm, takım, seçilen parçanın kaybolması, mesafe, görüş hattı ve odak kaybı kontrolleri sürer. **Basılı tut** modunda kilit tuşu basılı kalmalıdır; bırakıldığında kilit çözülür. **Aç / kapat** modunda bir kez basılır, yeniden basılarak bırakılır.
- İsteğe bağlı otomatik basılı tuş: Sol Shift, Sol Ctrl veya Boşluk; hedef kilitliyken ya da kilitli + sol tık sırasında. Varsayılan kapalıdır. Menü/chat/oyun odağı kaybı, hedef kaybı, yetki iptali ve kapanışta tuş bırakılır; geçici key-up hatası için kapanıştan bağımsız sınırlı tekrar denemesi vardır. Resmî [VirtualInput](https://create.roblox.com/docs/reference/engine/classes/VirtualInput) API'si desteklenmiyorsa ayarda durum gösterilir. Gerçek oyun input desteği test edilmedi.
- Right Shift menüyü açar; başlıktan sürüklenir; □ büyütür, × arayüz ve çizimleri kapatır. İstemciye yüklenmemiş karakterler hedeflenemez.

## AI modu

**AI** sekmesindeki mod, eldeki araca göre profil seçen yerel bir kural sistemidir. Durum alanında deneyimin adı/PlaceId'si ve araç profili gösterilir. LLM kullanmaz; oyunun kurallarını, bütün silah protokollerini veya gereken fare tuşunu kendiliğinden çözdüğü iddia edilmez.

1. Geçerli key ile giriş yaptıktan sonra **AI** sekmesini aç. **Araç profili: Otomatik**, kuşanılmış `Tool` nesnesinin `WeaponType` niteliği ve adı içindeki tanınan sözcüklerden yakın/uzak mesafe profilini seçer. Tanınmayan araç için **Uzak mesafe** veya **Yakın mesafe** profilini elle seçebilirsin; bu seçim oyuna özel saldırı protokolü eklemez.
2. Tarama alanı, mesafe ve ateş aralığını ayarla. **Otomatik ateş** varsayılan açıktır. **Hedefe yaklaş** varsayılan kapalıdır; açılırsa yalnız açık görüş hattında düz `Humanoid:Move` hareketi kullanır. Yol bulma, engel çevresinden dolaşma veya zıplama sistemi değildir.
3. **AI modunu başlat** anahtarını aç ve **Right Shift** ile Klew menüsünü kapat. Klew menüsü açıkken mod bekler; kapatılınca çalışır. Takım arkadaşları seçilmez. Yüklü, görünür, tarama alanı ve mesafe içindeki rakibin başı takip edilir; atış için görüş, menzil ve kamera hizası tekrar kontrol edilir.
4. Saldırı, hazır ve kuşanılmış aracın standart `Tool:Activate()` / `Tool:Deactivate()` girdisiyle yapılır. **Otomatik** profilde tanınmayan araç ateş etmez. Standart Tool aktivasyonunu kullanmayan özel silah sistemlerinde otomatik saldırı çalışmayabilir; keyfi RemoteEvent çağrısı veya fare tuşu tahmini yapılmaz.

Manuel hareket, fare hareketi/tıklama, dokunma veya gamepad girdisi AI modunu kapatır. Sohbet/başka metin alanına odaklanma, pencere odağının kaybı, Roblox menüsü, ölüm/karakter değişimi, erişimin bitmesi ve kapatma da modu durdurur; tekrar başlatmak için AI anahtarını yeniden aç. AI'nın çalışma durumu saklanmaz: **her yeni execute işleminde kapalı başlar**, profil ve diğer geçerli tercihler korunur.

AI profil seçimi, hedef/görüş/mesafe kontrolleri, standart Tool girdisi, durdurma koşulları ve tercihlerin korunmasına ilişkin taklit Roblox kontrolleri [KlewHubV1-ai-check.json](KlewHubV1-ai-check.json) dosyasındadır. Bu kontrol kaydı canlı Roblox oyunu veya her oyunun silah sistemine uyum kanıtı değildir.

## Eski Studio dosyaları

`KlewHubV1.server.luau`, `KlewHubV1.server-private.luau`, `KlewHubV1.StudioInstaller.luau` ve GitHub paketindeki server/installer kopyaları **önceki Studio + oyun serveri sürümüne aittir**. Yeni HTTP istemcisinin zorunlu parçaları değildir. Eski installer'ı yeni HTTP istemcisinin kurulumu olarak çalıştırma.

`KlewHubV1.Studio-one-line.luau` ve `KlewHubV1.loadstring.luau` önceki destek/wrapper çıktılarıdır; bu sürümde yeniden doğrulanmadı. `KlewHubV1.Studio-private.luau` adı uyumluluk nedeniyle kalmıştır; güncel payload ile eşitse bile HTTP adaptörü gerektirir. Eski `KURULUM.md` içindeki iki-script/oyun içi Admin açıklamaları önceki sürüme aittir; yeni sürüm için bu belgeyi esas al.

## Doğrulama

Güncel okunabilir istemci ve paketlenmiş `KlewHubV1.lua` üzerinde **213'er test** geçti; taklit Roblox ortamında çalışan mantık testleridir. Kaynak/payload için Luau derlemesi ve yerel adlardan bağımsız AST eşitliği ayrıca kontrol edilir.

13 Ekim 2026 incelemesinin tarihsel kaydı [KlewHubV1-final-audit-check.json](KlewHubV1-final-audit-check.json) dosyasındadır. O tarihte kaynak ve paket 187'şer kontrolü geçti; bu kayıt güncel kaynak, paket veya yukarıdaki son test toplamının kanıtı değildir.

15 hareket/kilit regresyonu mevcut render ve input yolunu kullanır. Eski sürüm fare merkezi sıçraması, WASD ile kamera/karakter hareketi ve hedefin kamera arkasına geçmesi testlerinde başarısızdı; düzeltilen kaynak ve payload hepsini geçer. [Kilit kontrol kaydı](KlewHubV1-client-motion-check.json). Bağlı Studio bulunmadığından canlı oyun testi yapılmadı; HTML/JPG menü önizlemesidir.

13 Ekim'de eklenen **19 key giriş kontrolü**, gerçek istemci `FocusLost`/Enter ve düğme callback'lerini taklit HTTP yanıtlarıyla çalıştırır. Bu ilk düzeltmenin tarihsel sonucunda toplam 175 kontrolde önce 166 geçti, 9 başarısızdı; düzeltme sonrası 175 geçti, sıfır hata. Bu tarihsel toplam, güncel **213'er test** sonucundan ayrıdır. Başlangıç kontrolü sırasında bekleyen gönderim, key metninin sabitlenmesi, Unicode kenar temizliği, boş/uzun giriş, eski yanıtın iptali ve tekrar deneme kapsanır. [Key giriş kontrol kaydı](KlewHubV1-key-entry-check.json). Kullanıcının canlı HTTP 400 yanıtının gövdesi/hata kodu bulunmadığından bu bildirimin kesin nedeni belirlenmiş değildir; bu testler gerçek Roblox girişi veya canlı key aktivasyonu kanıtı değildir.

HTTP backend için **178 test** geçti: Discord 55, Hub 57, LootLabs 31, bildirim 35. Yerel SQLite ve taklit HTTP/Discord yanıtları kullanılır; canlı Discord botu veya LootLabs görev testi değildir.

12 Ekim v12 canlı Discord kontrolünde sabit panelden 2 test key'i üretildi; liste, detay, vazgeç ve onaylı iptal denendi. İptal edilen key iki açık tokenla reddedildi. Test keyleri temizlendi. Bu tarihli kanıt, 13 Ekim menü/key giriş değişikliğinde Discord akışının yeniden canlı test edildiği anlamına gelmez. Sürüm, kanal ve canlı sonuç kaydı [KlewHubV1-discord-admin-live-check.json](KlewHubV1-discord-admin-live-check.json) içindedir. [Özel yönetim panelini aç](https://discord.com/channels/1492835611692765244/1558728156758933524/1558898250277199893).

Ek **2 gerçek workerd kontrolü**, dış ağ kapalı HTTP taklidiyle Worker fetch çağrılarının receiver ve yönlendirme davranışını doğrular.

Portalın gerçek TSX komponentinde **44 taklit ortam kontrolü** geçti; açılış, otomatik sekme, eski yapılandırmadan kurtarma ve takip/iptal akışlarını kapsar. Canlı tarayıcıda Key al düğmesinin tek tıklamayla LootLabs sekmesini açtığı ayrıca doğrulandı.

Yeni sonuç sayfasında **26 taklit ortam kontrolü** çalıştırıldı. Görev sonrası yönlendirme ve cookie doğrulaması backend testlerine dahildir. Bu kontroller gerçek LootLabs reklam tamamlamasının yerine geçmez.

Eski server/installer bu sürümde yeniden test edilmedi. Manifest mevcut hash'lerini saklar, yeni test başarısı iddia etmez. Gerçek Roblox çok oyunculu oturumu test edilmedi. Canlı HTTP/Discord/LootLabs/GitHub sonuçları yukarıdaki yayın tablosunda ayrı belirtilir; yerel testler bu sonuçları kanıtlamaz. HTML/JPG dosyaları önizlemedir. Güncel hash'ler ve paket kopyalarının eşleşmesi `KlewHubV1-release-manifest.json` içindedir.
