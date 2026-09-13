# beyin

Claude Code ve Codex'in **ortak okuduğu** bir hafıza katmanı.
Oturumlar arası devamlılık, projeler arası bağlam, harness'lar arası devir.

---

## Önce bunu oku: bu bir kurulum betiği değil

Bu repoyu klonlayıp bir ajana "şunu kur" demeni **istemiyorum.** Çalışır ama
yanlış şey olur.

Sebebi şu: burada işe yarayan şeylerin çoğu kod değil, **birinin gerçek
çalışma biçimine bakıp verilmiş kararlar.** Örnekler:

- Neyin "proje" sayılacağı — benimkinde `~/projeler/<x>`, ama bir kökün
  altındaki her klasörün ayrı proje olduğu bir istisna var, çünkü orada
  paralel yürüyen bağımsız araştırma bahisleri vardı
- Hangi dosyaların korunacağı — `data/` ve `manifests/` kilitli, `results/`
  ve `outputs/` serbest, çünkü ilki girdi ikincisi üretilen
- Çürütülmüş hipotez kaydının şeması — sahibinin makalesindeki
  `§6 Çürütülen hipotezler` tablosundan çıkarıldı, standart bir "negative
  results" bölümünden değil

Bunların hiçbiri senin için doğru olmayabilir. Senin projelerin başka yerde,
normların başka, korunması gerekenler başka.

**Önerim:** bu repoyu bir ajana ver ve *"bunu oku, sonra bana benim
çalışma biçimimi sor"* de. Aşağıda o konuşmanın soruları var. Kodu sonra
uyarlarsın — asıl iş sorularda.

> Bu yaklaşımı bilerek seçtim. İlham aldığım [beyin.md](https://avenox.lol/beyin.md)
> kendini kuran bir spesifikasyon olarak yazılmış: URL'i ajana veriyorsun, o
> her şeyi kuruyor. Zekice ve etkileyici. Ama sonuç, birinin çalışma biçiminin
> başkasının makinesine kopyalanması oluyor. Fikri ondan aldım, dağıtım
> biçimini almadım.

---

## Ne yapıyor

| ne zaman | ne olur |
|---|---|
| oturum açılışı | son durum, açık işler, bağlı projeler, o an çalışan kardeş oturumlar, bekleyen devir notu bağlama girer |
| her istem | canlı durum tazelenir (oturum başına ayrı dosya) |
| bağlam dolarken | devir notu arka planda kendiliğinden üretilir |
| oturum kapanışı | döküm özetlenir → karar, çürütülmüş hipotez, açık iş ayıklanır |
| periyodik | markdown görünümü derlenir (Obsidian vault olarak açılır) |
| her araç çağrısı | korunan yollara yazma reddedilir |

Ve iki harness da aynı store'u okur. Claude'da limit biterse Codex kaldığın
yerden devam eder.

## Neden iki harness

Çünkü hook protokolleri **aynı**. İkisi de `hook_event_name`,
`hookSpecificOutput`, `additionalContext`, `cwd`, `transcript_path` gönderiyor;
ikisinde de `SessionStart`, `SessionEnd`, `UserPromptSubmit`, `PreCompact`,
`PreToolUse`, `SubagentStart` var.

Tek script seti, iki ince config bağlantısı:

```
~/.claude/settings.json   →  hooks.<Event>[].hooks[].{type,command,timeout}
~/.codex/hooks.json       →  {"description":..., "hooks": {<Event>: [...]}}
```

Codex tarafında hook'lar **içerik hash'iyle** güvenilir işaretleniyor
(`config.toml` içinde `[hooks.state]`), yani bir kez elle onaylanıyor.

---

## Sekiz değişmez kural

Kod değişir, bunlar kalır. Kendi sürümünü yazacaksan taşınması gereken kısım burası.

**K1 — Hook script'leri ince ve sabit.**
Codex hook'un *içeriğinin* hash'ini tutuyor. Script değişirse onay tekrar
çıkar. O yüzden hook dosyaları iki satır ve bir daha asla değişmiyor:
`exec "$HOME/beyin/bin/beyin-capture" claude`. Bütün mantık `bin/` altında.
*Bu kural beş faz boyunca kendini ödedi: `bin/` defalarca baştan yazıldı,
Codex onayı bir kez soruldu.*

**K2 — `cwd` allowlist, ilk günden.**
Hook'lar kullanıcı seviyesinde olduğu için **her yerde** ateşler: `/tmp`,
klonladığın repo, üç dakikalık deneme klasörü. Sadece tanınan proje kökleri
kaydedilir. Sonradan eklenecek bir iyileştirme değil — yoksa store ilk günden
çöple dolar.

**K3 — Proje tanıtımı store'a girmez.**
`CLAUDE.md` ve `AGENTS.md` zaten `cwd`'de otomatik okunuyor. Store'un işi
hiçbir dosyanın kapsamadığı dört şey: son oturum, kenarlar, karar defteri,
kardeş oturumlar. *Bu sınır ilk fırsatta aşınır; kural olarak yazmazsan
bir hafta boyunca iki harness'ın hazır sunduğu özelliği yeniden inşa edersin.*

**K4 — Yakalama append-only.**
İki CLI × alt ajanlar = eşzamanlı yazıcılar. Markdown'a read-modify-write veri
kaybettirir. Ham katman JSONL + `flock`. Markdown türetilmiştir.

**K5 — Yapı sabit, kelime dağarcığı serbest.**
Kenar kaydı `(kaynak, hedef, etiket, neden, kanıt)` — kolonlar sabit,
`etiket` serbest metin. Hayat akıyor: bugünkü kategorilerin iki ay sonra
geçerli olmayabilir, ama tablo geçerli kalır.

**K6 — Okuma yolu bütçeli.**
Enjekte edilen her şey context'ten yer yer. Sahibinin store'unda 400+ karar
var; hepsi zaten sığmaz. Kural: son oturum ve kenarlar koşulsuz enjekte edilir,
karar ve hipotezler **sadece sayı + sorgu komutu** olarak geçer. Sayı vermek
ajana iki şey söyler: bilgi var, ve sorgulamaya değer mi.

**K7 — Özyineleme kalkanı.**
Özetleyici `claude -p` / `codex exec` çağırıyor; o çağrı da hook ateşliyor.
Kalkansız sonsuz döngü. İki katman: `BEYIN_NO_CAPTURE=1` ve CLI'ı allowlist
dışından (`/tmp`) çağırmak.

**K8 — Yankı deterministik elenir.**
Sistem kendi çıktısını yeniden yutabilir: ajan hafızadan okuduğunu kendi
cevabında tekrarlayınca özetleyici bunu yeni bulgu sanar. Prompt uyarısı
**olasılıksal** — model bazen uymuyor. Yazmadan önce store ile karşılaştır
(benzerlik > 0.80). Yankı tanım gereği tekrardır; bu filtre modele güvenmez.

---

## Kendine göre uyarlamak için: konuşmanın soruları

Kodu kurmadan önce ajanınla bunları konuş. Bu sistemin değerli kısmı, sahibinin
bu sorulara verdiği cevaplardı.

**Kapsam**
1. Projelerin nerede duruyor? Tek kök mü, birkaç mı?
2. Bir kökün altındaki klasörler ayrı proje mi, yoksa tek projenin parçaları mı?
3. Hangi klasörlerde çalışıyorsun ama **kaydedilmesini istemiyorsun**?

**Ne hatırlanmalı**
4. Şu an elle tuttuğun bir kayıt var mı? (karar günlüğü, deney defteri,
   TODO dosyası) — varsa **onun şemasını al**, yenisini icat etme
5. Bir projeyi altı ay sonra açtığında ilk bilmen gereken üç şey ne?
6. Sende "asla kaybolmamalı" olan kayıt türü hangisi?

**Projeler arası**
7. Hangi projelerin birbirine değiyor, ve nasıl? (ortak veri, ortak servis,
   biri diğerinin çıktısı, aynı kurum)
8. Bir projede aldığın karar hangi başka projeyi etkiliyor?

**Harness'lar arası**
9. Ajan değiştirdiğinde elle ne yeniden anlatıyorsun? — o liste devir notunun
   şemasıdır
10. İkisini aynı anda çalıştırıyor musun? Farklı iş mi yapıyorlar, aynı mı?

**Sınırlar**
11. Hangi dosyalara bir ajan **asla** yazmamalı? Neden?
12. Alt ajanlara gözetimsiz yazma yetkisi verecek misin?

**Prosedür**
13. Hangi işi sürekli aynı şekilde tekrarlıyorsun? — skill adayları
14. Bir ajanı en çok hangi konuda düzeltiyorsun?

---

## Ne doğrulandı, ne doğrulanmadı

Dürüst olmak faydalı, çünkü bu sistem sessiz çalışıyor — bozulduğunu fark
etmek zor.

**Doğrulandı** (gerçek oturumlarla, iki harness'ta)
- Hook'ların ateşlemesi ve aynı wire formatını kullanması
- Allowlist: proje içi kaydediliyor, `/tmp` düşüyor
- Enjeksiyon: ajan bağlı projeyi ve geçmiş kararları kendiliğinden söyledi
- Devir: bir harness'ta üretilen not diğerinde açılışta alındı
- Yazma koruması: korunan yola yazma iki harness'ta da reddedildi, dosya md5'i
  değişmedi
- Özyineleme ve yankı kalkanları

**Doğrulanmadı**
- Uzun süreli kullanım. Sistem test edildi, **yaşanmadı**
- Maliyet: oturum başına küçük bir özetleme çağrısı, haftalık toplamı ölçülmedi
- Kabuk komutu koruması **sezgisel** — dolambaçlı bir komut kaçabilir.
  Kaza koruması, kilit değil. Gerçek kilit: `chmod -w`
- Codex'in `asyncRewake`'i (canlı oturuma push) hiç denenmedi

**Bilinen kısıt:** derleyici bir oturum geride kalır — özetleme arka planda
sürerken derleme başlar. Veri kaybı yok, görünüm gecikir.

---

## Yine de kurmak istersen

```bash
git clone <repo> ~/beyin && cd ~/beyin
cp ornek/ayarlar.md ./ayarlar.md      # proje köklerini ve korunan yolları düzenle
cp ornek/graph.md   ./graph.md        # kendi kenarlarını yaz
mkdir -p store state
```

Claude Code — `~/.claude/settings.json` içine:

```json
{"hooks": {
  "SessionStart":     [{"hooks":[{"type":"command","command":"$HOME/beyin/hooks/claude.sh","timeout":10}]}],
  "UserPromptSubmit": [{"hooks":[{"type":"command","command":"$HOME/beyin/hooks/claude.sh","timeout":5}]}],
  "SessionEnd":       [{"hooks":[{"type":"command","command":"$HOME/beyin/hooks/claude.sh","timeout":10}]}],
  "PreCompact":       [{"hooks":[{"type":"command","command":"$HOME/beyin/hooks/claude.sh","timeout":10}]}],
  "PreToolUse":       [{"hooks":[{"type":"command","command":"$HOME/beyin/hooks/claude.sh","timeout":5}]}],
  "SubagentStart":    [{"hooks":[{"type":"command","command":"$HOME/beyin/hooks/claude.sh","timeout":5}]}]
}}
```

Codex — `~/.codex/hooks.json` içine aynı olaylar, `codex.sh` ile,
`{"description": "...", "hooks": {...}}` sarmalayıcısıyla. Sonra bir kez
`codex` açıp hook'lara **Trust all** de. (Codex `SessionEnd` timeout'unu
3 saniyeye kırpar — oturum sonu işi bloke edemez, arka plana atılmalı.)

Sonra: `~/beyin/bin/beyin doktor`

Geçmişini içeri almak istersen: `~/beyin/bin/beyin-import --dry-run`

**Not:** `store/` senin gerçek çalışmanın özetlerini tutar — proje adları,
dosya yolları, kararlar, bulgular. Yayınlamadan önce iki kez düşün.
`.gitignore`'da olması yetmez; git geçmişi de tutar.

---

## Komutlar

```
beyin durum                     tüm projeler
beyin gecmis  <proje> [n]       son n oturum
beyin karar   <proje> [n]       kararlar
beyin hipotez <proje> [n]       çürütülmüş hipotezler
beyin ara     <kelime>          her şeyde ara
beyin canli                     şu an çalışan oturumlar
beyin baglam  <proje>           açılışta ne enjekte ediliyor
beyin devir   <proje>           devir notu üret
beyin gorevlendir <proje> "..."  alt ajanı o projede çalıştır
beyin hipotez-ekle <proje> ...  çürütülmüş hipotezi elle kaydet
beyin skill-bagla <isim>        skill'i iki harness'a da bağla
beyin derle                     markdown görünümünü yenile
beyin doktor                    tanı
beyin yayinla [--kuru]          kodu yayın deposuna taşı (önce sızıntı tarar)
```

### `beyin yayinla` hakkında

Bu repo ile senin kendi `~/beyin`'in **iki ayrı depo.** Kasıtlı: hafıza deposu
proje adlarını, kararları, yayınlanmamış bulguları tutar; yayın deposu sadece
kodu. Aralarındaki köprü bu komut.

Tarama terimlerini elle yazmazsın — kendi verinden türetilir: proje
anahtarları, diskteki gerçek klasör adları, ev yolun, git e-postaların.
Yeni bir proje eklediğin gün liste kendini günceller. Kaynak kirliyse
kopyalama hiç yapılmaz.

`ayarlar.md > Yayin taramasi` ile terim eklersin; `- !kelime` yanlış
pozitifleri eler.

---

## Kaynak

Fikir [beyin.md](https://avenox.lol/beyin.md) / [avenoxbeyin](https://github.com/avenoxai/avenoxbeyin)
projesinden geliyor — "hafıza disiplin değil mekanizma olmalı" tezi oradan.
Bilgi derleme deseni [Karpathy'nin LLM bilgi tabanı notundan](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

Buradaki farklar: hafıza vault'un içine değil **üstüne** kuruldu (her projede
çalışır), iki harness ortak store okur, derleyici LLM kullanmaz, ve dağıtım
biçimi kurulum betiği değil konuşma daveti.

Türkçe yazıldı çünkü öyle kullanılıyor.
