# Skill'ler

Tek kaynak burası. İki harness da symlink ile buraya bakar:

```bash
ln -sfn ~/beyin/skills/<isim> ~/.claude/skills/<isim>
ln -sfn ~/beyin/skills/<isim> ~/.codex/skills/<isim>
```

**Klasörün tamamını linkleme** — `~/.codex/skills/.system` altında Codex'in
kendi sistem skill'leri var (imagegen, skill-creator, review-agent...), ve
`~/.claude/skills` altında da başka linkler olabilir. Skill başına link.

Kurulum için: `~/beyin/bin/beyin skill-bagla <isim>`

## Ne skill olur, ne olmaz

**Skill = nasıl yapılır** (prosedür). Nadiren değişir, elle/öğreterek yazılır.
**Hafıza = ne oldu** (olgu). Sürekli değişir, otomatik yakalanır.

Skill'in içine olgu koyma. *"X projesinde veri şurada"* bir skill'e yazılırsa,
taşındığı gün yalan söylemeye başlar. O bilgi store'a aittir.

## Nasıl yazılır

Önceden yazma. İşi yap, düzelt, oturunca kaydettir:

> *"Bu işi üçüncü kez aynı şekilde yaptık. Yöntemi bir skill'e çevir."*

Skill'ler aynı zamanda hafızanın **kasıtlı yazma yolu**: "hipotez çürütme
raporu" skill'i çalıştığında hem raporu üretir hem store'a alıntılanabilir
kaydı düşer.
