# Spec-Lite: Global Typography & Palette Redesign — Alam Sanggar Indah

**Tanggal:** 2026-11-17 · **Status:** `selesai` (Lulus `npx nuxi generate` & Playwright QA pada `1280x800`, `768x1024`, `375x667`) · **Agent:** `doc-agent`

---

## 1. Latar Belakang & Tujuan

Identitas visual lama masih memakai pasangan font *Dream Orphans* + *Poppins*, palet hijau-krem lama, serta sejumlah nilai warna hex dan `rgba()` yang di-*hardcode* di dalam komponen. Untuk menyelaraskan seluruh halaman dengan karakter visual baru **Alam Sanggar Indah**, dilakukan pembaruan sistem tipografi dan palet warna secara terpusat melalui token CSS dan konfigurasi Tailwind.

**Tujuan:**
- Menerapkan hierarki tipografi 3-peran (`Ptarmigan` untuk display, `Fraunces` untuk editorial serif, `Bricolage Grotesque` untuk body/UI).
- Memperbarui palet warna global bernuansa kertas terang netral, biru aksen utama, dan aksen zaitun pada eyebrow.
- Membersihkan warna hex *hardcoded* pada komponen agar mengonsumsi token CSS global dan memperbaiki parameter *variable font range* Google Fonts CSS2 API.

---

## 2. Arsitektur & Komponen

1. **Sistem Tipografi 3-Peran (`assets/css/tokens.css`, `assets/css/base.css`, `tailwind.config.js`, `nuxt.config.ts`):**
   - **Display (`font-display` / `--display`):** `'Ptarmigan', 'Bricolage Grotesque', sans-serif` — dimuat lokal via `@font-face` (`assets/fonts/ptarm__.ttf` normal `400 700` dan `assets/fonts/ptarmi_.ttf` italic `400 700`, `font-display: swap`).
   - **Editorial Serif (`font-serif` / `--serif` / `.serif`):** `'Fraunces', Georgia, serif` (`.serif` dengan `font-weight: 500; letter-spacing: -0.025em;`).
   - **Body & UI (`font-body` / `font-sans` / `--sans`):** `'Bricolage Grotesque', sans-serif` sebagai font default `body`.
   - **Google Fonts CSS2 API (`nuxt.config.ts`):** Menggunakan parameter range axis yang valid: `Bricolage+Grotesque:opsz,wght@12..96,400..700` dan `Fraunces:ital,opsz,wght@0,9..144,400..700;1,9..144,400..700`.

2. **Palet Warna Global (`assets/css/tokens.css`, `assets/css/base.css`, `tailwind.config.js`):**
   - Permukaan & Teks: `--paper: #f5f5f5`, `--paper-deep: #eaedf1`, `--white: #ffffff`, `--ink: #1a1916`, `--muted: #525353`, `--line: rgba(26, 25, 22, 0.14)`, `--shadow: 0 22px 60px rgba(26, 25, 22, 0.10)`.
   - Aksen Utama & Sekunder: `--green: #2a63b4`, `--green-deep: #1e4680`, `--green-soft: #cfe2ee`, `--sage-pale: #e4eff5`, `--olive: #48601c` / `--sand: #48601c` (`olive` didaftarkan pada `tailwind.config.js` dan dipakai `.eyebrow` di `base.css`; `.btn-primary` memakai shadow biru `rgba(42, 99, 180, .20)` / hover `rgba(30, 70, 128, .26)`).

3. **Pembersihan Hex Hardcoded pada Komponen:**
   - Mengganti hex dan shadow warna lama dengan token CSS global (`var(--white)`, `var(--sage-pale)`, `var(--green-deep)`, `var(--green)`, `var(--muted)`) serta shadow selaras `--ink` / `--green` pada 12 komponen: `AboutSection.vue`, `BaseButton.vue`, `FinalCtaSection.vue`, `HeroSection.vue`, `KprTermsModal.vue`, `LocationSection.vue`, `MemberGetMemberModal.vue`, `ProductsSection.vue`, `ReferralWelcomeModal.vue`, `TheFooter.vue`, `WhatsAppFloatingButton.vue`, dan `WhySection.vue`.

---

## 3. Acceptance Criteria (Given / When / Then)

- **AC-1 (Tipografi 3-Peran & Google Fonts):** *Given* aplikasi dijalankan atau di-generate, *When* stylesheet dan `head` dimuat, *Then* `Ptarmigan`, `Fraunces`, dan `Bricolage Grotesque` terpetakan konsisten pada token `--display`/`--serif`/`--sans` serta utilitas Tailwind `font-display`/`font-serif`/`font-body`/`font-sans`, dan URL Google Fonts CSS2 API menggunakan syntax range (`12..96`, `9..144`, `400..700`) tanpa error.
- **AC-2 (Palet Global & Token Komponen):** *Given* seluruh section dan modal di `pages/index.vue`, *When* dirender di viewport desktop (`1280x800`), tablet (`768x1024`), dan mobile (`375x667`), *Then* warna permukaan, teks, aksen biru, dan eyebrow zaitun bersumber dari token global pada `:root` tanpa sisa palet hijau lama yang bertabrakan.
- **AC-3 (Verifikasi Build & QA):** *Given* perubahan token, tipografi, dan komponen selesai, *When* dijalankan `npx nuxi generate` dan Playwright QA pada `1280x800`, `768x1024`, serta `375x667`, *Then* proses build dan pemeriksaan visual/layout seluruhnya lulus.

---

## 4. Out of Scope

- Perubahan struktur HTML/layout section, konten data di `data/content.js`, maupun penambahan rute atau fitur fungsional baru.
