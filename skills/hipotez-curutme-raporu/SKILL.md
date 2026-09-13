---
name: hipotez-curutme-raporu
description: Bir hipotez test edilip reddedildiğinde çürütme kaydını yazar ve hafızaya işler. "hipotez çürüdü", "bu iddia öldü", "negatif sonucu kaydet", "çürütme raporu" dendiğinde ya da bir deney bir hipotezi elediğinde kullan.
---

# Hipotez çürütme raporu

Bir hipotez elendiğinde bu **kayıp değil, bulgudur.**

Bu skill, makalesinde çürütülen hipotezleri ayrı bir bölümde numaralı bir
defter olarak tutan bir araştırmacının pratiğinden çıkarıldı. O defterin iki
kuralı vardı: hiçbir kayıt silinmez, ve hipotezin kime ait olduğu yazılır —
çoğu yazarın kendisine ait.

**Kendi pratiğine uyarla.** Aşağıdaki kurallar birinin gerçek çalışma
biçiminden geliyor; seninki farklıysa şablonu değiştir.

## Ne zaman tetiklenir

- Bir deney/ölçüm bir iddiayı geçersiz kıldığında
- Kullanıcı "bu hipotez öldü", "bu yol kapandı", "çürüdü" dediğinde
- Bir düzeltmenin kendisi sonradan yanlış çıktığında (bu da bir kayıttır)

Emin değilsen **yaz**. Fazladan kayıt zararsız, kayıp kayıt geri gelmez.

## Kurallar

1. **Silme, yeniden numaralama.** Defter büyür, kısalmaz. Bir kayıt sonradan
   yanlış çıkarsa üstüne yeni kayıt eklenir, eskisi durur.
2. **Kim öne sürdü, yaz.** Kullanıcının kendi iddiasıysa öyle işaretle
   ("benim tahminim", "benim varsayımım"). Bu bir kusur değil, dürüstlük.
3. **Somut sayı ver.** "Başarısız oldu" değil: *"0/5000 gradient adımı"*,
   *"%85'i çekirdek aşırı"*, *"2.91× → 1.52×"*.
4. **Yerine ne kondu, yaz.** Çürütmek yarım iş; yerine geçen iddia varsa o da
   kayda girer. Yoksa "yok" yaz.
5. **Kanıt yolu ver.** Dosya, bölüm numarası ya da deney kimliği:
   `results/kosu-12.csv`, `§4.3`, `E2`.
6. **Yumuşatma.** "Beklendiği kadar iyi sonuç vermedi" değil, **"ÖLDÜ"**.
7. **Ölçüm talimatı yener.** Kullanıcının verdiği talimat ölçümle çelişiyorsa
   ölçüm kazanır — ve çelişkinin kendisi rapora yazılır.

## Çıktı

İki parça üret.

### 1. Rapor (kullanıcıya)

```markdown
## Çürütüldü: <iddianın kısa hâli>

**İddia** (kim öne sürdü): "<orijinal ifade, alıntı olarak>"
**Nasıl test edildi:** <deney, koşul, kaç koşu>
**Sonuç:** ÖLDÜ. <somut sayılarla neden>
**Yerine:** <yeni iddia, ya da "yok">
**Kanıt:** <dosya / bölüm / deney kimliği>
```

### 2. Hafıza kaydı

```bash
~/beyin/bin/beyin hipotez-ekle <proje> \
  --iddia "..." --kim "kullanici|ajan|kaynak" \
  --test "..." --red "..." --yerine "..." --kanit "..."
```

Bu kayıt proje klasörü silinse bile yaşar ve sonraki oturumlarda
`beyin hipotez <proje>` ile geri gelir.

## Yapma

- Kanıt yolu olmadan kayıt açma
- Birden fazla iddiayı tek kayda sıkıştırma — her biri ayrı satır
- Henüz test edilmemiş bir şüpheyi çürütülmüş gibi yazma
- Daha önce kaydedilmiş bir çürütmeyi tekrar yazma; önce
  `beyin hipotez <proje>` ile bak
