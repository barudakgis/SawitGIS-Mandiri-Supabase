# SawitGIS Mandiri — Paket PWA untuk GitHub Pages

Paket ini berisi **front-end** aplikasi yang siap di-host sebagai situs statis di
GitHub Pages, sehingga aplikasi **tetap terbuka walau HP tanpa sinyal**.
Google Apps Script tetap dipakai, tetapi hanya sebagai **API data**.

> **Kenapa perlu ini?** Versi yang di-host langsung di Apps Script (`…/exec`)
> mengunduh HTML-nya dari server Google setiap kali dibuka. Tanpa sinyal,
> halaman jadi blank. Service Worker (mesin offline PWA) juga tidak bisa
> didaftarkan di dalam iframe Apps Script. Menghosting front-end secara statis
> adalah satu-satunya cara agar benar-benar bisa offline.

---

## Isi paket

| Berkas | Fungsi |
|---|---|
| `index.html` | Seluruh aplikasi (HTML + CSS + JS jadi satu berkas) |
| `sw.js` | Service Worker — menyimpan cangkang aplikasi agar bisa dibuka offline |
| `manifest.json` | Identitas aplikasi (nama, ikon, warna) untuk "Install app" |
| `icon-192.png` | Ikon aplikasi 192×192 |
| `icon-512.png` | Ikon aplikasi 512×512 |
| `.nojekyll` | Menonaktifkan Jekyll agar GitHub menyajikan berkas apa adanya |
| `README.md` | Panduan ini |

---

## Langkah 1 — Pastikan Apps Script sudah jadi API

1. `Code.gs` versi terbaru sudah ditempel (yang memiliki fungsi `doPost`).
2. Jalankan **`setupDatabase()`** satu kali (izinkan otorisasi, termasuk Drive).
3. **Deploy → New deployment → Web app**
   - *Execute as*: **Me**
   - *Who has access*: **Anyone**
4. Salin **URL `/exec`**, contoh:
   `https://script.google.com/macros/s/AKfycbxxxxxx/exec`

## Langkah 2 — Cocokkan URL API di `index.html`

Buka `index.html`, cari baris berikut (ada di dekat awal blok `<script>`):

```js
var API_URL = 'https://script.google.com/macros/s/..../exec';
```

Pastikan isinya **sama persis** dengan URL `/exec` hasil deploy terbaru Anda.
Kalau Anda men-deploy ulang dengan *New deployment* (bukan *New version*),
URL-nya berubah dan baris ini **wajib** diperbarui.

> Berkas yang sama tetap bisa dipakai di Apps Script. Aplikasi mendeteksi
> sendiri: di dalam Apps Script memakai `google.script.run`, di hosting statis
> memakai `fetch` ke `API_URL`.

## Langkah 3 — Unggah ke GitHub

### Cara A — lewat web (paling mudah, tanpa Git)

1. Buka <https://github.com/new> → buat repo baru, misal `barudak-gis`.
   Pilih **Public** (GitHub Pages gratis hanya untuk repo publik).
2. Di halaman repo → **Add file → Upload files**.
3. Seret **seluruh isi folder ini** (7 berkas, termasuk `.nojekyll`).
   > Kalau `.nojekyll` tidak ikut terseret karena diawali titik, buat manual:
   > **Add file → Create new file** → beri nama `.nojekyll` → biarkan kosong → Commit.
4. Klik **Commit changes**.

### Cara B — lewat Git

```bash
git init
git add .
git commit -m "SawitGIS PWA"
git branch -M main
git remote add origin https://github.com/<username>/<repo>.git
git push -u origin main
```

## Langkah 4 — Aktifkan GitHub Pages

1. Di repo → **Settings** → menu kiri **Pages**.
2. **Source**: `Deploy from a branch`
3. **Branch**: `main` — folder: `/ (root)` → **Save**.
4. Tunggu 1–2 menit. URL akan muncul di halaman itu:
   `https://<username>.github.io/<repo>/`

## Langkah 5 — Pasang di HP

1. Buka URL tersebut di **Chrome Android** (atau Safari iOS) **saat ada sinyal**.
2. Login seperti biasa — sekali ini butuh internet untuk verifikasi & unduh data.
3. Menu Chrome (⋮) → **Install app** / **Tambahkan ke Layar Utama**.
4. Buka aplikasi dari ikon di layar utama.

Setelah itu, saat **tanpa sinyal**: aplikasi tetap terbuka, data terakhir muncul
dari cache, input tersimpan di antrean, dan tersinkron otomatis begitu sinyal kembali.

---

## Memperbarui aplikasi di kemudian hari

Setiap kali `index.html` diubah, lakukan **dua-duanya**:

1. Unggah ulang `index.html` ke repo.
2. **Naikkan angka versi** di `sw.js`:
   ```js
   var VERSI = 'sawitgis-v10';   // ubah ke v11, v12, dst.
   ```
   Tanpa ini, HP pengguna masih memakai versi lama dari cache.

Pengguna cukup membuka aplikasi 2× (sekali untuk mengunduh versi baru di latar
belakang, sekali untuk memakainya).

---

## Mengganti nama & ikon aplikasi

- **Nama di layar utama HP**: ubah `name` dan `short_name` di `manifest.json`.
- **Ikon**: ganti `icon-192.png` dan `icon-512.png` (ukuran harus tetap sama).
- **Judul di dalam aplikasi**: tidak perlu diubah di sini — atur lewat menu
  **Pengaturan** di aplikasi, karena tersimpan di Google Sheets.

---

## Catatan penting

- **HTTPS wajib.** GitHub Pages sudah HTTPS, jadi aman. Service Worker dan GPS
  memang hanya berjalan di HTTPS.
- **Repo publik = berkas bisa dilihat siapa saja.** `index.html` memuat URL API
  Anda. Data tetap terlindungi login, tetapi jangan menaruh kata sandi,
  kunci rahasia, atau data karyawan di dalam berkas ini.
- **Peta offline terbatas.** Hanya area peta yang **pernah dibuka saat online**
  yang tersimpan; area yang belum pernah dilihat tidak tersedia offline.
- **Foto & video** butuh koneksi untuk diunggah ke Drive. Saat offline, catatan
  teks tersimpan lebih dulu; media menyusul lewat Edit setelah online.
- **Cuaca dan panggilan API** sengaja tidak di-cache agar datanya selalu segar.
- Versi Apps Script (`/exec`) tetap berfungsi normal untuk pemakaian di kantor
  yang selalu online. Mode PWA ini khusus untuk lapangan bersinyal buruk.

---

## Kalau ada masalah

| Gejala | Penyebab & solusi |
|---|---|
| Halaman blank / 404 | GitHub Pages belum selesai membangun (tunggu 2 menit), atau folder Pages salah — pastikan `/ (root)`. |
| Bisa dibuka tapi data tidak muncul | `API_URL` di `index.html` tidak cocok dengan URL `/exec` terbaru, atau deployment belum *Anyone*. |
| Perubahan tidak muncul di HP | Angka `VERSI` di `sw.js` belum dinaikkan. Naikkan, unggah ulang, buka aplikasi 2×. |
| Tombol Install tidak muncul | Harus HTTPS, `manifest.json` terbaca, dan Service Worker terdaftar. Cek Chrome → ⋮ → Alat pengembang → Application. |
| GPS tidak jalan | Pastikan URL diakses langsung (bukan di dalam iframe) dan izin lokasi diizinkan. |

---

## Memperbarui ke v5.0.0

Unggah ulang **`index.html`** dan **`sw.js`** dari folder ini. Versi cache naik ke
`sawitgis-v5.0.0`, sehingga HP pengguna mengambil versi baru dan menampilkan
pemberitahuan pembaruan.

Fitur **Peta Offline** memerlukan HTTPS — GitHub Pages sudah memenuhinya.
Salinan foto udara tersimpan di penyimpanan berkas peramban (OPFS), terpisah
dari cache service worker, sehingga tidak ikut terhapus saat versi naik.
