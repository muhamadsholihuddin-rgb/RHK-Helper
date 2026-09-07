# AutoReport Pro — PKH (PWA → APK)

Aplikasi laporan RHK ASN PPPK PKH ini sudah disiapkan sebagai **PWA (Progressive Web App)**
lengkap dengan manifest, ikon, service worker (mode offline), dan tombol **Bagikan** (Web Share API)
sehingga siap di-build menjadi file **.apk** lewat PWABuilder.

Isi folder ini **wajib diupload bersama-sama**, jangan hanya `index.html`:

```
├── index.html          ← aplikasi utama
├── manifest.json        ← identitas PWA (nama, ikon, warna)
├── sw.js                 ← service worker (offline & caching)
└── icons/
    ├── icon-192.png
    ├── icon-512.png
    ├── icon-512-maskable.png
    └── apple-touch-icon.png
```

---

## 0. Deploy ke Vercel (via GitHub) — cara yang dipakai saat ini

1. Push semua file di folder ini (termasuk `vercel.json`) ke repo GitHub kamu.
2. Di [vercel.com](https://vercel.com) → **Add New → Project** → import repo tsb.
3. Framework preset pilih **Other** (bukan Next.js/dst), root directory biarkan default (folder yang berisi `index.html`).
4. Deploy. `vercel.json` sudah mengatur:
   - Semua path (`/jurnal`, `/form`, `/preview`, `/daftar-rhk`) diarahkan ke `index.html` (SPA rewrite) — supaya alamat itu tidak 404 dan bisa dibuka langsung.
   - `manifest.json` disajikan dengan `Content-Type: application/manifest+json` dan `sw.js` dengan `Cache-Control: no-cache` (supaya service worker baru selalu terdeteksi, tidak nyangkut versi lama).
5. Buka URL Vercel-nya (mis. `https://nama-project.vercel.app`) — otomatis diarahkan ke `/jurnal`. Inilah URL yang dipakai di PWABuilder.

> Catatan tombol **Instal Aplikasi**: tombol ini sekarang selalu langsung tampil begitu halaman dibuka. Di Chrome/Edge Android/Desktop, saat diklik ia akan memicu dialog instal native jika browser sudah menganggap situs "layak instal" (butuh sedikit interaksi/waktu di beberapa kasus — batasan dari Chrome sendiri, bukan dari aplikasi ini); jika belum, tombol menampilkan petunjuk instal manual (menu ⋮ → "Instal aplikasi"/"Tambahkan ke Layar Utama") sebagai fallback, sehingga pengguna tidak pernah melihat tombol yang tidak berfungsi.

---

## 1. Upload ke GitHub

1. Buat repository baru di GitHub (public), misal `autoreport-pkh`.
2. Upload **semua file & folder di atas** ke root repository itu (lewat web "Add file → Upload files",
   atau via git):
   ```bash
   git init
   git add .
   git commit -m "AutoReport Pro PKH - PWA"
   git branch -M main
   git remote add origin https://github.com/USERNAME/autoreport-pkh.git
   git push -u origin main
   ```
3. Buka **Settings → Pages** di repo tersebut.
   - Source: `Deploy from a branch`
   - Branch: `main` / folder `/ (root)`
   - Klik **Save**.
4. Tunggu 1–2 menit, GitHub akan memberi URL seperti:
   ```
   https://USERNAME.github.io/autoreport-pkh/
   ```
   Buka URL itu — pastikan aplikasi tampil normal dan tombol **Bagikan**/**Cetak PDF** berfungsi.
   Ini adalah URL yang akan dipakai di PWABuilder.

> Situs **harus HTTPS** agar Service Worker & Web Share API aktif — GitHub Pages otomatis HTTPS, aman.

---

## 2. Build APK dengan PWABuilder

1. Buka **https://www.pwabuilder.com**
2. Tempel URL GitHub Pages kamu (`https://USERNAME.github.io/autoreport-pkh/`) → klik **Start**.
3. PWABuilder akan mengambil `manifest.json` dan `sw.js` secara otomatis lalu menilai skor PWA.
   Jika ada peringatan minor (misal "screenshots"), boleh diabaikan — tidak wajib untuk build APK.
4. Klik tab **Android** → **Generate Package**.
   - Package ID: contoh `com.namamu.autoreportpkh` (boleh diubah bebas).
   - App name: `AutoReport Pro`.
   - Signing key: pilih **"Create new signing key"** bila ini APK pertama kamu — **simpan file `.keystore`
     dan passwordnya baik-baik**, dibutuhkan lagi kalau nanti mau update APK ke versi baru.
5. Klik **Generate** → PWABuilder akan menyiapkan file `.zip` berisi `app-release-signed.apk`
   (atau `.aab` untuk Play Store) → unduh.
6. Ekstrak zip, ambil file `.apk`, kirim/instal ke HP Android (aktifkan "Izinkan sumber tidak dikenal"
   saat instalasi pertama kali).

---

## 3. Bank Data Narasi (default otomatis dari histori laporan)

Isi narasi (Maksud & Tujuan, Narasi Kegiatan, Hasil yang Dicapai) sekarang otomatis terisi dari **Bank Data Laporan RHK** (diambil dari histori 26 laporan yang pernah dibuat), dicocokkan ke kegiatan yang sedang dipilih. Bank narasi lama (tulisan tangan, banyak variasi acak) tetap tersedia sebagai variasi tambahan — muncul saat tombol **Perbarui/Acak Narasi** ditekan. Field "Pendahuluan Umum" tetap memakai bank narasi lama karena data histori tidak memiliki bagian ini. Semua isian tetap bisa diedit manual kapan saja.

---

## 4. Tentang fitur di dalam aplikasi

- **Tombol Bagikan** memakai Web Share API: aplikasi otomatis membuat file PDF asli dari halaman
  Pratinjau, lalu membuka menu share bawaan Android (WhatsApp, Email, Drive, dll). Jika perangkat/versi
  Android tidak mendukung share file, aplikasi otomatis mengunduh PDF-nya sebagai gantinya.
- **Cetak PDF** tetap memakai dialog cetak browser (cocok untuk print langsung / simpan manual).
- **Export Word** mengunduh file `.doc` yang bisa dibuka di Microsoft Word.
- Data **profil & foto** (nama, NIP, tanda tangan, kop surat) tersimpan otomatis di penyimpanan lokal
  perangkat (localStorage) — aman dan tidak terkirim ke server manapun, karena aplikasi ini 100%
  berjalan di sisi klien (tidak ada backend).
- Tampilan sudah disesuaikan agar teks, label, dan tombol lebih besar & mudah dibaca di layar HP,
  dengan navigasi bawah (Edit / Lihat / RHK / Bagikan / PDF) khusus untuk mode ponsel.

---

## 5. Update aplikasi di kemudian hari

Setiap kali kamu mengubah `index.html` (atau file lain), cukup `git push` lagi ke branch `main` —
GitHub Pages akan otomatis memperbarui. Untuk APK yang sudah terpasang di HP, PWA berbasis
**Trusted Web Activity** (yang dihasilkan PWABuilder) akan memuat versi terbaru dari GitHub Pages
secara otomatis saat dibuka (karena isinya sebenarnya membuka website kamu di dalam wrapper APK),
jadi kamu **tidak perlu build ulang APK** untuk perubahan konten biasa. Build ulang hanya diperlukan
jika kamu mengubah `manifest.json` (nama app, ikon) atau ingin naik versi di Play Store.
