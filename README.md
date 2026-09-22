# sapancadayim.com

Sapancadayım uygulamasının tanıtım sitesi. GitHub Pages ile yayınlanıyor,
alan adı `CNAME` dosyasında.

## Yapı

- `index.html` — ana tanıtım sayfası
- `style.css` — dört sayfanın paylaştığı palet ve tipografi. Renk değişikliği burada yapılır.
- `assets/` — logo ve sayfa görselleri
- `destek/`, `gizlilik/`, `indir/` — alt sayfalar

## Bekleyen görseller

Üçü de `index.html` içinde kesikli çerçeveli yer tutucu olarak duruyor. Gerçek
dosya geldiğinde `.slot` satırı silinip hemen altındaki yorumlu `<img>` açılır:

| Dosya | Nereye |
|---|---|
| `assets/ekran.png` | Hero'daki telefon çerçevesi |
| `assets/sapanca.jpg` | Özellik kutularının üstündeki geniş şerit |
| `assets/qr.png` | QR bölümü. Kod `https://sapancadayim.com/indir/` adresine bakmalı |

## Yayın günü

`index.html` içinde `YAYIN GÜNÜ` diye aranır, dört yer çıkar — üçü numaralı:

1. Mağaza rozetlerindeki `<small>Yakında</small>` — silinir, `<span class="badge">`
   yerine mağaza adresine giden `<a class="badge">` gelir
2. QR kutusundaki yer tutucu — `assets/qr.png` ile değiştirilir
3. Hero'daki "Yakında" etiketi — silinir

Numarasız dördüncü: `<style>` içindeki `.qr-box { opacity: 0.55; }` kuralı ve
QR bölümündeki "henüz yayında değil" notu (`.qr-note`) — ikisi de silinir.

Ayrıca mağaza rozetlerindeki Apple/Google logoları elle çizilmiş SVG; resmî
rozet dosyalarıyla değiştirilmeli.

## Yerelde çalıştırma

```bash
python3 -m http.server 8787
```

Sonra `http://localhost:8787/`.

## Tasarım kararları

Spec ve plan `docs/superpowers/` altında. Site tek temalı — beyaz zemin marka
kimliğinin parçası, `prefers-color-scheme` kuralı bilerek yok.
