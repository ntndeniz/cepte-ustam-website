# Cepte Ustam — Tanıtım Web Sitesi (PUBLIC)

Bu klasör, **herkese açık** tanıtım sitesidir. Uygulamanın (Flutter) kaynak
koduyla **hiçbir bağı yoktur** — tek başına çalışan statik bir sayfadır
(`index.html` içinde HTML + CSS + JS). Firebase/uygulama bağlantısı içermez.

## Yapı
- `index.html` — tanıtım sayfası + tıklanabilir telefon demosu (tek dosya).

## Yayınlama (Netlify)
1. Bu klasörü **kendi (public) GitHub reposuna** yükle.
2. Netlify → "Add new site" → GitHub reposunu seç.
3. Build ayarı gerekmez (statik site): Build command boş, Publish directory `.`
4. Deploy → `https://<siteadi>.netlify.app` hazır.

## ÖNEMLİ — Ayrım
- **Bu repo:** PUBLIC (tanıtım sitesi). Kimliğe/gizli bilgiye yer yok.
- **Uygulama (`CepteUsta` klasörü):** ayrı bir **PRIVATE** GitHub reposu olacak;
  Firebase anahtarları ve uygulama kaynağı orada, kimseye görünmez.

İki repoyu **karıştırma** — site public, uygulama private.
