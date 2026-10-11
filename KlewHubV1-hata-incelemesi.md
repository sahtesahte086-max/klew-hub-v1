# Klew Hub v1 — 11 Ekim 2026 hata incelemesi

Bu sürüm key portalı, Discord yönetimi ve Roblox HTTP istemcisindeki bağlantı hatalarını düzeltir. Yeni key sonucu `/key` sayfasında gösterilir.

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

İstemcinin okunabilir ve dağıtım kopyalarında 96'şar taklit Roblox kontrolü geçti. Luau derlemesi ve AST eşitliği doğrulandı. Canlı API eski boş-dizi isteğine 200 döndürüyor; GitHub Raw dosyası yerel dağıtım dosyasıyla birebir eşleşiyor.

Backend ve web arayüzü testleri gerçek kaynak kodunu, SQLite ve taklit HTTP cevaplarını kullanır. Gerçek Roblox çok oyunculu oturumu, canlı Discord komut sonucu ve LootLabs reklam/insan doğrulaması tamamlanarak key teslimi bu raporla test edilmiş sayılmaz.
