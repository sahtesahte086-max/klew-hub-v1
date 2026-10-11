# Klew Hub v1 — 11 Ekim 2026 hata incelemesi

Bu sürüm key portalı, Discord yönetimi ve Roblox HTTP istemcisindeki bağlantı hatalarını düzeltir. Yeni key sonucu `/key` sayfasında gösterilir. Hatırlanan oturum ve ayarlar, özel Discord bildirim kuyruğu, key iptal düğmesi ve modern menü eklendi.

## Menü açılıp kapanıyordu

Roblox `HttpService:JSONEncode({})` boş tabloyu `[]` olarak gönderir. İstemci `Hello`, `Open` ve `GetKeyLink` isteklerine boş `payload` ekliyordu; sunucu ise nesne beklediği için 400 `invalid_payload` döndürüyordu. Key içeren `Redeem` başarılı olup ardından açılış kaydı başarısız olduğunda menü kapanıyordu. Önceki test taklidi boş tabloyu nesne olarak yazdığı için bu hata kaçmıştı.

İstemci artık boş payload alanını göndermiyor. Sunucu eski istemcilerin boş dizisini de kabul ediyor; dolu diziler geçersiz kalıyor. Test JSON davranışı gerçek Roblox ile eşlendi. Geçici açılış kaydı ve bağlantı hataları mevcut erişimi hemen kapatmıyor; gerçek key iptali, hesap uyuşmazlığı ve süre dolması erişimi kapatmaya devam ediyor. Yetki kontrolü 15 saniyede bir başlıyor, doğrulanmış erişim 35 saniye boyunca yenilenemezse kapanıyor. Gecikmiş eski yanıtlar yeni oturumu etkileyemiyor.

## Key al tıklaması sonuç vermiyordu

Eski sekme kurulum durumunu yalnız açılışta kontrol ediyordu. Ayrıca ilk tıklama kampanya bağlantısını hazırlıyor, kullanıcıdan ikinci bir bağlantıya tıklamasını bekliyordu.

Kurulum bekleyen görünür sekmeler 15 saniyede bir tekrar kontrol ediyor; sekmeye dönüşte de durum yenileniyor. İlk tıklama sırasında ayrılan pencere, sunucu bağlantıyı döndürünce LootLabs'a gidiyor. Pencere engellenirse açıklama ve elle açılabilen bağlantı gösteriliyor. Tekrarlı tıklamalar üst üste istek başlatmıyor; hata veya sayfadan çıkış boş pencereyi kapatıyor. Canlı tarayıcıda tek tıklama ile LootLabs'ın açılması doğrulandı.

## Discord botu düşünmede kalıyordu

Discord komutu önce özel bekleme yanıtı gönderir, ardından key işlemi ve asıl mesaj güncellemesi yapılır. Worker'ın arka plan işi gerçek `waitUntil` bağlamına bağlandı. Discord'un ilk yanıtı henüz kaydetmemesi, geçici ağ ve oran sınırı hatalarında aynı mesaj tekrar deneniyor. Başarılı key işlemi tekrar çalıştırılmıyor. Bütün teslim denemeleri ortak 20 saniyelik süre kullanıyor; geç gelen yanıt da aynı key metnini taşır. Tanılama olayları key ve etkileşim tokenlarını yazmaz.

Bu düzeltme gerçek route kodu ve Vinext bağlamıyla yerel olarak doğrulandı. Yeni sürümle canlı Discord komutu gönderilmesi ayrıca doğrulanmalıdır; eski düşünme mesajı kendiliğinden tamamlanmayabilir.

## Görev sonrası ayrı key sayfası

LootLabs'ın mevcut kampanyasının hedef adresi düzenleme ekranında kilitli. Görev sonrası açtığı kök adres, geçerli tarayıcı oturumunu `/key` sayfasına yönlendirir. Postback henüz gelmediyse yeni sayfa tamamlanmayı bekler. Key yalnız sunucunun görev bildirimi doğrulandığında gösterilir.

Tarayıcı oturumu Secure, HttpOnly, host ile sınırlı cookie ile aktarılır. Cookie, key ve özel takip tokenı adres çubuğuna yazılmaz. Önceki IP bağlama, süre ve iptal kontrolleri kullanılır. `/portal` yeni key başlatma sayfasıdır; mevcut key oturumunun yönlendirmesi yeni bağlantı istemeyi engellemez.

Canlı kontrolde sonuç sayfasındaki istemci yönlendirmesi portal dönüşünde takıldı. Dönüş bağlantıları tam sayfa yükleyen standart bağlantılarla değiştirildi; yeni bağlantı akışı cookie yönlendirmesine takılmadan açılır.

## Doğrulamanın sınırları

İstemcinin okunabilir ve dağıtım kopyalarında 141'er taklit Roblox kontrolü geçti. Luau derlemesi ve AST eşitliği doğrulandı. Canlı API eski boş-dizi isteğine 200 döndürüyor; GitHub Raw dosyası yerel dağıtım dosyasıyla birebir eşleşiyor.

Backend ve web arayüzü testleri gerçek kaynak kodunu, SQLite ve taklit HTTP cevaplarını kullanır. Gerçek Roblox çok oyunculu oturumu, canlı Discord komut sonucu ve LootLabs reklam/insan doğrulaması tamamlanarak key teslimi bu raporla test edilmiş sayılmaz.

## Oturum ve ayarlar her çalıştırmada sıfırlanıyordu

Geçerli normal oturum tokenı ve beyaz listedeki ayarlar hesap/servis kapsamında saklanır. Yeni çalıştırma sunucuya yeniden doğrulatır; iptal edilmiş token erişim sağlamaz. Ham key ve yönetici oturumu saklanmaz. Dosya API'si sunan ortamda kayıt diske yazılır; bu API yoksa aynı çalışma ortamının belleği kullanılır. Geçici ağ kesintisi yeniden deneme için kaydı korur. Kayıtlı girişi unut ve Varsayılanlara dön denetimleri ayrı işlemlerdir.

## Otomatik basılı tuş ve menü

Main gelişmiş ayarlarında isteğe bağlı Sol Shift/Sol Ctrl/Boşluk, kilitliyken veya kilitli + ateş ederken modu bulunur. Menü, sohbet, odak kaybı, hedef kaybı, izin iptali ve kapanışta tuş bırakılır. Kapanış sırasında başarısız key-up tekrarının render döngüsü durduktan sonra kaybolması düzeltildi; sınırlı bağımsız tekrar ve eski denemelerin yeni tuş basışını bırakmasını engelleyen nesil kontrolü eklendi. Resmî Roblox VirtualInput API'si desteklenmiyorsa durum gösterilir. Gerçek oyun input desteği test edilmedi. AI modu eklenmedi.

Menü koyu yarı saydam yuvarlak kartlar, beyaz sekmeler ve kırmızı iOS tarzı anahtarlarla yenilendi. Logo 88391726271714 kullanılıyor. HTML/JPG önizlemeleri gerçek Roblox ekran görüntüsü değildir.

## Bildirimler kuyrukta kalıyordu

Canlı serviste key oluşturma, aktivasyon ve iptal başarılıydı; bildirimler kanal kontrolü aşamasında TypeError alıyor ve hiç teslim edilmiyordu. Asıl neden, Node test ortamının kabul ettiği `redirect: "error"` seçeneğinin workerd tarafından desteklenmemesi. Gerçek workerd ve dış ağı kapatılmış 200 HTTP yanıtıyla aynı hata yeniden üretildi; `manual` aynı istekte başarılı oldu. Bildirim GET/POST ve Discord yanıt PATCH istekleri manual kullanıyor; 3xx adresleri izlenmiyor ve bot kimlik bilgisi başka bir adrese taşınmıyor.

Discord yanıtındaki `dependencies.fetch(...)` global fetch fonksiyonunu yanlış `this` bağlamında çağırıyordu. Gerçek workerd testi bu durumda Illegal invocation hatasını da doğruladı; varsayılan adaptör global receiver'ı koruyor. Discord'un [User-Agent gereksinimi](https://docs.discord.com/developers/reference#user-agent) de uygulandı.

Kalıcı kuyruk olayı key işlemiyle aynı transaction içinde kaydeder; hata işlemi kaybetmez. Özel kanal kontrolü başarılı olmadan IP içeren mesaj gönderilmez. Mesaj key oluşturma/ilk kullanım/panel açılışı, IP, istemcinin bildirdiği Roblox kimliği ve Key'i iptal et düğmesi içerir. Tam key ve token mesajda yoktur. Kullanıcı kimliği Roblox OAuth ile doğrulanmış değildir. İptal, tüm bağlı oturumlarda sunucu tarafında hemen geçerlidir; açık istemci sonraki kontrolünde görür.

129 backend testi (Discord 34, Hub 32, LootLabs 31, bildirim 32) ve ek 2 gerçek workerd testi geçti. Portal 39 ve sonuç sayfası 26 komponent taklit kontrolü önceki akış doğrulamasına dahildir. Botun canlı slash komut yanıtı, gerçek Roblox input desteği ve LootLabs reklam tamamlaması bu kontrollerle test edilmiş sayılmaz.

11 Ekim 2026 17:14 UTC canlı API kurulum testinde key oluşturma, ilk kullanım ve panel açılışı bildirimlerinin üçü de yapılandırılmış özel Discord kanalına ulaştı. IP alanı ve iptal düğmesi vardı; kullanılan Roblox adı açıkça Klew_Test olarak bildirilmiş bir API testidir. Test key'i iptal edildi; eski token ile Hello 401 ve authorized:false döndü. Önceden bekleyen bildirimler de kuyruktan teslim edildi. Gerçek oyun oturumu veya LootLabs görevi tamamlanmış sayılmaz.
