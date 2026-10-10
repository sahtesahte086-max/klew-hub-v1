# Klew Hub v1

Kırmızı tema ve `131548509676467` logo ID'siyle Roblox test menüsü. ESP başlangıçta yalnız karakterin vücudunu dolgu ve konturla vurgular; kutu, isim, can ve mesafe isteğe bağlıdır. Aimbot'un varsayılan kilit tuşu **E**, menü tuşu **RightShift**.

ESP sekmesindeki **Takım arkadaşlarını gizle** seçeneği başlangıçta açıktır. Kapatırsan takım arkadaşlarının ESP'si de görünür; aimbot bu seçenekten bağımsız olarak kendi takımına kilitlenmez. ESP yenileme hızı varsayılan **60 Hz**; menüden **30–240 Hz** seçilebilir ve oyun FPS'iyle sınırlıdır. Baş ekran dışına çıkıp gövde görünür kaldığında vücut vurgusu korunur. Yalnız vücut vurgusunda gereksiz kutu projeksiyonları hesaplanmaz.

## HTTP yükleyici

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/sahtesahte086-max/klew-hub-v1/main/KlewHubV1.lua"))()
```

Bu biçim `game:HttpGet` ve `loadstring` sunan istemci ortamı gerektirir. Standart StarterPlayer LocalScript'inde desteklenmez. Roblox Studio'da `KlewHubV1.lua` içeriğini **StarterPlayer > StarterPlayerScripts > LocalScript** içine doğrudan yapıştır.

## Dağıtım

`KlewHubV1.lua`, yerel değişken adları karıştırılmış tek satırlık dağıtım dosyasıdır. İstemciye dağıtılan kod okunabilir ve geri çözümlenebilir; Raw bağlantısı gizlilik sağlamaz. Kullanım ve dağıtım koşulları `LICENSE.txt` içindedir. Okunabilir geliştirme kaynağı bu depoda yayımlanmamıştır.

## Doğrulama

Luau derlemesi ve 55 tam script akış kontrolü geçti. Takım gizleme seçeneği, dost hedefin seçilmemesi, takım değişiminde kilidin bırakılması, tek karede konum yenileme ve 144 FPS'te 60 Hz yenileme ritmi kontrol edildi. Testler Roblox API sınırlarını taklit eder; gerçek çok oyunculu Studio çalışmasını doğrulamaz.
