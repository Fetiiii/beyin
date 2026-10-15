# Ayarlar

Bu dosya **elle yazılır.** Derleyici üzerine yazmaz.
`~/beyin/ayarlar.md` olarak kopyala ve kendine göre değiştir.

## Proje kökleri

Hook'lar her yerde ateşler; sadece bu köklerin altı kaydedilir.
Gerisi sessizce düşer. (Yoksa `/tmp` ve üç dakikalık deneme klasörleri
hafızayı çöple doldurur.)

- ~/projeler

## Bölünmüş kökler

Normalde `<kök>/<klasör>` bir projedir ve alt klasörleri ona sayılır.
Burada listelenenler istisnadır: altındaki her klasör ayrı proje olur.
Kökün *kendisinden* çalışılan oturumlar kök projeye (portföy) yazılır.

- research

## Korunan yollar

Bu yollara yazma girişimi `PreToolUse` hook'uyla **reddedilir** — prompt
uyarısı değil, harness seviyesinde engel.

Satır bir klasör adıysa yolun herhangi bir bileşeni o adsa eşleşir.
`*` içeriyorsa dosya adına glob uygulanır.
`~/` ya da `/` ile başlıyorsa mutlak yol öneki sayılır.
`arac:` ile başlıyorsa yalnız kesin mekanizmalarla korunur (Write/Edit aracı ve
yama başlıkları), kabuk komutu sezgiseliyle değil.

- ~/beyin/store
- ~/beyin/events.jsonl
- arac: ~/beyin/bin
- arac: ~/beyin/skills
- arac: ~/beyin/hooks

### Veri setlerini korumak istersen — önce bunu oku

İlk sürümde `data`, `datasets`, `manifests`, `*.sha256`, `*frozen*` düz desen
olarak buradaydı. İki ayda **292 engelleme** üretti, büyük kısmı yanlış pozitif:
`hashlib.sha256` çağrısı `*.sha256`'ya, bir HuggingFace indirme URL'si
(`.../datasets/...`) `datasets`'e takıldı; `./data/` geçen her `>` içeren komut
engellendi. Sonunda kaldırıldı.

Yine de istersen `arac:` önekiyle ekle: Write/Edit ile o klasöre yazmayı engeller,
kabuk komutlarına karışmaz.

    - arac: data
    - arac: manifests

Gerçek kilit dosya sisteminde durur: `chmod -w` ya da `chattr +i`.
