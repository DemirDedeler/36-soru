# 36 Soru: Uygulama nasıl çalışıyor?

Bu not, oyunun kodunu ([index.html](index.html)) ileride kendin okuyup değiştirebilmen için yazıldı.

## Büyük resim

Oyunun tamamı **tek bir dosya**. Sunucu, veritabanı ya da kullanıcı hesabı yok. Tarayıcı dosyayı indirir ve her şey telefonun içinde çalışır. Dosyanın üç parçası var:

| Parça | Görevi | Benzetme |
|---|---|---|
| **HTML** | Sayfada ne olduğunu söyler (başlık, buton, kart) | İskelet |
| **CSS** (`<style>` içi) | Nasıl göründüğünü söyler (renk, yazı tipi, kart dönme animasyonu) | Kıyafet |
| **JavaScript** (`<script>` içi) | Ne olduğunu ve neye tepki verileceğini söyler (karta dokununca dön, sayaç say) | Beyin |

Aslında HTML'in büyük kısmı da JavaScript tarafından üretiliyor. Sayfada başta sadece boş bir `<div id="app">` var, gerisini kod dolduruyor.

## Temel fikir: "durum" ve "çizim"

Uygulamanın kalbi iki şey:

1. **`S` adlı bir nesne (state/durum).** Oyunun o anki her bilgisi burada durur:
   - `screen`: hangi ekrandayız (`start`, `level`, `play`, `eye`, `end`)
   - `names`: iki oyuncunun ismi
   - `level`: kaçıncı seviye (0, 1, 2)
   - `order`: her seviyedeki kartların sırası (pas deyince bu sıra değişir)
   - `pos`: seviyenin kaçıncı kartındayız
   - `flipped`: kart açık mı
   - `passes`: kaç kez pas denildi
   - `timer`: çalışan sayaç varsa kalan süre

2. **`render()` fonksiyonu.** `S`'ye bakar, ona uygun ekranı (Start, Level, Play, Eye, End fonksiyonlarından birini) HTML olarak üretir ve sayfaya basar.

Akış hep aynı: **kullanıcı bir şeye dokunur → `act()` fonksiyonu `S`'yi değiştirir → `render()` ekranı yeniden çizer.** Bir hatayı ararken önce "S'de ne yazıyordu?" diye sormak çoğu zaman cevabı verir.

## Dokunmalar nasıl yakalanıyor?

Her butonda `data-a="next"`, `data-a="pass"` gibi bir etiket var. `bind()` fonksiyonu, her çizimden sonra bu butonlara "tıklanınca `act(etiket)` çalıştır" der. `act()` içinde de her etiket için ne yapılacağı yazıyor:

- `start`: isimleri alır, desteleri hazırlar, 1. seviye ekranına geçer
- `flip`: kartı açar
- `next`: sonraki karta geçer; seviye bittiyse sonraki seviyeye, 36 bittiyse göz teması ekranına
- `pass`: o anki kartı seviyenin destesinden çıkarıp sonuna ekler
- `back`: bir önceki karta döner
- `timer` / `eye`: sayacı başlatır veya durdurur
- `restart`: her şeyi sıfırlar

## Sorular nerede?

Script'in en başında:

- `LEVELS`: üç seviyenin adı ve kısa açıklaması
- `Q`: 36 sorunun listesi, sırayla. **Bir soruyu düzeltmek için sadece bu metni değiştirmen yeterli.**
- `TIMED`: hangi soruda sayaç olacağı. `{ 10: 240 }` "11. soruda 240 saniye" demek (programlamada sayma 0'dan başladığı için 11. soru 10 numaradır).
- `EYE_SECONDS`: son göz teması süresi (240 saniye = 4 dakika)

## Küçük ama işe yarar mantıklar

- **Sıra kimde?** Şimdiye kadar geçilen kart sayısı çiftse 1. kişi, tekse 2. kişi okur (`done() % 2`). Böylece ayrıca saklamaya gerek kalmıyor.
- **Sayaç neden kaymıyor?** Sayaç "her saniye 1 azalt" diye çalışmıyor; bitiş anını not alıp "şu an ile bitiş arasında ne kadar var" diye hesaplıyor. Telefon bir anlığına yavaşlasa bile süre doğru kalır.
- **Kaldığın yerden devam:** Her adımda `S`, tarayıcının küçük deposu olan `localStorage`'a yazılır (`save()`). Sayfa yeniden açılınca `load()` bunu okur ve "devam edin" butonu çıkar. Bu veri **sadece o telefonda** durur, kimseye gönderilmez. Gizli sekmede veya tarayıcı verisi silinince kaybolur; kod bu durumda da hata vermeden çalışacak şekilde yazıldı (`try/catch`).
- **Ekran kararmasın:** `keepAwake()`, destekleyen tarayıcılardan ekranı açık tutmalarını ister. Desteklemeyen tarayıcıda sessizce hiçbir şey yapmaz.
- **Titreşim:** Süre dolunca `buzz()` telefonu titretir (iPhone Safari bunu desteklemez, sorun değil).
- **En alttaki `window.claude?.hot` satırları:** Sadece Claude'un yayın ortamında, sayfa güncellenince oyunun kaldığı yerden devam etmesi için. Başka bir yere taşıyınca etkisizdir, silinebilir.

## Sık yapacağın değişiklikler

| İstediğin | Nereye bakmalı |
|---|---|
| Soru metnini düzeltmek | `Q` listesi |
| Seviye adını değiştirmek | `LEVELS` |
| Sayaç süresi | `TIMED` ve `EYE_SECONDS` |
| Renkler | `<style>` en üstündeki `:root` bloğu (`--felt`, `--brass` vb.) |
| Yazı tipleri | En üstteki Google Fonts linki ve `--display`, `--body` |
| Başlangıç ekranındaki yazılar | `Start()` fonksiyonu |

## Bir şey bozulursa

1. Telefonda değil bilgisayarda aç, sağ tık → **İncele (Inspect)** → **Console** sekmesine bak. Kırmızı yazılar hatanın nerede olduğunu söyler.
2. Değişiklikten sonra bozulduysa en sık sebepler: bir tırnak işaretinin (`"`) kapanmaması veya listede iki soru arasındaki virgülün unutulması.
3. "Devam edin" garip bir yere götürüyorsa kaydedilmiş eski durum yeni kodla uyuşmuyor olabilir. Kodda `KEY = "36soru.v1"` değerini `v2` yapmak herkesin eski kaydını yok sayar.

## Yayına alırken

Bu dosya Claude'un yayın ortamı için yazıldı; o ortam başına standart HTML iskeletini (`<!doctype html>`, `<head>`) kendisi ekliyor. GitHub Pages veya Netlify'a koyarken bu iskeleti dosyaya eklememiz gerekecek. O adımı birlikte yaparız.
