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

- data
- datasets
- manifests
- archives
- mlruns
- checkpoints
- *.sha256
- ~/beyin/store
- ~/beyin/events.jsonl

*`results/` ve `outputs/` bilerek listede yok: onlar üretilen şeyler.*
