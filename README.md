# RamaLabs FED Review v3 — Pemulihan 11 Standar, Pemetaan 73 Butir & Rekomendasi Otomatis (2026)

Aplikasi **review internal FED Sarjana Ilmu Komunikasi** menggunakan matriks 73 indikator dalam 11 standar dari berkas yang diunggah pengguna. **Copyright © 2026 ramalabs.** Tampilan menampilkan logo ASPIKOM di kanan atas dari situs organisasi; penggunaan tidak berarti afiliasi resmi.

## A. Deploy frontend ke GitHub Pages

1. **Jika memperbarui repository lama, hapus/replace file versi sebelumnya dan lakukan hard refresh (Ctrl+Shift+R).** Ekstrak ZIP. Unggah **isi direktori aplikasi** (termasuk `index.html`, `app.js`, `integration.js`, `styles.css`, `data/`, dan `.nojekyll`) ke root repository GitHub. Folder `worker/` merupakan sumber backend AI dan tidak menjalankan fungsi server di Pages.
2. `Settings → Pages → Deploy from a branch → main → /(root) → Save`.
3. Buka `https://USERNAME.github.io/NAMA-REPOSITORY/` setelah publikasi.
4. **Tanpa backend AI pun** dashboard, review 73 butir, pengunggahan DOCX/PDF, pengaitan kutipan, rekomendasi berbasis aturan, dan ekspor penilaian tetap tersedia.

## B. Membaca FED DOCX/PDF dan meninjau per item

1. Menu **Pembaca Dokumen & AI** → unggah dokumen `.docx`, `.pdf` berbasis teks, atau `.txt` (≤18 MB per file, delapan file per batch).
2. Parser DOCX: Mammoth browser dari jsDelivr. Parser PDF: PDF.js dari cdnjs. Karena CDN pihak ketiga, **memerlukan internet pada penggunaan pertama**; pustaka dijalankan pada browser pengguna. Tidak ada server unggah berkas pada frontend.
3. Klik **Petakan ke 73 indikator**. **Semua 73 butir kini ditampilkan**, termasuk butir tanpa kutipan yang cocok. Ringkasan membedakan kandidat leksikal dan butir yang belum memiliki kandidat. Kecocokan istilah bukan validasi bukti dan dapat menghasilkan false positive/negative; nama file dan halaman PDF atau nomor blok DOCX ditampilkan bila ditemukan.
4. Klik nomor butir → panel **Pemetaan dokumen & bukti** → tinjau kutipan → **Tautkan sebagai kandidat bukti**. Klik **Periksa kesesuaian bukti** untuk menemukan data yang masih kurang. Cantumkan bukti asli berikut periode, nomor/tautan arsip dan verifikasi substansi oleh reviewer.
5. Pada setiap butir tulis narasi asli, bukti, temuan, skor reviewer, status, dan PIC; klik **Buat draf perbaikan**. Setiap butir juga memiliki tombol **Buat draf perbaikan** dalam hasil pemetaan, termasuk bila belum ada bukti, serta tombol **Review butir**.
6. Filter standar menampilkan **11 standar + Semua standar**, dan daftar indikator mempertahankan semua 73 butir tanpa memerlukan berkas indikator eksternal; data dasar juga tertanam langsung di `index.html` untuk mencegah masalah asset tidak ditemukan pada GitHub Pages.
7. Tombol **Buat rekomendasi & narasi otomatis (AI opsional)** bekerja tanpa endpoint menggunakan mode aturan yang mengikuti rubrik, kebutuhan dokumen, dan referensi normatif. Jika proxy Worker diaktifkan, tombol yang sama memakai AI generatif. **Mode otomatis lokal bukan layanan AI dan tidak mengklaim dapat menilai kebenaran data.**

**Penting:** PDF yang hanya berupa gambar (hasil scan) tidak didukung OCR, dan DOCX tidak menyimpan informasi halaman (ditandai berdasarkan blok teks). Spreadsheet `.xlsx` belum diparsing dalam aplikasi ini. Rumus workbook sumber tetap perlu divalidasi secara terpisah.

## C. Aktifkan AI generatif melalui Cloudflare Worker

Frontend GitHub Pages **tidak boleh** menyimpan `OPENAI_API_KEY`. Implementasi contoh ada di `worker/index.js`, menggunakan API Chat Completions dan JSON terstruktur. Backend ini dipasang terpisah dari GitHub Pages.

1. Siapkan akun Cloudflare dan CLI Wrangler (`npm install -g wrangler`); login (`wrangler login`).
2. Buka direktori `worker/`; pada `wrangler.toml` ubah `ALLOWED_ORIGIN` menjadi **origin GitHub Pages tepat**, misalnya `https://namapengguna.github.io` (tanpa path repository). Atur `OPENAI_MODEL` bila diperlukan; model default `gpt-4.1-mini`.
3. Jalankan `wrangler secret put OPENAI_API_KEY` dan masukkan kunci API milik pemilik aplikasi. Lalu `wrangler secret put ACCESS_TOKEN` dan masukkan token acak yang kuat, berbeda dari kunci API.
4. Jalankan `wrangler deploy` dan catat URL HTTPS Worker yang ditampilkan.
5. Kembali ke halaman **Pembaca Dokumen & AI**, isi **Endpoint HTTPS Worker** dan **Token akses Worker**, lalu **Simpan konfigurasi**. Pilih indikator; tautkan kutipan bukti yang relevan; klik **Analisis & buat narasi substantif**. Respons dapat disisipkan ke editor dengan klik **Gunakan draf AI di editor**.

Hanya data indikator, isian reviewer, dan maksimal 8 kutipan tertaut yang dikirim saat pengguna menekan tombol AI. **Dokumen sumber lengkap tidak otomatis dikirim ke AI.** Token akses disimpan dalam `sessionStorage` pada browser (bukan repository), tetapi penggunaan perangkat bersama tetap berisiko. Selain CORS dan token, produksi publik sebaiknya menerapkan Cloudflare Access, rate limiting, quota/biaya, logging yang minim, dan kebijakan privasi institusi. Token akses yang dibagikan luas dapat disalahgunakan. Respons AI harus diverifikasi; AI tidak membuktikan otentisitas dokumen atau menetapkan status Unggul.

## D. Privasi dan backup

- Narasi, skor, temuan, dan status disimpan di **localStorage** browser pengunjung; **tidak otomatis tersinkron**. Tombol **Unduh JSON** menyimpan hasil review, **bukan berkas DOCX/PDF atau seluruh arsip kutipan**. Berkas dan kutipan tersimpan lokal secara terpisah, dan harus diimpor ulang jika perangkat berubah.
- Dokumen sensitif: gunakan komputer dan browser yang disetujui institusi. GitHub repository hanya berisi kode dan matriks; **jangan commit dokumen FED yang berisi data pribadi, token, atau API key**.
- Penyimpanan lokal terbatas; dokumen besar mungkin tidak dapat disimpan persisten. Bila muncul notifikasi gagal menyimpan, kurangi jumlah berkas dan ekspor hasil review.
- PDF hasil pindai memerlukan OCR terpisah. Pengaitan kutipan otomatis adalah pencarian heuristik, bukan ekstraksi semantik terjamin atau pemeriksaan hukum keabsahan bukti.

## E. Dasar dan penggunaan sumber

- Matriks PDF: `14_Matriks Penilaian Unggul_Sarjana.pdf` — 73 butir, 11 standar; bobot total yang terbaca 141, maksimum skor aritmetis 282. Penetapan peringkat resmi dan syarat perlu **tidak** dihitung otomatis.
- Template: `FED_Unggul_Sarjana.docx` — berisi instruksi untuk narasi per indikator, bukan FED sebuah program studi yang telah diisi.
- Dokumen contoh: `254_DOK Sarjana Ilmu Komunikasi Universitas Dian Nusantara-1.pdf` — surat informasi asesmen lapangan (25–27 Juni 2026) dan agenda, **bukan bukti kinerja 73 indikator**. Isi dokumen contoh tidak disisipkan menjadi bukti tanpa verifikasi.
- Logo ASPIKOM: https://aspikom.org/makna-logo-aspikom/ dan gambar dari https://aspikom.org/wp-content/uploads/2021/08/Logo-Aspikom-1030x1030.jpg . Hak merek tetap pada pemiliknya.

## F. Struktur file

```
index.html                 UI GitHub Pages
styles.css                 tampilan
app.js                     dashboard, skor, laporan, aturan rekomendasi
data/indicators.js         73 indikator + rubrik untuk browser
data/indicators.json       data terstruktur untuk developer
integration.js             parser DOCX/PDF, pemetaan kutipan, AI UI
worker/index.js            backend Cloudflare Worker (harus dideploy sendiri)
worker/wrangler.toml       konfigurasi Worker tanpa rahasia
.nojekyll                  pengaturan Pages
```

**Batas layanan:** tanpa koneksi endpoint Worker yang aktif, model AI generatif tidak bisa digunakan. Ini disengaja agar kunci API tidak bocor. Aplikasi bersifat alat bantu penjaminan mutu internal, **bukan aplikasi resmi LAMSPAK atau ASPIKOM**.

## G. Perubahan versi v3 dan verifikasi

- **Perbaikan pemuatan indikator:** 73 butir disematkan di HTML sekaligus disediakan di `data/indicators.json`; filter 11 standar selalu terisi meskipun asset `data/indicators.js` terlupa pada deploy sebelumnya.
- **Perbaikan pemetaan:** semua 73 indikator tampil dalam daftar akordeon, tidak menyembunyikan butir tanpa hit. Kandidat berbasis token dari judul/rubrik dan tautan ke cuplikan asal. Periksa manual agar tidak terjadi *false positive*.
- **Sumber eksternal legal:** aplikasi memuat referensi tautan publik resmi LAMSPAK dan regulasi nasional; tidak menyalin dokumen web atau menganggap data dari website pihak lain sebagai capaian institusi. Sumber normatif bukan bukti primer FED.
- **AI tanpa kunci:** mode rekomendasi/draf berbasis aturan bekerja langsung di browser. AI generatif eksternal tetap memerlukan backend proxy dan API key server-side, tidak mungkin menyimpan kunci aman di GitHub Pages.
- **Pemeriksaan UI:** diuji melalui Chromium dengan HTML/skrip disuntikkan ke halaman uji: 73 butir, 12 pilihan standar, pemetaan 73 baris, draf revisi dan fallback tanpa Worker, serta unggah TXT. Deploy langsung dan parser PDF/DOCX yang dimuat dari CDN perlu diuji pada URL publik.

**Catatan sumber:** rekomendasi dari internet disajikan sebagai sintesis kaidah normatif berdasarkan rubrik unggahan, instrumen LAMSPAK dan peraturan penjaminan mutu; tidak mengambil/mencampur capaian program studi lain menjadi fakta institusi pengguna. Penggunaan logo ASPIKOM di pojok kanan mengacu pada situs ASPIKOM dan tidak menyatakan afiliasi.
