# sapancadayim.com ana sayfa yeniden tasarımı

Tarih: 2026-09-22
Durum: onaylandı, plan yazılacak

## Amaç

Ana sayfa şu an tek ekranlık bir "yakında" duyurusu. İşletme sahibine
"uygulamaya ekleyeyim mi" denince ilk yapacağı şey adı aratmak; karşısına
uygulamanın ne olduğunu anlatan derli toplu bir sayfa çıkmalı. Ayrıca QR kodlu
sticker işi (mobil repo `docs/YAPILACAKLAR.md` §5) için sitede QR'a ayrılmış bir
yer gerekiyor.

Bu tasarım yalnızca `index.html`'i kapsar. `/destek/`, `/gizlilik/`, `/indir/`
sayfalarının içeriği değişmez — sadece ortak palete bağlanırlar.

## Kararlar

| Konu | Karar |
|---|---|
| Kapsam | Zengin tanıtım sayfası: hero, görsel şerit, özellikler, QR, SSS, footer |
| Görseller | Telefon ekranı, Sapanca fotoğrafı ve QR şimdilik **yer tutucu**; hazır oldukça doldurulacak |
| Yayın hali | Uygulama mağazada olmadığı için rozetler "Yakında", QR kutusu soluk |
| Yapı | Tek `index.html` + ortak `style.css` (yalnızca palet ve tipografi) |
| Tema | Tek tema, beyaz zemin. Koyu tema yok — marka kararı |

## Yapı

Mevcut `index.html` bir tanıtım sayfasına dönüşür. Sıra:

1. **Başlık çubuğu** — solda logo, sağda `Destek` ve `Gizlilik` bağlantıları.
2. **Hero** — solda başlık, tek paragraf açıklama ve iki mağaza rozeti; sağda
   telefon çerçevesi (içi yer tutucu).
3. **Sapanca şeridi** — tam genişlikte fotoğraf bandı (yer tutucu).
4. **Neler var** — 4 kutu, uygulamanın 4 sekmesini birebir karşılar:
   Keşfet & Rehber, Sapanca Yaşam, Bungalova Kurye, Yakınımda.
   Her kutu: ikon, başlık, iki satır açıklama.
5. **QR bölümü** — solda kare QR kutusu (yer tutucu), sağda
   "Telefonunla okut, buradan indir" başlığı ve tek satır açıklama.
6. **SSS** — 4 madde, `<details>`/`<summary>` ile aç-kapa.
7. **Footer** — mevcut footer aynen korunur (alan adı, konum, e-posta,
   gizlilik, destek).

Telefonda tek sütun; hero görseli başlığın altına, QR kutusu metnin üstüne geçer.

## Yer tutucular

Üç yer tutucu aynı bileşeni paylaşır: kesikli kenarlıklı, `#F4FAF7` zeminli,
ortasında ne geleceğini söyleyen küçük not. `aria-hidden="true"` taşır, ekran
okuyucu okumaz.

Görsel geldiğinde değişiklik tek satır olmalı. Kalıp:

```html
<!-- QR hazır olunca: assets/qr.png ekle, slot satırını sil, img'i aç -->
<div class="slot slot-qr" aria-hidden="true">QR kodu buraya</div>
<!-- <img src="assets/qr.png" alt="Uygulamayı indirmek için QR kodu"> -->
```

Aynı kalıp telefon ekranı (`assets/ekran.png`) ve Sapanca fotoğrafı
(`assets/sapanca.jpg`) için de geçerli. Dosyalar `assets/` klasöründe durur;
mevcut logo gibi base64 gömme yapılmaz — büyük görsellerde dosya şişer.

**QR hedefi:** `https://sapancadayim.com/indir/`. Mağaza bağlantıları sonradan
değişse bile basılı sticker geçerli kalır.

## Palet dosyası

Şu an `:root` bloğu 4 HTML dosyasında birebir tekrar ediyor; dosyalardaki yorum
da "hepsi birlikte değiştirilmeli" diyor. Palet ve tipografi değişkenleri
`style.css`'e taşınır, dört sayfa da `<head>` içinden bağlar. Sayfaya özgü
kurallar kendi dosyasında kalır — bu iş bir CSS birleştirme değil, yalnızca
tekrar eden değişkenleri tek yere almak.

Değişkenler korunur:
`--ground #FFFFFF`, `--ink #0C2A22`, `--brand #059669`, `--line #E4EBE6`,
`--display "Baloo 2"`, `--body "Source Sans 3"`.

Eklenen: `--wash #F4FAF7` — bölüm aralarındaki açık yeşil zemin.

**Tek değişen değer: `--muted` #64796F → #5E7369.** Sebep aşağıda, Erişilebilirlik
bölümünde. Fark gözle neredeyse seçilmiyor, dört sayfada birden geçerli olur.

## SSS içeriği

- **Uygulama ücretsiz mi?** Evet, ücretsiz.
- **Hangi telefonlarda çalışıyor?** iPhone ve Android.
- **İşletmemi nasıl eklerim?** info@sapancadayim.com adresine yaz.
- **"Sapanca Onaylı" rozeti ne demek?** Uygulama içindeki metnin aynısı
  kullanılır. Belge/ruhsat denetimi ima edilmez — mobil repo
  `docs/superpowers/specs/2026-09-20-onay-rozeti-instagram-butonu-design.md`
  gerekçeyi anlatıyor.

## Erişilebilirlik

- Her gerçek görselde `alt`; yer tutucular `aria-hidden="true"`.
- SSS `<details>`/`<summary>` — klavye ve ekran okuyucu desteği hazır gelir.
- Mevcut `:focus-visible` odak halkası kuralı korunur ve yeni bağlantılara da uygulanır.
- **Metin/zemin karşıtlığı.** Mevcut `--muted` (#64796F) beyaz üstünde 4.66:1
  ile AA'yı geçiyor, ama yeni `--wash` (#F4FAF7) zemini üstünde 4.41:1'e
  düşüyor — normal metin için eşik 4.5:1, yani kalıyor. Bu yüzden `--muted`
  #5E7369'a koyulaştırılır: beyazda 5.08:1, wash üstünde 4.80:1, ikisi de geçer.

## Yayına çıkışta yapılacak tek değişiklik

Rozetlerdeki "Yakında" satırı ve QR kutusunun soluk durumu kaldırılır, mağaza
bağlantıları rozetlere bağlanır. Bu üç nokta HTML'de yorumla işaretlenir ki
o gün aranmasın.

## Kapsam dışı

- Koyu tema
- E-posta ile haber verme formu (KVKK aydınlatma metni gerektirir)
- Çoklu dil
- `/destek/`, `/gizlilik/`, `/indir/` içeriklerinde değişiklik
- Mağaza rozetlerinin resmî dosyalarıyla değiştirilmesi (ayrı iş, yayın günü)
