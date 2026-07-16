# MCN-Performance-Report

Dashboard statis (HTML + Tailwind CSS + Chart.js) untuk memantau performa video & finansial MCN. Data dimuat dari Google Sheet yang di-publish sebagai CSV; jika gagal dimuat, dashboard otomatis memakai data cadangan lokal di dalam `index.html`.

## Setup Google Sheet

### 1. Buat Google Sheet
    
Cara tercepat: pakai [`Monthly_Data_template.csv`](Monthly_Data_template.csv) di folder ini sebagai starting point — file CSV ini bisa langsung dibuka di Excel, atau di-import ke Google Sheets lewat **File → Import → Upload**, lalu pilih **Replace current sheet** dan beri nama sheet-nya `Monthly_Data`. File ini sudah berisi 18 baris data yang sama persis dengan yang saat ini ter-hardcode di dashboard (2025 Jan-Des + 2026 Jan-Jun), jadi begitu diimpor & di-publish, dashboard akan tampil identik seperti sekarang — tinggal lanjutkan menambah baris bulan berikutnya di sheet ini ke depannya.

Atau buat manual: buat satu worksheet bernama `Monthly_Data` dengan header persis seperti berikut di baris pertama (urutan kolom bebas, ejaan harus sama persis):

```
year, qtr, month, views, uploads, revenue_3fta, revenue_img, revenue_starhits, target_3fta, target_img, target_starhits, cms_mnctv, cms_gtv, cms_inews, cms_scm, cms_sma, cms_rcti, cms_sh, cms_shmcn
```

Satu baris = satu bulan. Contoh:

```
year,qtr,month,views,uploads,revenue_3fta,revenue_img,revenue_starhits,target_3fta,target_img,target_starhits,cms_mnctv,cms_gtv,cms_inews,cms_scm,cms_sma,cms_rcti,cms_sh,cms_shmcn
2026,Q1,Jan,2161614761,41564,3200123110,145121000,803660238,7189700000,0,692884000,128221110,125112110,620456123,90123112,910412332,1190412331,265112332,819050112
```

Aturan format:
- `month` harus salah satu dari: `Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec`.
- `qtr` harus salah satu dari: `Q1 Q2 Q3 Q4`.
- 16 kolom sisanya berupa angka. **Format sel sebagai "Number" biasa (bukan Currency/Accounting)** agar tidak diekspor dengan simbol mata uang/pemisah ribuan yang bisa gagal terbaca.

### 2. Publish to Web sebagai CSV

1. Di Google Sheets: **File → Share → Publish to web**.
2. Pilih sheet `Monthly_Data`, format **Comma-separated values (.csv)**.
3. Klik **Publish**, salin URL yang muncul (bentuknya mirip `https://docs.google.com/spreadsheets/d/e/xxxxx/pub?output=csv`).

> Gunakan URL dari **Publish to web**, bukan URL "Share" biasa — hanya URL Publish to web yang bisa diakses langsung oleh dashboard tanpa login.

### 3. Pasang URL ke dashboard

1. Salin `config.example.js` menjadi `config.js` (ada di folder yang sama dengan `index.html`).
2. Buka `config.js`, ganti nilai `GOOGLE_SHEET_CSV_URL` dengan URL dari langkah 2.

```js
const GOOGLE_SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/e/xxxxx/pub?output=csv";
```

`config.js` sengaja tidak ikut ter-commit ke git (lihat `.gitignore`) supaya URL Sheet Anda tidak tersimpan di riwayat repo.

### 4. Update data

Setiap kali Anda mengubah/menambah baris di Google Sheet, tidak perlu langkah tambahan — dashboard fetch ulang data setiap kali halaman dibuka/direfresh (tidak ada cache).

## Troubleshooting

Cek indikator status kecil di bawah judul dashboard ("Memuat data...", "Data berhasil dimuat", atau "Gagal memuat Google Sheet"), dan buka Console browser (F12) untuk detail error.

| Gejala | Kemungkinan penyebab |
| --- | --- |
| Status "Gagal memuat Google Sheet" terus muncul, dashboard menampilkan data cadangan lama | `config.js` belum dibuat, atau `GOOGLE_SHEET_CSV_URL` masih placeholder / salah / bukan URL Publish to web |
| Error `HTTP 401`/`403`/halaman login di Console | Sheet belum di-Publish to web, atau publikasi sudah dicabut |
| Error "Header CSV tidak lengkap, kolom hilang: ..." | Ada nama kolom header yang salah ketik / terhapus di sheet `Monthly_Data` |
| Data tampak 0 di beberapa baris + ada `console.warn` "kolom ... kosong" | Ada sel angka kosong di sheet pada baris tsb — sel tersebut dianggap 0, baris lain tetap normal |
| Ada baris yang tidak muncul di dashboard + `console.warn` "baris dilewati" | Nilai `year`/`qtr`/`month` di baris tsb tidak sesuai format yang diwajibkan |
