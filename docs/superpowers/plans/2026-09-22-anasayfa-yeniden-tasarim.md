# sapancadayim.com Ana Sayfa Yeniden Tasarımı — Uygulama Planı

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `sapancadayim.com` ana sayfasını tek ekranlık "yakında" duyurusundan; hero, görsel şerit, özellik kutuları, QR bölümü ve SSS içeren bir tanıtım sayfasına dönüştürmek — telefon ekranı, Sapanca fotoğrafı ve QR yer tutucu olarak.

**Architecture:** Derleme adımı yok. Palet ve tipografi değişkenleri kökteki `style.css`'e taşınır, dört HTML sayfası da onu bağlar. Ana sayfanın kendi düzen kuralları `index.html` içindeki `<style>` bloğunda kalır. Gömülü base64 logo `assets/logo.png` dosyasına çıkarılır. Üç görsel alanı, gerçek dosya gelince tek satır değişiklikle açılacak ortak bir yer tutucu bileşeni kullanır.

**Tech Stack:** Statik HTML + CSS. GitHub Pages (`main` dalının kökü). Google Fonts: Baloo 2, Source Sans 3. Derleme aracı, paket yöneticisi, JavaScript çerçevesi YOK.

**Spec:** `docs/superpowers/specs/2026-09-22-anasayfa-yeniden-tasarim-design.md`

## Global Constraints

- Depo: `/Users/ekremcansirin/sapancadayim-web`, dal `main`, uzak `github.com/ekremcansirin/sapancadayim-web` (public).
- **Tek tema.** Koyu tema yok, `prefers-color-scheme` kuralı yazılmaz. Beyaz zemin marka kararıdır.
- Palet: `--ground #FFFFFF`, `--ink #0C2A22`, `--muted #5E7369`, `--brand #059669`, `--line #E4EBE6`, `--wash #F4FAF7`.
- Tipografi: `--display "Baloo 2"` (600/800), `--body "Source Sans 3"` (400/600). Gövde 17px, satır yüksekliği 1.6.
- Dil `tr-TR`. Tüm metin Türkçe, kesme işareti `&rsquo;` ile yazılır (mevcut sayfalardaki kalıp).
- **Uygulama mağazada YOK.** Mağaza rozetleri "Yakında" der, hiçbir mağaza bağlantısı verilmez, QR kutusu soluk durur.
- Yeni görseller `assets/` klasöründe dosya olarak durur. Base64 gömme yapılmaz.
- QR hedefi: `https://sapancadayim.com/indir/`.
- Metin/zemin karşıtlığı WCAG AA (normal metin 4.5:1) sağlanmalı.
- JavaScript yazılmaz. SSS aç-kapa `<details>`/`<summary>` ile.
- Her görevin sonunda commit. Commit mesajları Türkçe, `feat:` / `refactor:` / `fix:` öneki ile.

## Dosya yapısı

| Dosya | Sorumluluk |
|---|---|
| `style.css` | **Yeni.** Yalnızca paylaşılan tasarım değişkenleri ve sıfırlama (reset). Düzen kuralı içermez. |
| `assets/logo.png` | **Yeni.** `index.html`'den çıkarılan logo. |
| `index.html` | **Değişecek.** Ana sayfa: düzen kuralları + içerik. |
| `destek/index.html` | **Değişecek.** Yalnızca `:root` bloğu `style.css` bağlantısıyla değiştirilir. |
| `gizlilik/index.html` | Aynı. |
| `indir/index.html` | Aynı + gömülü logo `assets/logo.png` ile değiştirilir. |

## Doğrulama yöntemi

Bu depoda test çerçevesi yok ve statik bir site için kurmak gereksiz. Her görev
iki aşamada doğrulanır:

1. **Makine kontrolü** — `curl` + `grep` ile dosyanın beklenen içeriği taşıdığı.
2. **Göz kontrolü** — yerel sunucuda tarayıcıda açıp 390px (telefon) ve 1280px
   (masaüstü) genişlikte bakmak.

Yerel sunucu her görevde aynı şekilde başlatılır:

```bash
cd /Users/ekremcansirin/sapancadayim-web && python3 -m http.server 8787
```

Adres: `http://localhost:8787/`. İş bitince sunucu durdurulur.

---

### Task 1: Ortak `style.css` ve logo dosyası

Düzen değişmeden önce altyapı ayrılır. Bu görev bittiğinde site **gözle aynı
görünmeli** — tek fark gri metnin bir tık koyulaşması.

**Files:**
- Create: `style.css`
- Create: `assets/logo.png`
- Modify: `index.html` (`:root` bloğu, ikinci `<style>` bloğu, logo `<img>`)
- Modify: `destek/index.html`, `gizlilik/index.html`, `indir/index.html` (`:root` blokları; `indir` ayrıca logo)

**Interfaces:**
- Produces: `style.css` içindeki `:root` değişkenleri ve `.page` dışı sıfırlama kuralları. Sonraki görevler bu değişkenleri kullanır, yeniden tanımlamaz.
- Produces: `assets/logo.png` — Task 2 başlık çubuğunda ve hero'da bu dosyayı kullanır.

- [ ] **Step 1: Logoyu dosyaya çıkar**

```bash
cd /Users/ekremcansirin/sapancadayim-web
mkdir -p assets
python3 - <<'PY'
import re, base64
s = open('index.html').read()
m = re.search(r'data:image/png;base64,([^"]+)', s)
assert m, "logo bulunamadı"
open('assets/logo.png', 'wb').write(base64.b64decode(m.group(1)))
print("yazıldı:", len(m.group(1)) * 3 // 4, "byte")
PY
```

Beklenen çıktı: `yazıldı: 48057 byte`

- [ ] **Step 2: Logonun geçerli bir PNG olduğunu doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web && file assets/logo.png
```

Beklenen çıktı `PNG image data` içermeli. İçermiyorsa dur — base64 yanlış çıkarılmış.

- [ ] **Step 3: `style.css` dosyasını oluştur**

```css
/* sapancadayim.com — paylaşılan tasarım değişkenleri.
   Dört sayfa da bunu bağlar. Yalnızca değişkenler ve sıfırlama burada;
   sayfaya özgü düzen kuralları kendi dosyasında kalır.

   Tek temaya bağlı, bilerek: beyaz zemin marka kimliğinin parçası.
   Bu yüzden her renk açıkça boyanıyor, hiçbiri host temasından miras alınmıyor.
   prefers-color-scheme kuralı EKLENMEYECEK. */

:root {
  --ground: #FFFFFF;
  --ink: #0C2A22;
  /* #64796F idi. --wash zemini üstünde 4.41:1 veriyordu, AA eşiği 4.5:1.
     #5E7369 beyazda 5.08:1, --wash üstünde 4.80:1 — ikisi de geçiyor. */
  --muted: #5E7369;
  --brand: #059669;
  --line: #E4EBE6;
  --wash: #F4FAF7;
  --display: "Baloo 2", "Trebuchet MS", sans-serif;
  --body: "Source Sans 3", -apple-system, "Segoe UI", sans-serif;
}

* { box-sizing: border-box; }

body {
  margin: 0;
  background: var(--ground);
  color: var(--ink);
  font-family: var(--body);
  font-size: 17px;
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}

img { max-width: 100%; }

a {
  color: var(--brand);
  text-decoration-thickness: 1px;
  text-underline-offset: 3px;
}
a:hover { color: var(--ink); }
a:focus-visible {
  outline: 2px solid var(--brand);
  outline-offset: 3px;
  border-radius: 3px;
}
```

- [ ] **Step 4: Dört sayfayı `style.css`'e bağla**

Her dosyada, Google Fonts `<link>` satırının hemen ALTINA şunu ekle. Yol her
sayfada farklı — alt klasördekiler `../` ile çıkar:

`index.html` için:
```html
<link rel="stylesheet" href="/style.css">
```

`destek/index.html`, `gizlilik/index.html`, `indir/index.html` için aynı satır
kullanılır — kök yol olduğu için hepsinde `/style.css` doğrudur.

- [ ] **Step 5: Dört sayfadan tekrar eden kuralları sil**

Her dosyada şunları KALDIR (artık `style.css` sağlıyor):
- `:root { ... }` bloğunun tamamı ve üstündeki "Tek temaya bağlı, bilerek"
  yorumu (yorum `style.css`'e taşındı)
- `* { box-sizing: border-box; }`
- `body { margin: 0; }` satırı — ama `body`'nin diğer özellikleri
  (`background`, `color`, `font-family`, `font-size`, `line-height`,
  `-webkit-font-smoothing`) da `style.css`'te olduğu için **`body` bloğunun
  tamamı silinir**
- `img { max-width: 100%; }`
- `a`, `a:hover`, `a:focus-visible` kuralları

Geri kalan her şey (`.page`, `h1`, `.badge`, `footer` vb.) dosyada kalır.

- [ ] **Step 6: `index.html` ve `indir/index.html`'de logoyu dosyaya çevir**

Her iki dosyada `src="data:image/png;base64,..."` taşıyan `<img>` etiketini bul,
`src` değerini şununla değiştir:

```html
src="/assets/logo.png"
```

`class`, `alt` ve diğer öznitelikler aynen kalır.

- [ ] **Step 7: Sunucuyu başlat ve dört sayfanın da yüklendiğini doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web && python3 -m http.server 8787 &
sleep 1
for p in / /destek/ /gizlilik/ /indir/ /style.css /assets/logo.png; do
  printf "%s -> " "$p"
  curl -s -o /dev/null -w "%{http_code} %{size_download}\n" "http://localhost:8787$p"
done
```

Beklenen: altı satırın hepsi `200` ile başlar. `/style.css` boyutu 0'dan büyük,
`/assets/logo.png` yaklaşık `48057`.

- [ ] **Step 8: Hiçbir sayfada base64 kalmadığını doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web && grep -c 'data:image' index.html destek/index.html gizlilik/index.html indir/index.html
```

Beklenen: dördü de `0`.

- [ ] **Step 9: Dört sayfaya tarayıcıda bak**

`http://localhost:8787/`, `/destek/`, `/gizlilik/`, `/indir/` adreslerini aç.
Her birinde kontrol et: logo görünüyor mu, yazı tipi doğru mu (başlıklar Baloo 2'nin
yuvarlak hatlı hali), yeşil vurgu rengi yerinde mi, düzen bozulmamış mı.

Bir sayfa stilsiz (Times New Roman, mavi bağlantılar) görünüyorsa `style.css`
bağlantısı eksik ya da yolu yanlış.

- [ ] **Step 10: Commit**

```bash
cd /Users/ekremcansirin/sapancadayim-web
git add style.css assets/logo.png index.html destek/index.html gizlilik/index.html indir/index.html
git commit -m "refactor: palet ve logo ortak dosyalara taşındı

:root bloğu dört sayfada birebir tekrar ediyordu, style.css'e alındı.
Gömülü 48 KB base64 logo assets/logo.png oldu — iki sayfa aynı dosyayı
paylaşıyor ve tarayıcı önbelleğe alabiliyor.

--muted #64796F -> #5E7369: yeni --wash zemini üstünde 4.41:1 ile AA'yı
geçmiyordu, şimdi 4.80:1.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

### Task 2: Yer tutucu bileşeni, başlık çubuğu ve hero

Ana sayfanın üst yarısı. Bu görev bitince sayfa yeni düzene geçmiş olur ama
altında henüz özellik/QR/SSS bölümleri yoktur.

**Files:**
- Modify: `index.html` (`<style>` bloğu ve `<body>` içeriği)

**Interfaces:**
- Produces: `.slot` yer tutucu sınıfı — Task 3 ve Task 4 aynı sınıfı kullanır, yeniden tanımlamaz.
- Produces: `.wrap` (maksimum genişlik sarmalayıcı) ve `.section` (dikey boşluk) sınıfları — sonraki bölümler bunlara oturur.
- Consumes: Task 1'den `style.css` değişkenleri ve `/assets/logo.png`.

- [ ] **Step 1: `index.html`'deki eski düzen kurallarını sil**

`<style>` bloğunda Task 1'den geriye kalan `.page`, `img.mark`, `.eyebrow`,
`.dot`, `h1`, `p`, `.badges`, `.badge`, `.badge-text`, `footer`, `@media`
kurallarının HEPSİNİ sil. Blok boş kalsın — yerine Step 2 geliyor.

`<body>` içindeki `<div class="page">...</div>` da tamamen silinir.

- [ ] **Step 2: Yeni temel kuralları yaz**

`<style>` bloğunun içine:

```css
/* Sayfa iskeleti */
.wrap {
  max-width: 1040px;
  margin: 0 auto;
  padding-inline: 24px;
}
.section { padding-block: 72px; }
.section-wash { background: var(--wash); }

/* Başlık çubuğu */
.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding-block: 20px;
}
.topbar img { width: 148px; height: auto; }
.topbar nav {
  display: flex;
  gap: 20px;
  font-size: 15px;
  font-weight: 600;
}
.topbar nav a { color: var(--muted); text-decoration: none; }
.topbar nav a:hover { color: var(--brand); }

/* Yer tutucu — üç görsel alanı da bunu kullanıyor.
   Gerçek görsel gelince .slot satırı silinip img açılacak. */
.slot {
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  background: var(--wash);
  border: 2px dashed var(--line);
  border-radius: 14px;
  color: var(--muted);
  font-size: 14px;
  padding: 16px;
}

/* Hero */
.hero {
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 56px;
  align-items: center;
  padding-block: 56px 80px;
}
.hero-text { display: flex; flex-direction: column; align-items: flex-start; gap: 22px; }
.eyebrow {
  display: inline-flex; align-items: center; gap: 9px;
  font-size: 12px; font-weight: 600; letter-spacing: 0.15em; text-transform: uppercase;
  color: var(--brand);
}
.dot { width: 7px; height: 7px; border-radius: 50%; background: var(--brand); }
h1 {
  font-family: var(--display); font-weight: 800;
  font-size: clamp(30px, 5.2vw, 46px); line-height: 1.12; margin: 0;
  text-wrap: balance; letter-spacing: -0.01em; max-width: 16ch;
}
.lede { margin: 0; color: var(--muted); max-width: 44ch; font-size: 18px; }

/* Telefon çerçevesi */
.phone {
  aspect-ratio: 9 / 19.5;
  border: 10px solid var(--ink);
  border-radius: 38px;
  overflow: hidden;
  background: var(--wash);
}
.phone .slot { height: 100%; border: 0; border-radius: 0; }

/* Mağaza rozetleri */
.badges { display: flex; flex-wrap: wrap; gap: 10px; }
.badge {
  display: inline-flex; align-items: center; gap: 10px;
  border: 1px solid var(--line); border-radius: 11px;
  padding: 9px 15px 9px 12px; background: var(--ground); color: var(--ink);
}
.badge-text { display: flex; flex-direction: column; line-height: 1.12;
  font-family: var(--display); font-weight: 600; font-size: 15px; }
.badge-text small { font-family: var(--body); font-weight: 400; font-size: 10px;
  letter-spacing: 0.12em; text-transform: uppercase; color: var(--muted); }

@media (max-width: 800px) {
  .hero { grid-template-columns: 1fr; gap: 36px; padding-block: 36px 56px; }
  .phone { max-width: 260px; }
  .section { padding-block: 52px; }
}
```

- [ ] **Step 3: Başlık çubuğu ve hero işaretlemesini yaz**

`<body>` içine:

```html
<header class="wrap topbar">
  <img src="/assets/logo.png" alt="sapancadayım">
  <nav>
    <a href="/destek/">Destek</a>
    <a href="/gizlilik/">Gizlilik</a>
  </nav>
</header>

<main>
  <section class="wrap hero">
    <div class="hero-text">
      <span class="eyebrow"><span class="dot"></span>Yakında</span>

      <h1>Sapanca&rsquo;da ihtiyacın olan her şey tek uygulamada.</h1>

      <p class="lede">Kahvaltıdan bungalova, nöbetçi eczaneden tren saatine kadar &mdash; Sapanca&rsquo;yı bilen bir rehber cebinde.</p>

      <!-- YAYIN GÜNÜ (1/3): rozetlerdeki <small>Yakında</small> satırları silinecek,
           span yerine mağaza bağlantısı veren <a> gelecek. -->
      <div class="badges">
        <span class="badge" aria-label="Yakında App Store&rsquo;da">
          <svg viewBox="0 0 24 24" width="21" height="21" aria-hidden="true">
            <path fill="currentColor" d="M16.2 12.6c0-2.2 1.8-3.3 1.9-3.3-1-1.5-2.7-1.7-3.3-1.7-1.4-.1-2.7.8-3.4.8-.7 0-1.8-.8-2.9-.8-1.5 0-2.9.9-3.7 2.2-1.6 2.7-.4 6.8 1.1 9 .8 1.1 1.6 2.3 2.8 2.2 1.1 0 1.5-.7 2.9-.7 1.3 0 1.7.7 2.9.7 1.2 0 1.9-1.1 2.7-2.2.8-1.2 1.2-2.4 1.2-2.5 0 0-2.2-.9-2.2-3.7zM14 5.8c.6-.8 1-1.8.9-2.8-.9 0-2 .6-2.6 1.3-.6.7-1.1 1.7-.9 2.7 1 .1 2-.5 2.6-1.2z"/>
          </svg>
          <span class="badge-text"><small>Yakında</small>App Store</span>
        </span>
        <span class="badge" aria-label="Yakında Google Play&rsquo;de">
          <svg viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
            <path fill="#34A853" d="M3.9 2.1 14.7 12 3.9 21.9c-.5-.3-.9-.9-.9-1.6V3.7c0-.7.4-1.3.9-1.6z"/>
            <path fill="#FBBC04" d="M18.6 8.6 14.7 12l3.9 3.4 3-1.7c.9-.5.9-1.9 0-2.4l-3-1.7z"/>
            <path fill="#EA4335" d="M3.9 2.1c.4-.2.9-.2 1.4.1l13.3 6.4L14.7 12 3.9 2.1z"/>
            <path fill="#4285F4" d="M3.9 21.9 14.7 12l3.9 3.4L5.3 21.8c-.5.3-1 .3-1.4.1z"/>
          </svg>
          <span class="badge-text"><small>Yakında</small>Google Play</span>
        </span>
      </div>
    </div>

    <div class="phone">
      <!-- Ekran görüntüsü hazır olunca: assets/ekran.png ekle,
           .slot satırını sil, aşağıdaki img'i aç. -->
      <div class="slot" aria-hidden="true">Uygulama ekran görüntüsü buraya</div>
      <!-- <img src="/assets/ekran.png" alt="Sapancadayım uygulamasının ana ekranı"> -->
    </div>
  </section>
</main>
```

Not: `<main>` etiketi AÇIK kalıyor, Task 5'te kapatılacak. Task 3 ve Task 4 bölümlerini
onun içine ekleyecek.

- [ ] **Step 4: Yapıyı doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web
grep -c 'class="slot"' index.html          # beklenen: 1
grep -c 'YAYIN GÜNÜ' index.html            # beklenen: 1
grep -c 'assets/logo.png' index.html       # beklenen: 1
```

- [ ] **Step 5: Tarayıcıda iki genişlikte bak**

Sunucu çalışıyorsa `http://localhost:8787/` aç. Kontrol:
- **1280px:** solda metin, sağda telefon çerçevesi yan yana. Telefon çerçevesi
  uzun ve ince (9:19.5), içi kesikli kutu.
- **390px:** tek sütun, telefon metnin altına inmiş ve 260px'i geçmiyor.
- Logo başlık çubuğunda, sağında Destek/Gizlilik bağlantıları.

- [ ] **Step 6: Commit**

```bash
cd /Users/ekremcansirin/sapancadayim-web
git add index.html
git commit -m "feat: ana sayfa başlık çubuğu ve hero bölümü

Tek sütunlu 'yakında' düzeni, iki sütunlu hero'ya geçti. Telefon çerçevesi
şimdilik yer tutucu. Mağaza rozetleri 'Yakında' halinde kalıyor.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

### Task 3: Sapanca şeridi ve özellik kutuları

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: Task 2'den `.wrap`, `.section`, `.section-wash`, `.slot`.
- Produces: `.section-title` ve `.section-lede` — **Task 4 ve Task 5 bu iki sınıfı kullanır, yeniden tanımlamaz.**
- Produces: `.cards`, `.card`, `.card-icon` — yalnızca bu görevde kullanılıyor.

- [ ] **Step 1: Bölüm kurallarını `<style>` bloğunun sonuna ekle**

```css
/* Sapanca şeridi */
.strip { height: clamp(200px, 30vw, 320px); border-radius: 18px; }

/* Özellik kutuları */
.section-title {
  font-family: var(--display); font-weight: 800;
  font-size: clamp(24px, 3.4vw, 32px); margin: 0 0 8px;
  letter-spacing: -0.01em;
}
.section-lede { margin: 0 0 36px; color: var(--muted); max-width: 52ch; }
.cards {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 18px;
}
.card {
  background: var(--ground);
  border: 1px solid var(--line);
  border-radius: 16px;
  padding: 26px 24px;
}
.card-icon {
  width: 42px; height: 42px; border-radius: 12px;
  background: var(--wash);
  display: flex; align-items: center; justify-content: center;
  color: var(--brand);
  margin-bottom: 14px;
}
.card h3 {
  font-family: var(--display); font-weight: 600; font-size: 19px;
  margin: 0 0 6px;
}
.card p { margin: 0; color: var(--muted); font-size: 16px; }

@media (max-width: 800px) {
  .cards { grid-template-columns: 1fr; }
}
```

- [ ] **Step 2: Şerit ve kutu işaretlemesini ekle**

Task 2'de yazılan `</section>` etiketinden SONRA, hâlâ `<main>` içinde:

```html
  <div class="wrap">
    <!-- Sapanca fotoğrafı hazır olunca: assets/sapanca.jpg ekle,
         .slot satırını sil, aşağıdaki img'i aç. -->
    <div class="slot strip" aria-hidden="true">Sapanca fotoğrafı buraya &mdash; geniş, yatay</div>
    <!-- <img class="strip" src="/assets/sapanca.jpg" alt="Sapanca Gölü kıyısı"> -->
  </div>

  <section class="section section-wash">
    <div class="wrap">
      <h2 class="section-title">Uygulamada neler var?</h2>
      <p class="section-lede">Dört bölüm: gezmek için rehber, yaşamak için pratik bilgi, konakladığın yere sipariş ve konumuna göre en yakını.</p>

      <div class="cards">
        <article class="card">
          <div class="card-icon" aria-hidden="true">
            <svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="11" cy="11" r="7"/><path d="m20 20-3.6-3.6"/>
            </svg>
          </div>
          <h3>Keşfet &amp; Rehber</h3>
          <p>Kahvaltı, bungalov, villa, kafe, doğa ve aktivite. Telefonu ve yol tarifi tek dokunuş uzakta.</p>
        </article>

        <article class="card">
          <div class="card-icon" aria-hidden="true">
            <svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
              <path d="M12 3v18M3 12h18"/>
            </svg>
          </div>
          <h3>Sapanca Yaşam</h3>
          <p>Taksi, nöbetçi eczane, veteriner, yakıt ve şarj, usta ve çilingir, tren saatleri.</p>
        </article>

        <article class="card">
          <div class="card-icon" aria-hidden="true">
            <svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
              <path d="M4 7h11v9H4z"/><path d="M15 10h4l2 3v3h-6z"/>
              <circle cx="7.5" cy="18" r="1.6"/><circle cx="17" cy="18" r="1.6"/>
            </svg>
          </div>
          <h3>Bungalova Kurye</h3>
          <p>Konakladığın yere market, yemek ve ihtiyaç siparişi. Adres tarif etmekle uğraşma.</p>
        </article>

        <article class="card">
          <div class="card-icon" aria-hidden="true">
            <svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
              <path d="M12 21s7-5.6 7-10.4A7 7 0 0 0 5 10.6C5 15.4 12 21 12 21z"/>
              <circle cx="12" cy="10.5" r="2.4"/>
            </svg>
          </div>
          <h3>Yakınımda</h3>
          <p>Konumuna göre en yakın mekanlar, mahalle mahalle sıralı.</p>
        </article>
      </div>
    </div>
  </section>
```

- [ ] **Step 3: Doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web
grep -c '<article class="card">' index.html   # beklenen: 4
grep -c 'class="slot' index.html              # beklenen: 2
```

- [ ] **Step 4: Tarayıcıda bak**

- **1280px:** kutular 2x2 ızgara, açık yeşil zemin üstünde. Şerit tam genişlikte,
  yuvarlak köşeli.
- **390px:** kutular alt alta tek sütun.
- Kutu metinlerinin açık yeşil zemin üstünde rahat okunduğunu gözle teyit et.

- [ ] **Step 5: Commit**

```bash
cd /Users/ekremcansirin/sapancadayim-web
git add index.html
git commit -m "feat: Sapanca şeridi ve dört özellik kutusu

Kutular uygulamanın dört sekmesini birebir karşılıyor. Fotoğraf şeridi
yer tutucu.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

### Task 4: QR bölümü

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: Task 2'den `.wrap`, `.section`, `.slot`; Task 3'ten `.section-title`, `.section-lede`.
- Produces: `.qr-row`, `.qr-box`, `.qr-note` — Task 6'da yayın günü notu bu bölüme bakar.

- [ ] **Step 1: Kuralları `<style>` bloğunun sonuna ekle**

```css
/* QR bölümü */
.qr-row {
  display: grid;
  grid-template-columns: 200px 1fr;
  gap: 36px;
  align-items: center;
}
.qr-box { aspect-ratio: 1; border-radius: 16px; }
/* Uygulama mağazada olmadığı için kutu soluk duruyor.
   YAYIN GÜNÜ: aşağıdaki iki satır silinecek. */
.qr-box { opacity: 0.55; }
.qr-note { font-size: 14px; color: var(--muted); margin: 10px 0 0; }

@media (max-width: 800px) {
  .qr-row { grid-template-columns: 1fr; gap: 24px; justify-items: start; }
  .qr-box { width: 180px; }
}
```

- [ ] **Step 2: İşaretlemeyi ekle**

Task 3'ün `</section>` etiketinden sonra, hâlâ `<main>` içinde:

```html
  <section class="section">
    <div class="wrap qr-row">
      <div>
        <!-- YAYIN GÜNÜ (2/3): QR hazır olunca assets/qr.png ekle,
             .slot satırını sil, img'i aç, .qr-box opacity kuralını kaldır.
             QR şu adrese bakmalı: https://sapancadayim.com/indir/ -->
        <div class="slot qr-box" aria-hidden="true">QR kodu buraya</div>
        <!-- <img class="qr-box" src="/assets/qr.png" alt="Uygulamayı indirmek için QR kodu"> -->
      </div>
      <div>
        <h2 class="section-title">Telefonunla okut, buradan indir.</h2>
        <p class="section-lede" style="margin-bottom:0">Kodu kameranla okut; seni telefonuna uygun mağazaya götürsün.</p>
        <p class="qr-note">Uygulama henüz yayında değil. Kod, çıktığı gün burada çalışır durumda olacak.</p>
      </div>
    </div>
  </section>
```

- [ ] **Step 3: Doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web
grep -c 'sapancadayim.com/indir/' index.html   # beklenen: 1
grep -c 'YAYIN GÜNÜ' index.html                # beklenen: 2
grep -c 'class="slot' index.html               # beklenen: 3
```

- [ ] **Step 4: Tarayıcıda bak**

- **1280px:** solda kare QR kutusu (soluk), sağda başlık ve iki satır metin.
- **390px:** QR kutusu metnin üstünde, 180px genişlikte, sola hizalı.

- [ ] **Step 5: Commit**

```bash
cd /Users/ekremcansirin/sapancadayim-web
git add index.html
git commit -m "feat: QR bölümü (kod yer tutucu)

Kutu soluk duruyor çünkü uygulama henüz mağazada yok. QR hazır olunca
/indir/ adresine bakacak — mağaza bağlantıları değişse bile basılı
sticker geçerli kalır.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

### Task 5: SSS ve footer

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: Task 2'den `.wrap`, `.section`, `.section-wash`; Task 3'ten `.section-title`.
- Produces: sayfanın kapanışı — `</main>` bu görevde kapanır, footer geri gelir.

**Not:** Footer işaretlemesi Task 2'de silindi ve bu görevde geri geliyor. Yani
Task 2-4 arası sayfanın altında footer yok. Bu bilinçli — footer'ın yeni
`.wrap` düzenine oturması gerekiyordu.

- [ ] **Step 1: Kuralları `<style>` bloğunun sonuna ekle**

```css
/* SSS */
.faq { max-width: 680px; }
.faq details {
  border-bottom: 1px solid var(--line);
  padding: 18px 0;
}
.faq summary {
  cursor: pointer;
  font-family: var(--display); font-weight: 600; font-size: 18px;
  list-style: none;
  display: flex; align-items: center; justify-content: space-between; gap: 16px;
}
.faq summary::-webkit-details-marker { display: none; }
.faq summary::after {
  content: "+";
  font-weight: 600; color: var(--brand); font-size: 22px; line-height: 1;
}
.faq details[open] summary::after { content: "\2212"; }
.faq summary:focus-visible {
  outline: 2px solid var(--brand); outline-offset: 3px; border-radius: 3px;
}
.faq p { margin: 12px 0 0; color: var(--muted); }

/* Footer */
footer.site {
  border-top: 1px solid var(--line);
  padding-block: 28px 40px;
}
footer.site .wrap {
  display: flex; flex-wrap: wrap; gap: 6px 18px;
  font-size: 14.5px; color: var(--muted);
}
footer.site b { font-family: var(--display); font-weight: 600; color: var(--ink); }
```

- [ ] **Step 2: SSS, `</main>` kapanışı ve footer'ı ekle**

Task 4'ün `</section>` etiketinden sonra:

```html
  <section class="section section-wash">
    <div class="wrap faq">
      <h2 class="section-title">Sık sorulanlar</h2>

      <details>
        <summary>Uygulama ücretsiz mi?</summary>
        <p>Evet, ücretsiz.</p>
      </details>

      <details>
        <summary>Hangi telefonlarda çalışıyor?</summary>
        <p>iPhone ve Android telefonlarda.</p>
      </details>

      <details>
        <summary>İşletmemi nasıl eklerim?</summary>
        <p><a href="mailto:info@sapancadayim.com">info@sapancadayim.com</a> adresine yaz, işletmeni birlikte ekleyelim.</p>
      </details>

      <details>
        <summary>&ldquo;Sapancadayım Onaylı&rdquo; rozeti ne demek?</summary>
        <p>Bu işletmenin telefon, adres ve görsel bilgileri doğrudan işletme sahibinden alınmış, yayına girmeden önce teyit edilmiştir. Rozet, işletmenin gerçekten faaliyette olduğunu gösterir.</p>
        <p>Sunulan hizmetin kalitesi ya da yapacağınız rezervasyon ve ödeme ilişkisinin güvencesi anlamına gelmez. Kapora veya ön ödeme yapmadan önce işletmeyle doğrudan görüşmenizi öneririz.</p>
      </details>
    </div>
  </section>
</main>

<footer class="site">
  <div class="wrap">
    <b>sapancadayim.com</b>
    <span>Sapanca, Sakarya</span>
    <a href="mailto:info@sapancadayim.com">info@sapancadayim.com</a>
    <a href="/gizlilik/">Gizlilik Politikası</a>
    <a href="/destek/">Destek</a>
  </div>
</footer>
```

**Rozet metni uyarısı:** yukarıdaki iki paragraf uygulamadaki
`src/components/verified-info.tsx` metninin aynısıdır. Kelimeleri değiştirme —
mobil repodaki `docs/superpowers/specs/2026-09-20-onay-rozeti-instagram-butonu-design.md`
neden bu kadar sınırlı olduğunu anlatıyor: rozet belge/ruhsat denetimi ima etmemeli.

- [ ] **Step 3: Etiketlerin kapandığını doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web
grep -c '<details>' index.html    # beklenen: 4
grep -c '</main>' index.html      # beklenen: 1
grep -c '<main>' index.html       # beklenen: 1
python3 -c "
from html.parser import HTMLParser
class P(HTMLParser):
    def __init__(self): super().__init__(); self.stack=[]; self.bad=[]
    def handle_starttag(self,t,a):
        if t not in ('img','br','meta','link','hr','path','circle','input'): self.stack.append(t)
    def handle_endtag(self,t):
        if self.stack and self.stack[-1]==t: self.stack.pop()
        else: self.bad.append((t,list(self.stack[-3:])))
p=P(); p.feed(open('index.html').read())
print('kapanmayan:',p.stack); print('fazla kapanış:',p.bad)
"
```

Beklenen: `kapanmayan: []` ve `fazla kapanış: []`. Boş değilse etiket dengesi
bozuk, düzelt.

- [ ] **Step 4: Tarayıcıda bak ve klavyeyle dene**

- Dört SSS maddesi de tıklayınca açılıp kapanıyor, sağdaki `+` işareti açıkken
  `−` oluyor.
- `Tab` tuşuyla dolaş: başlık çubuğu bağlantıları, SSS başlıkları ve footer
  bağlantılarının hepsinde yeşil odak halkası görünmeli.
- `Enter` ile SSS maddesi açılmalı.

- [ ] **Step 5: Commit**

```bash
cd /Users/ekremcansirin/sapancadayim-web
git add index.html
git commit -m "feat: SSS bölümü ve footer

Dört soru, details/summary ile — JavaScript yok, klavye desteği hazır geliyor.
Rozet açıklaması uygulamadaki metnin birebir aynısı.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
```

---

### Task 6: Son denetim, meta bilgiler ve yayın

**Files:**
- Modify: `index.html` (`<head>` meta etiketleri)
- Modify: `README.md`

**Interfaces:**
- Consumes: Task 1-5'in tamamı.

- [ ] **Step 1: Meta açıklamalarını yeni sayfaya göre güncelle**

`<head>` içinde şu iki satırı bul ve değiştir:

```html
<meta name="description" content="Sapanca'nın dijital rehberi: kahvaltı, bungalov, nöbetçi eczane, tren saatleri, bungalova kurye ve yakınındaki mekanlar. Tek uygulamada.">
<meta property="og:description" content="Kahvaltı, bungalov, nöbetçi eczane, tren saatleri. Sapanca'nın dijital rehberi App Store ve Google Play'de yakında.">
```

- [ ] **Step 2: Yayın günü notlarının üçünün de yerinde olduğunu doğrula**

```bash
cd /Users/ekremcansirin/sapancadayim-web && grep -n 'YAYIN GÜNÜ' index.html
```

Beklenen: iki satır (`1/3` hero rozetleri, `2/3` QR). Üçüncüsü Step 3'te ekleniyor.

- [ ] **Step 3: Üçüncü yayın notunu hero'daki eyebrow'a ekle**

`<span class="eyebrow">` satırının hemen ÜSTÜNE:

```html
      <!-- YAYIN GÜNÜ (3/3): bu "Yakında" etiketi tamamen silinecek. -->
```

- [ ] **Step 4: Tüm sayfaları son kez kontrol et**

```bash
cd /Users/ekremcansirin/sapancadayim-web && python3 -m http.server 8787 &
sleep 1
for p in / /destek/ /gizlilik/ /indir/ /style.css /assets/logo.png; do
  printf "%s -> " "$p"
  curl -s -o /dev/null -w "%{http_code}\n" "http://localhost:8787$p"
done
grep -c 'YAYIN GÜNÜ' index.html     # beklenen: 3
grep -c 'prefers-color-scheme' index.html style.css   # beklenen: ikisi de 0
```

- [ ] **Step 5: Telefon genişliğinde yatay kaydırma olmadığını doğrula**

Tarayıcıyı 390px genişliğe getir, sayfayı en alta kadar kaydır. Sağa sola
kayma OLMAMALI. Oluyorsa sorumlu genelde sabit genişlikli bir öğedir —
`.phone`, `.qr-box` ve `.strip` ölçülerine bak.

- [ ] **Step 6: README'yi güncelle**

`README.md` dosyasına, varsa mevcut içeriğin altına ekle:

```markdown
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

`index.html` içinde `YAYIN GÜNÜ` diye aranır, üç yer çıkar:

1. Hero'daki "Yakında" etiketi — silinir
2. Mağaza rozetlerindeki `<small>Yakında</small>` — silinir, `<span class="badge">`
   yerine mağaza adresine giden `<a class="badge">` gelir
3. QR kutusundaki `opacity: .55` kuralı ve "henüz yayında değil" notu — silinir

Ayrıca mağaza rozetlerindeki Apple/Google logoları elle çizilmiş SVG; resmî
rozet dosyalarıyla değiştirilmeli.
```

- [ ] **Step 7: Commit ve yayına al**

```bash
cd /Users/ekremcansirin/sapancadayim-web
git add index.html README.md
git commit -m "docs: meta açıklamaları ve yayın günü notları

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>"
git push origin main
```

- [ ] **Step 8: Canlı siteyi doğrula**

GitHub Pages dağıtımı birkaç dakika sürer.

```bash
sleep 90
curl -s -o /dev/null -w "%{http_code}\n" https://sapancadayim.com/
curl -s https://sapancadayim.com/ | grep -c 'Uygulamada neler var'
curl -s -o /dev/null -w "%{http_code}\n" https://sapancadayim.com/style.css
curl -s -o /dev/null -w "%{http_code}\n" https://sapancadayim.com/assets/logo.png
```

Beklenen: `200`, `1`, `200`, `200`. Hâlâ eski sayfa geliyorsa 60 saniye bekleyip
tekrar dene; `gh api repos/ekremcansirin/sapancadayim-web/pages/builds/latest`
dağıtım durumunu gösterir.

**DİKKAT — DNS'e dokunma.** Bu iş DNS değişikliği gerektirmiyor. Squarespace
Domains'teki `MX @ → smtp.google.com`, SPF TXT ve DKIM TXT (`google._domainkey`)
kayıtları silinirse `info@sapancadayim.com` çalışmaz.
