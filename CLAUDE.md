# Web Promo MIG Swalayan

Website belanja untuk pembeli: produk promo MIG Swalayan tampil di rak kaca berlampu. Ketuk produk → lampu di bawahnya menyala, produk lain meredup, lalu zoom ke halaman detail (harga coret, harga promo, hemat, jumlah, Tambah ke Keranjang). Checkout membuka WhatsApp toko dengan daftar pesanan terisi otomatis.

- Pemilik: product designer, bukan programmer. Jelaskan perubahan dengan bahasa sederhana (Bahasa Indonesia). Claude yang mengerjakan kodenya.
- Hosting: Cloudflare Pages, deploy otomatis dari branch `main`, tanpa build.
- Mockup acuan (Claude Design): https://claude.ai/artifact/7HRvP6qMPtzeTF5bpxthWi

## File
- `index.html` — seluruh halaman (HTML + CSS + JS vanilla, satu file, tanpa library).
- `logo.png`, `favicon-32.png`, `apple-touch-icon.png`, `icon-512.png` — disalin dari repo generator `iif069/promo-katalog-MIG-Swalayan`.

## Data
Tidak punya backend sendiri. Memakai Web App Apps Script yang SAMA dengan generator poster promo (`API_URL`, Sheet "Promo MIG Swalayan 2026"), dibaca lewat JSONP:
1. `?action=posters` → poster tersimpan terbaru (urut terbaru dulu) → `?action=poster&nama=…` → judul, periode, items {nama, coret, promo, beli2, label, foto}. Jadi produk di web = isi poster promo terakhir yang disimpan staf di generator.
2. Cadangan kalau belum ada poster: `?action=list`, ambil produk yang `Checkbox`-nya dicentang.
Perubahan Code.gs dikerjakan di repo generator, bukan di sini.

## Cara kerja penting
- `WA_NUMBER` di atas script: nomor WA toko format `62xxxxxxxxxx`. Kosong = tombol checkout abu-abu + teks "Nomor WhatsApp toko belum diatur". (Nomor belum diberikan pemilik.)
- Foto lh3 (`=w800`) dimuat dengan `crossOrigin=anonymous`, lalu `cutWhite` membuang latar putih yang menyambung ke tepi, membuang bintik lepas (<2% bagian terbesar), lalu memotong ruang kosong (pernah dicoba mengikis tepi terang keabuan 3 putaran — DIBATALKAN karena bagian putih produk seperti segel plastik ikut hilang), supaya produk "berdiri" di rak gelap. Kalau foto tidak bisa dibaca canvas, foto asli tampil di alas putih membulat (class `raw`). Foto diproses satu per satu.
- Nama produk: kalau di Sheet seluruhnya HURUF BESAR diubah jadi Huruf Awal Besar (satuan kg/gr/ml kecil, kata tanpa vokal mis. SKM tetap kapital); kalau sudah huruf besar-kecil dibiarkan.
- Badge: diskon % (kuning) kalau harga coret > promo; label promo merah (`label`, default "Beli 2 lebih hemat") kalau `beli2`.
- Keranjang disimpan di localStorage `migPromoCart` bersama id poster; kalau poster promo berganti, keranjang lama dikosongkan dan produk yang tidak ada lagi dibuang.
- Detail & keranjang memakai `history.pushState`, jadi tombol kembali HP menutupnya. Esc juga.
- Zoom: foto di detail dianimasikan dari posisi fotonya di rak (FLIP, `element.animate`).
- `?contoh` di URL = mode contoh dengan 8 produk dan foto buatan, untuk mengecek tampilan tanpa Sheet (ada pita "Mode contoh").
- Ujung kiri/kanan tiap bilah rak dibulatkan (`markEdges`, dihitung ulang saat ukuran layar berubah).
- Performa (HP kelas bawah, Samsung Internet): animasi hanya memakai transform & opacity. JANGAN pakai `filter` (drop-shadow/brightness) pada foto atau lampu, dan jangan animasikan `box-shadow` — cahaya LED ada di `::after` yang hanya berubah opacity (`breathe` = opacity). Header `position:fixed` z-index 30 + `translateZ(0)`, solid tanpa `backdrop-filter`; `main` dan `.rak` diberi `isolation:isolate` (di Samsung Internet garis rak sempat tergambar di atas header sticky). Jangan pakai `will-change` di tiap slot. Foto diproses saat browser senggang (`requestIdleCallback`), ditunda selama animasi detail (`state.busy`), hasilnya blob WebP (object URL), bukan dataURL. Zoom menunggu `img.decode()`.

## Aturan desain
- Tema gelap: dinding #14110C bertekstur halus, rak kaca akrilik (atas berperspektif, sisi depan tebal berkilau, bayangan), LED di bawah rak.
- Keranjang dan panel detail: putih (#fff), baris produk putih bergaris abu tipis, tombol jumlah abu muda.
- Kuning brand MIG Swalayan #F7EC1F (label, badge diskon, harga di rak, lampu). Hijau brand #0D8A2C untuk harga promo di detail, teks hemat, dan tombol WhatsApp. Merah #E3201B untuk pill periode dan label promo.
- Font: Bricolage Grotesque (judul & harga) + Plus Jakarta Sans (teks).
- Rak: 2 kolom di HP, 3 (≥640px), 4 (≥960px), 5 (≥1140px). Detail di PC: panggung kiri, panel putih kanan.
- Tombol minimal 44px, ikon SVG (bukan emoji).
