# Klew Hub v1 — 13 Ekim 2026 hata incelemesi

13 Ekim değişikliği, referansa uygun gri/cyan menüyü ve key girişindeki testle doğrulanan yarış/temizleme sorunlarının düzeltmesini içerir. Hareket sırasında hedefi koruma, oturum/ayar kaydı ve mevcut HTTP key servisi korunur. Aşağıdaki portal ve Discord bölümleri önceki sürümlerin tarihli sonuçlarıdır; yeni istemci değişikliği bunların yeniden canlı test edildiği anlamına gelmez.

## Enter ile key girişi

Enter ve Doğrula ve devam et düğmesi aynı `redeemKey` giriş yolunu kullanıyordu. Başlangıç `Hello` kontrolü sürerken `authBusy` erken dönüşü gönderimi sessizce düşürüyordu. Gerçek kaynakla yapılan gecikmeli HTTP testinde Enter veya düğmeden sonra hiçbir Redeem gönderilmediği doğrulandı. Gönderilen key artık bir kez bekletilir; Hello yanıtı veya 15 saniyelik zaman aşımından sonra gönderilir. Gönderim anındaki metin sabitlenir; alana sonradan yazılan metin bu isteği değiştirmez. Devam eden Redeem sırasında Enter/düğme tekrarları ikinci istek üretmez.

ASCII boşluk temizliği, yapıştırılan BOM, NBSP ve zero-width kenar işaretlerini temizlemiyordu. Özellikle başında bu işaretler bulunan admin key'i Bearer başlığına taşınmıyordu. Yalnız key'in başındaki/sonundaki boşluk ve kopyalama işaretleri temizlenir; içerideki karakterler değiştirilmez. Boş veya 1024 bayttan uzun giriş HTTP isteği üretmez. Hatalı key, bağlantı hatası ve zaman aşımında giriş alanı korunur; kullanıcı tekrar deneyebilir. Redeem `invalid_payload` mesajı artık key girişini anlatır, admin süre alanını anlatmaz.

Bekleyen Redeem yanıtı, Kayıtlı girişi unut sonrasında yeniden erişim açabiliyordu. Giriş yanıtı oturum revizyonuyla karşılaştırılır; unutulan girişin bekleyen key'i silinir ve eski yanıt erişim açamaz. Önceden mevcut olan timeout/yeniden deneme koruması da sürer: geç kalan bir yanıt, daha yeni başarılı oturumu değiştiremez.

**19 yeni kontrol** gerçek istemci FocusLost/Enter, düğme ve render akışını taklit HTTP sınırlarıyla çalıştırır. Düzeltme öncesi 175 kontrolün 166'sı geçti, 9'u başarısızdı. Düzeltme sonrası kaynakta **175 geçti, 0 hata**. Dağıtım kopyasının sonucu ve kaynak/payload hash'leri `KlewHubV1-key-entry-check.json` ile sürüm manifestinde kaydedilir; yayın durumu ayrı doğrulanır.

Bu sorunlar testle yeniden üretildi; kullanıcının bildirdiği canlı HTTP 400 yanıtının kesin nedeni bulunmuş değildir. Servis kaydında istek gövdesi, işlem ve JSON hata kodu görülmedi. Canlı anonim Hello kontrolünün 200 döndürmesi erişilebilirliği doğrular; gerçek kullanıcının key'inin başarılı doğrulandığını kanıtlamaz. Gerçek Roblox çok oyunculu oturumu bu düzeltme için test edilmedi.

## Menü açılıp kapanıyordu

Roblox `HttpService:JSONEncode({})` boş tabloyu `[]` olarak gönderir. İstemci `Hello`, `Open` ve `GetKeyLink` isteklerine boş `payload` ekliyordu; sunucu ise nesne beklediği için 400 `invalid_payload` döndürüyordu. Key içeren `Redeem` başarılı olup ardından açılış kaydı başarısız olduğunda menü kapanıyordu. Önceki test taklidi boş tabloyu nesne olarak yazdığı için bu hata kaçmıştı.

İstemci artık boş payload alanını göndermiyor. Sunucu eski istemcilerin boş dizisini de kabul ediyor; dolu diziler geçersiz kalıyor. Test JSON davranışı gerçek Roblox ile eşlendi. Geçici açılış kaydı ve bağlantı hataları mevcut erişimi hemen kapatmıyor; gerçek key iptali, hesap uyuşmazlığı ve süre dolması erişimi kapatmaya devam ediyor. Yetki kontrolü 15 saniyede bir başlıyor, doğrulanmış erişim 35 saniye boyunca yenilenemezse kapanıyor. Gecikmiş eski yanıtlar yeni oturumu etkileyemiyor.

## Key al tıklaması sonuç vermiyordu

Eski sekme kurulum durumunu yalnız açılışta kontrol ediyordu. Ayrıca ilk tıklama kampanya bağlantısını hazırlıyor, kullanıcıdan ikinci bir bağlantıya tıklamasını bekliyordu.

Kurulum bekleyen görünür sekmeler 15 saniyede bir tekrar kontrol ediyor; sekmeye dönüşte de durum yenileniyor. İlk tıklama sırasında ayrılan pencere, sunucu bağlantıyı döndürünce LootLabs'a gidiyor. Pencere engellenirse açıklama ve elle açılabilen bağlantı gösteriliyor. Tekrarlı tıklamalar üst üste istek başlatmıyor; hata veya sayfadan çıkış boş pencereyi kapatıyor. Canlı tarayıcıda tek tıklama ile LootLabs'ın açılması doğrulandı.

## Discord botu düşünmede kalıyordu

Discord komutu önce özel bekleme yanıtı gönderir, ardından key işlemi ve asıl mesaj güncellemesi yapılır. Worker'ın arka plan işi gerçek `waitUntil` bağlamına bağlandı. Discord'un ilk yanıtı henüz kaydetmemesi, geçici ağ ve oran sınırı hatalarında aynı mesaj tekrar deneniyor. Başarılı key işlemi tekrar çalıştırılmıyor. Bütün teslim denemeleri ortak 20 saniyelik süre kullanıyor; geç gelen yanıt da aynı key metnini taşır. Tanılama olayları key ve etkileşim tokenlarını yazmaz.

Bu ilk düzeltme gerçek route kodu ve Vinext bağlamıyla yerel olarak doğrulanmıştı. 12 Ekim canlı Discord panel kontrolleri aşağıdaki v12 bölümünde ayrı olarak belirtilir; eski düşünme mesajının kendiliğinden tamamlandığı iddia edilmez.

## Görev sonrası ayrı key sayfası

LootLabs'ın mevcut kampanyasının hedef adresi düzenleme ekranında kilitli. Görev sonrası açtığı kök adres, geçerli tarayıcı oturumunu `/key` sayfasına yönlendirir. Postback henüz gelmediyse yeni sayfa tamamlanmayı bekler. Key yalnız sunucunun görev bildirimi doğrulandığında gösterilir.

Tarayıcı oturumu Secure, HttpOnly, host ile sınırlı cookie ile aktarılır. Cookie, key ve özel takip tokenı adres çubuğuna yazılmaz. Önceki IP bağlama, süre ve iptal kontrolleri kullanılır. `/portal` yeni key başlatma sayfasıdır; mevcut key oturumunun yönlendirmesi yeni bağlantı istemeyi engellemez.

Canlı kontrolde sonuç sayfasındaki istemci yönlendirmesi portal dönüşünde takıldı. Dönüş bağlantıları tam sayfa yükleyen standart bağlantılarla değiştirildi; yeni bağlantı akışı cookie yönlendirmesine takılmadan açılır.

## Doğrulamanın sınırları

12 Ekim hareket/kilit sürümünün okunabilir ve dağıtım kopyalarında 156'şar taklit Roblox kontrolü geçmişti; Luau derlemesi ve AST eşitliği doğrulanmıştı. 13 Ekim key giriş kontrolüyle güncel toplam 175'tir. Güncel kaynak/payload sonucu, derleme, AST eşitliği ve GitHub Raw hash doğrulaması bu sürümün kontrol kaydı ve manifestinde ayrı belirtilir. Canlı API'nin eski boş-dizi isteğine 200 dönmesi, gerçek kullanıcının key girişini doğrulamaz.

Backend ve web arayüzü testleri gerçek kaynak kodunu, SQLite ve taklit HTTP cevaplarını kullanır. Ayrıca aşağıda belirtilen canlı API ve Discord panel kontrolleri yapılmıştır. Gerçek Roblox çok oyunculu oturumu veya LootLabs reklam/insan doğrulaması tamamlanarak key teslimi bu raporla test edilmiş sayılmaz.

## Oturum ve ayarlar her çalıştırmada sıfırlanıyordu

Geçerli normal oturum tokenı ve beyaz listedeki ayarlar hesap/servis kapsamında saklanır. Yeni çalıştırma sunucuya yeniden doğrulatır; iptal edilmiş token erişim sağlamaz. Ham key ve yönetici oturumu saklanmaz. Dosya API'si sunan ortamda kayıt diske yazılır; bu API yoksa aynı çalışma ortamının belleği kullanılır. Geçici ağ kesintisi yeniden deneme için kaydı korur. Kayıtlı girişi unut ve Varsayılanlara dön denetimleri ayrı işlemlerdir.

## Otomatik basılı tuş ve menü

Main gelişmiş ayarlarında isteğe bağlı Sol Shift/Sol Ctrl/Boşluk, kilitliyken veya kilitli + ateş ederken modu bulunur. Menü, sohbet, odak kaybı, hedef kaybı, izin iptali ve kapanışta tuş bırakılır. Kapanış sırasında başarısız key-up tekrarının render döngüsü durduktan sonra kaybolması düzeltildi; sınırlı bağımsız tekrar ve eski denemelerin yeni tuş basışını bırakmasını engelleyen nesil kontrolü eklendi. Resmî Roblox VirtualInput API'si desteklenmiyorsa durum gösterilir. Gerçek oyun input desteği test edilmedi. AI modu eklenmedi.

13 Ekim referansına göre 482×360 gri/cyan panel ve solda gezinme menüsü kullanılır. Büyütme düğmesi paneli 680×500 yapar; gelişmiş ayarlar açılır gruplarda sadeleştirilmiştir. Hedefi koru Main sayfasında doğrudan görünür. Logo 88391726271714, oturum/ayar kaydı ve hareket sırasında hedefi koruma düzeltmesi korunur. Panel gerçek kullanılabilir canvas boyutuna göre küçülür. HTML/JPG önizlemeleri gerçek Roblox ekran görüntüsü değildir.

## WASD ve fare hareketinde hedef kilidi düşüyordu

`chooseTarget` içindeki Sticky dalı ilk edinme için kullanılan `targetPoint` kontrollerini tekrar uyguluyordu. Fare merkezinin kayması veya normal kameranın karakter hareketi sırasında bakışı/konumu değiştirmesi, seçili hedefi FOV/ekran sınırı dışına taşıyınca kilit kayboluyordu. WASD ve MouseMovement olayları doğrudan kilidi bırakmıyordu.

Mevcut Sticky hedefin korunması ilk hedef ediniminden ayrıldı. Yeni hedefler hâlâ FOV içinde, ekranda ve kamera önünde seçilir. Mevcut hedef karakter, takım, seçilen parça, mesafe ve görüş hattı kontrollerini geçtiğinde kamera/fare hareketinde korunur. Ölüm, takım değişikliği, duvar, mesafe sınırı veya odak kaybında bırakılır. Basılı tut modunda kilit tuşunu bırakmak, aç/kapat modunda yeniden basmak kilidi çözer. Sticky kapalıysa en yakın uygun hedef her kare yeniden seçilir. `Hedefi koru` Main sayfasındadır; varsayılan açık olmakla birlikte kayıtlı kapalı tercih de korunur.

12 Ekim'deki 15 hareket regresyonu gerçek kaynak ve render/input yolunda çalıştırıldı. Eski kaynakta fare FOV sıçraması, WASD ile kamera/karakter konum değişikliği ve kamera arkasına geçen hedef testleri başarısızdı. O sürümün düzeltilen kaynak ve obfuscation uygulanmış payload'ında 156'şar test, sıfır hata ile geçti; AST eşitliği ve Luau derlemesi başarılıydı. Bu hareket kontrolleri 13 Ekim'in 175 testlik kapsamına da dahildir. Bağlı Roblox Studio bulunmadığından gerçek oyun testi yapılamadı. Tarihli hareket sonuçları `KlewHubV1-client-motion-check.json`, yeni key giriş sonuçları `KlewHubV1-key-entry-check.json` dosyasında tutulur.

## Bildirimler kuyrukta kalıyordu

Canlı serviste key oluşturma, aktivasyon ve iptal başarılıydı; bildirimler kanal kontrolü aşamasında TypeError alıyor ve hiç teslim edilmiyordu. Asıl neden, Node test ortamının kabul ettiği `redirect: "error"` seçeneğinin workerd tarafından desteklenmemesi. Gerçek workerd ve dış ağı kapatılmış 200 HTTP yanıtıyla aynı hata yeniden üretildi; `manual` aynı istekte başarılı oldu. Bildirim GET/POST ve Discord yanıt PATCH istekleri manual kullanıyor; 3xx adresleri izlenmiyor ve bot kimlik bilgisi başka bir adrese taşınmıyor.

Discord yanıtındaki `dependencies.fetch(...)` global fetch fonksiyonunu yanlış `this` bağlamında çağırıyordu. Gerçek workerd testi bu durumda Illegal invocation hatasını da doğruladı; varsayılan adaptör global receiver'ı koruyor. Discord'un [User-Agent gereksinimi](https://docs.discord.com/developers/reference#user-agent) de uygulandı.

Kalıcı kuyruk olayı key işlemiyle aynı transaction içinde kaydeder; hata işlemi kaybetmez. Özel kanal kontrolü başarılı olmadan IP içeren mesaj gönderilmez. Mesaj key oluşturma/ilk kullanım/panel açılışı, IP, istemcinin bildirdiği Roblox kimliği ve Key'i iptal et düğmesi içerir. Tam key ve token mesajda yoktur. Kullanıcı kimliği Roblox OAuth ile doğrulanmış değildir. İptal, tüm bağlı oturumlarda sunucu tarafında hemen geçerlidir; açık istemci sonraki kontrolünde görür.

Bildirim düzeltmesinin önceki doğrulamasında 129 backend testi (Discord 34, Hub 32, LootLabs 31, bildirim 32) ve ek 2 gerçek workerd testi geçmişti. Portal 39 ve sonuç sayfası 26 komponent taklit kontrolü önceki akış doğrulamasına dahildir. Bu sayılar tarihsel sonuçlardır; güncel v12 test toplamı aşağıdadır. Gerçek Roblox input desteği ve LootLabs reklam tamamlaması bu kontrollerle test edilmiş sayılmaz.

11 Ekim 2026 17:14 UTC canlı API kurulum testinde key oluşturma, ilk kullanım ve panel açılışı bildirimlerinin üçü de yapılandırılmış özel Discord kanalına ulaştı. IP alanı ve iptal düğmesi vardı; kullanılan Roblox adı açıkça Klew_Test olarak bildirilmiş bir API testidir. Test key'i iptal edildi; eski token ile Hello 401 ve authorized:false döndü. Önceden bekleyen bildirimler de kuyruktan teslim edildi. Gerçek oyun oturumu veya LootLabs görevi tamamlanmış sayılmaz.

## Discord yönetiminde key iptali ve panel düzeni

Önceki Discord yönetimi her komutta public `Redeem` üzerinden yeniden yönetici girişi yapıyordu. Bu işlem kullanıcı başına dakikada 10 istek sınırına takılıyor, ardından key iptali dahil yönetim komutları `rate_limited` döndürüyordu. Gerçek SQLite üzerinde art arda komutlarla sorun yeniden üretildi.

İmzalı Discord işlemleri artık sunucudaki güvenilen aktör üzerinden çalışıyor. Aktörün Discord izin listesinde bulunması, bootstrap yapılandırması ve izin verilen yönetim işlemi ayrıca kontrol ediliyor; geçici yönetici tokenı veya oturumu oluşturulmuyor. HTTP gövdesi, başlıkları ya da aynı isimde oluşturulan bir Symbol bu yetkiyi taklit edemiyor. Kotalar Discord kullanıcısı ve işlem başına ayrı: Hello 120, liste ve detay 60, toplu üretim ve her iptal işlemi 30 istek/dakika. On komuttan fazla panel gezintisinden sonra key iptalinin çalışması test edildi.

Yeni yönetim paneli tek işlemde 1–10 key üretir; süre 1–720 saat, hedef Roblox UserId isteğe bağlıdır. Liste varsayılan olarak sayfada 5 kayıt gösterir; aktif, kullanılmamış, süresi dolmuş ve iptal edilmiş filtreleri ile detay görünümü bulunur. İptal işlemi onay ve vazgeç düğmeleriyle yürür. Daha önce iptal edilmiş key için `alreadyRevoked` sonucu gösterilir; ilk iptal zamanı değişmez. Eşzamanlı iki iptal yalnız bir değişiklik yapar. Toplu üretim ve bildirim kaydı aynı transaction içindedir; ara hata bütün yeni kayıtları geri alır. İptal edilen key ile yeniden Redeem ve mevcut tüm tokenlarla Hello/Open istekleri reddedilir.

v11, kaynak `a9d8808158926296ada2ab859e29a1e7f2aca017` ile yayımlanmıştı. Özel kanal `🔐・klew-admin` olarak düzenlendi; sabit panel mesajı `1558898250277199893` üzerinde 5 düğmenin bulunduğu doğrulandı. Bu aşamadaki canlı UI kontrolü sonrasında v12 düzeltmesi yayımlandı; güncel kanıtlar aşağıdadır.

v11 doğrulamasında **151 backend testi** geçmişti: Discord 44, Hub 44, LootLabs 31, bildirim 32. Ek **2 gerçek workerd fetch testi** ayrı olarak geçmişti. Hub'ın yeni 12 regresyonu sayfalama/filtre/özet, süre sınırları, detaylar, toplu üretim ve rollback, idempotent iptal, tüm oturumların kapanması, güvenilen Discord aktörü ve bağımsız kotaları kapsar. Bu sayıların yerini aşağıdaki güncel v12 sonuçları alır.

## v12 özel Discord yanıtı ve canlı kontrol

v12, `efbf704538483d4bc56a43a6d622c811888d6549` kaynağından 12 Ekim 2026 14:29:31 UTC'de başarıyla yayımlandı. Deployment kimliği `appgdep_6acceeb60a9481919814f6d75fb7443b`, kaydedilen sürüm `appgprj_6aca5807e07c8191bfba974724c854d2~appgver_273177ec87c08191b69007566d26083b`.

Özel liste ve detay mesajı üzerindeki düğmeler, mevcut mesajı güncelleyen ACK6 yanıtını kullanır. Sayısal ve güvenli ACK tanılamaları eklendi. Yalnız Discord `10008` hata kodunda, özel ACK6 akışı için tek özel follow-up yanıtı denenir; bu sınırlı fallback birim testinde doğrulandı. v11'de görülen özel mesaj 404 hatasının kök nedeni kanıtlanmış değildir. Canlıda `10008` hatasının tetiklendiği veya fallback'in çalıştırıldığı iddia edilmez.

Canlı API'de bu çalışmaya ait iki test key'iyle 10 kontrol geçti. İptalden vazgeçme key'i aktif tuttu; onaylı iptal sonrasında iki eski tokenın Hello isteği 401 döndürdü, Redeem `revoked_key` döndürdü. Tekrarlanan iptal ilk iptal zamanını korudu. API kontrolündeki iki test key'inin temizliği tamamlandı.

12 Ekim 2026 14:28 UTC canlı Discord modal akışında, mevcut sabit panelden 1 saatlik ve hedef UserId `4484679888` olan 2 key oluşturuldu; gerçek yanıt özel olarak teslim edildi. v12'nin özel ACK6 liste akışında 20 kayıt ve key detayındaki doğru zaman bilgisi doğrulandı. `🔐・klew-admin` kanalı özel, panel sabitlenmiş ve 7 slash komutu kayıtlıdır. Aynı `1558898250277199893` panel mesajı kullanılır; ek panel kopyası oluşturulmadı.

Son v12 canlı UI iptal akışı da doğrulandı: bu çalışmaya ait test key'i için özel yanıt `1559211109045633154`, “Key iptal edildi. Bu key’e bağlı erişim artık geçersiz.” metnini gösterdi. Yanıttaki iptal zamanı 12 Ekim 2026 17:32 Türkiye saatiydi. Sonraki API kontrolü Discord UI'dan oluşturulan iki key'in de 1 saatlik ve hedef UserId `4484679888` olduğunu, UI iptalinin kalıcı kayda geçtiğini ve ilk iptal zamanının korunduğunu doğruladı. Bu iki test key'i de iptal edilerek temizlik tamamlandı; başka kullanıcıların key'lerine dokunulmadı. Canlı panelden oluşturma, listeleme, detay ve iptal akışları ile iptalin API'deki kalıcılığı doğrulandı.

12 Ekim v12 doğrulamasında **157 backend testi** geçti: Discord 50, Hub 44, LootLabs 31, bildirim 32. Ek **2 gerçek workerd fetch/receiver testi** ayrı olarak geçti. ESLint, TypeScript ve üretim build'i başarılı oldu. 13 Ekim menü/key giriş düzeltmesi backend değişikliği veya yeni canlı Discord yayını gerektirmez; bu backend ve canlı panel sonuçları tarihli geçmiş kanıt olarak korunur.

12 Ekim Discord kontrolünde tarayıcı ekran görüntüsü alma denemesi zaman aşımıyla başarısız olmuştu; o Discord kontrolünden gerçek ekran görüntüsü mevcut değildir. Canlı kontroller gerçek Roblox çok oyunculu oturumu, oyun input desteği veya LootLabs reklam görevinin tamamlanması olarak sunulmaz.

## 13 Ekim son tarama ve v13 portal düzeltmesi

Güncel kaynak ve paket **187 istemci kontrolünü**, Luau derlemesini ve binding-normalized AST eşitliğini geçti. Dokuz yeni yaşam döngüsü kontrolü eski kaynakta 184 toplam kontrolün 8'ini başarısız gösterdi. Düzeltme sonrası bunların tamamı geçti. Ardından eski Hello sürerken girişi unutma veya lease kaybından sonra yeni key gönderme için üç ek kontrol, düzeltme öncesi üç hata üretti; son kaynak ve paket 187/0 sonucuna ulaştı.

Süre ve lease kontrolü kamera yazımından önce çalışır; kamera yokken de bağlantı kontrolü sürer. Önceki nesle ait tuş bırakma işi yeni bir bırakma denemesini engelleyemez. Erişim GUI'si dışarıdan silindiğinde runtime temizlenir. Bozuk servis adresi ve geçersiz başarı/süre alanları kontrollü hata verir. Eski Hello yanıtı reddedilirken güncel revizyonda bekleyen yeni key gönderimi devam eder.

Portalın geç yanıt hatası da düzeltildi: 15 saniyelik deadline sonrasında başarılı fetch veya JSON gövdesi gelse bile işlem kabul edilmez. Eski kaynakta ilk yeni Hello kontrolü bu hatayı yeniden üretti. Beş yeni deadline/yeniden deneme kontrolü dahil portal **44**, key sonuç sayfası **26** komponent taklit kontrolünü geçti. Discord 50, Hub 44, LootLabs 31 ve bildirim 32 olmak üzere **157 backend kontrolü**, ayrıca **2 gerçek workerd fetch/receiver kontrolü** yeniden çalıştırılarak geçti.

Portal v13, kaynak `eef39457ec1773c10e0f039899f1974dc688952e` ve deployment `appgdep_6ace49655fb88191b748ac90ce2fa8da` ile 13 Ekim 2026 15:08:40 UTC'de başarıyla yayımlandı; audience public ve secret revision 4 korundu. Yayın sonrası anonim Hello 200 / ok:true / authorized:false / storageReady:true / lootlabsConfigured:true döndü. Bu turda canlı key aktivasyonu, Discord yönetim işlemi veya LootLabs görevi tamamlanması denenmedi; önceki tarihli kanıtlar ayrı tutulur.

Menüde Main varsayılan olarak ESP ve Settings kartlarını gösterir; aimbot ve Hedefi koru ayrı Aimbot sekmesindedir. Referansa göre panel 482×360, sidebar 160 px, satırlar 40 px, kart aralıkları 5 px ve cyan anahtarlar kullanır. Büyütülmüş panel 680×500'dür. Key ekranı aynı gri pencere stilini kullanır. Logo ve 29 gerçek ayar korunur; örnek görseldeki oyuna özel sahte kontroller eklenmez. HTML önizlemesi 11 DOM kontrolünü geçti; bu bir tasarım demosudur.

Bu taramada bağlı Roblox Studio yoktu. Canlı Roblox render ve input davranışı doğrulanmış değildir. Ayrıntılı, sır içermeyen kontrol kaydı: [Son tarama](KlewHubV1-final-audit-check.json).

## 15 Ekim v14 sıfırlama ve bildirim yarışları

v14, `71caff48e1162450a696388d2be1ce502090e27c` kaynağından 15 Ekim 2026 17:14:31 UTC'de yayımlandı. Bir defalık migration bütün normal keyleri, oturumları, grantleri, kullanım kayıtlarını, eski LootLabs challenge'larını ve bildirim kuyruğunu sildi. Canlı tabloların boş olduğu doğrulandı; normal oturum kalmadı, 11 yönetici oturum kaydı korundu. Yönetici ayarları ve Discord tekrar işlem koruması korunur. Sonradan üretilen yeni keyler ve yeni kullanım kayıtları normal biçimde yeniden sayılır.

Discord özel kanalında dört eski bot mesajı silindi; mevcut sabit panel korundu, yenilendi ve sekiz komut kaydedildi. `/klew-reset` ve paneldeki sıfırlama düğmesi, sahip/amaç/kanal/süreye bağlı imzalı boş form açar. Tam `TUM KEYLERI SIFIRLA` onayı olmadan işlem çalışmaz. Sıfırlama bir D1 transaction'ıdır; ara hata bütün silmeleri geri alır. Aynı Discord interaction tekrar gelirse işlem yeniden yürütülmez.

Genel incelemede eski Open isteğinin reset/iptal sonrasında yeniden log ve bildirim yazabildiği yarışı yeniden üretildi. Kayıt yazımı artık oturum, key, grant ve süreyi aynı SQL içinde doğrular; bildirim de aynı canlı yetkiyi ve logu kontrol eder. Eski sekiz yarış testi eski kaynakta başarısız, düzeltmeli kaynakta başarılıydı. Güncel **180 servis testi** geçti: Discord 55, Hub 57, bildirim 35, LootLabs 31 ve ayrıca bu toplamın içindeki 2 workerd kontrolü. Üretim derlemesi başarılıdır. [Canlı sıfırlama kaydı](KlewHubV1-reset-live-check.json).

## 15 Ekim açık görünen ama çalışmayan kayıtlı ESP

Erişim kapandığında `applyAccess(false)` kaydedilmiş ESP ve aim tercihini runtime'da false yapıyor, önceden çizilmiş açık anahtar boyasını güncellemiyordu. Yeni key doğrulansa da ESP kapalı kaldığından elle aç-kapa gerekiyordu. Seçilen tercihler artık korunur; her erişim değişimi ESP güncellemesini tetikler. Erişim yokken mevcut çizimler hemen silinir, tuşlar bırakılır ve kamera/ESP/FOV erişim kontrolüyle kapalı kalır.

Üç yeni gerçek istemci regresyonu süresi bitmiş kayıtlı giriş, yeniden çalıştırmada iptal edilmiş giriş ve lease kopması sonrası yeniden doğrulamayı kapsar. Önceki kaynakta toplam 190 kontrolde 187 geçti, bu üçü başarısızdı. Güncel okunabilir kaynak ve paket **190/190** geçti; Luau derlemesi ve normalized AST eşitliği başarılı. Canlı Roblox oturumu test edilmedi. [ESP geri yükleme kaydı](KlewHubV1-esp-restore-check.json).
