# Checklist Keamanan — paulgg — Studio

Peta keamanan website portfolio statis, mengikuti 5 tahap standar
(perencanaan → pengodean → pengujian → deployment → pemeliharaan).
Setiap item ditandai: **[SELESAI]** sudah diterapkan, **[TUGAS KAMU]** perlu
tindakan manual saat deploy, **[N/A]** tidak relevan untuk situs statis.

---

## 1. Perencanaan & Threat Modeling

- **[SELESAI] Petakan data sensitif.** Situs ini TIDAK menyimpan data apa pun:
  tanpa database, tanpa form input, tanpa login, tanpa cookie. Tidak ada
  password atau data pribadi pengunjung yang bisa dicuri. Satu-satunya data
  pribadi di halaman adalah milik pemilik situs (nama, Instagram, email).
- **[SELESAI] Threat modeling.** Permukaan serangan yang tersisa:
  1. *Content injection* — kalau hosting/file manager diretas → ditutup oleh CSP.
  2. *Clickjacking / iframe tiruan* → ditutup oleh `frame-ancestors 'none'` / `X-Frame-Options: DENY`.
  3. *Peniruan konten (plagiat)* → hanya bisa ditangani hukum (DMCA), bukan teknis.
- **[SELESAI] Pilih stack aman.** Situs statis murni HTML/CSS/JS tanpa backend —
  secara arsitektur ini pilihan paling aman: tidak ada server-side code yang
  bisa dieksploitasi. (Framework berat justru menambah permukaan serangan.)
- **[TUGAS KAMU] Email di `mailto:`** — pertimbangkan pakai email khusus publik
  (bukan email pribadi utama) untuk mengurangi spam scraping.

## 2. Pengodean Aman (Secure Coding)

- **[N/A] Validasi & sanitasi input** — tidak ada form/endpoint yang menerima
  input user.
- **[N/A] Hashing password / MFA / RBAC / session** — tidak ada autentikasi.
- **[SELESAI] XSS** — CSP membatasi sumber script hanya `'self'` +
  inline terkontrol; `object-src 'none'` memblokir plugin/flash. Semua konten
  adalah teks statis, tidak ada rendering data user.
- **[SELESAI] CSRF** — `form-action 'none'` di CSP: browser dilarang
  mengirim form dari halaman ini ke mana pun.
- **[SELESAI] External resources** — hanya Google Fonts (HTTPS) dan dua gambar
  eksternal (Pinterest). Semua sumber lain diblokir `default-src 'self'`.
- **[SELESAI] Deterrent peniruan** — klik kanan & shortcut view-source
  diblokir, gambar anti-drag, notice hak cipta di console. (Menghalau peniru
  biasa — bukan pengaman absolut.)

## 3. Pengujian Keamanan (Testing)

- **[TUGAS KAMU] Validasi header** — setelah deploy, cek dengan:
  - https://securityheaders.com (masukkan URL domain kamu) — target minimal **A**
  - https://observatory.mozilla.org — target minimal **A**
- **[TUGAS KAMU] Pentest ringan** — jalankan OWASP ZAP quick scan (gratis)
  terhadap domain. Untuk situs statis, temuan yang diharapkan: nol celah.
- **[N/A] SAST** — tidak ada dependensi/library pihak ketiga yang bisa discan
  (hanya font dari CDN). Tidak ada `package.json` → tidak ada supply chain.

## 4. Konfigurasi Server & Deployment

- **[SELESAI] HTTPS/SSL** — siapkan `Strict-Transport-Security` di file
  `_headers` (Netlify) / `vercel.json` (Vercel). SSL otomatis disediakan
  hosting; HSTS memaksa browser selalu pakai HTTPS.
- **[SELESAI] Anti-iframe & hardening header** — dua file konfigurasi siap:
  - `_headers` → untuk deploy ke **Netlify** (upload/taruh di root, otomatis terbaca)
  - `vercel.json` → untuk deploy ke **Vercel** (taruh di root)
  Header yang dipasang: `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy`,
  `Permissions-Policy`, HSTS, dan CSP lengkap dengan `frame-ancestors 'none'`
  (versi meta tag di HTML tidak mendukung frame-ancestors — header server yang
  berlaku).
- **[N/A] WAF / hardening server** — tidak ada server yang kamu kelola;
  Netlify/Vercel/GitHub Pages sudah menangani DDoS & TLS di edge. Kalau mau
  lapisan ekstra, bisa taruh domain di belakang Cloudflare (gratis).
- **[TUGAS KAMU] Git hygiene** — jangan commit file/folder rahasia (`.env`,
  key API) ke repo. Situs ini tidak punya secret, tapi biasakan `.gitignore`
  berisi `.env*` sejak awal.

## 5. Pemeliharaan & Monitoring

- **[TUGAS KAMU] Patching** — hampir nol: tidak ada framework/library yang
  perlu di-update. Satu-satunya dependensi eksternal adalah Google Fonts
  (dikelola Google).
- **[TUGAS KAMU] Monitoring** — aktifkan analytics ringan
  (Cloudflare Web Analytics / GoatCounter — tanpa cookie, gratis) untuk
  mendeteksi lonjakan trafik janggal.
- **[TUGAS KAMU] Backup** — situs ini satu file `index.html`; cukup simpan
  salinan di repo Git privat. Git history juga jadi bukti tanggal karya
  (penting kalau ada yang meniru → klaim DMCA).
- **[TUGAS KAMU] Cek berkala** — ulangi scan securityheaders.com setiap
  beberapa bulan, dan pastikan domain + hosting tidak expired
  (domain kedaluwarsa yang diambil orang lain = risiko terbesar situs statis).

---

## Langkah deploy yang disarankan (urutan)

1. Commit semua file ke repo Git (index.html, _headers atau vercel.json,
   robots.txt, SECURITY.md ini — sesuaikan dengan hosting pilihanmu).
2. Deploy: Netlify (drag-and-drop folder, atau hubungkan repo) ATAU Vercel.
3. Verifikasi: buka domain → gembok HTTPS aktif → cek securityheaders.com →
   minimal grade A.
4. Simpan bukti: screenshot hasil scan + tanggal (arsip keamanan pribadi).
