# Proje Kenarları

Bu dosya **elle yazılır.** Derleyici üzerine yazmaz.
`SessionStart` ve `SubagentStart`'ta, açtığın projeye değen satırlar bağlama girer.

**Yapı sabit, kelime dağarcığı serbest.** `etiket` ne istersen olabilir.
`<->` çift yönlü · `->` tek yönlü (sadece kaynak tarafında görünür)

| kaynak | yön | hedef | etiket | neden | kanıt |
|---|---|---|---|---|---|
| [[api]] | `<->` | [[web]] | ortak-sozlesme | Web'in testleri API'nin şemasına bağlı; şema değişince ikisi birden kırılır. | `api/openapi.json` |
| [[makale]] | `<->` | [[deneyler]] | makale-deney | Kod ve koşular deneyler reposunda, nihai rapor makale klasöründe. | `makale/RAPOR.md` |
