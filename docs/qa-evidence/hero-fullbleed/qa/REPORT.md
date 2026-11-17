# QA Report — Redesign Hero Full-Bleed "Alam Sanggar Indah"

**Tanggal:** 2026-11-14 · **QA:** `qa-agent` · **Target:** `components/HeroSection.vue` + `data/content.js` (`hero.subline`)
**Baseline spec:** `docs/superpowers/specs/2026-11-14-redesign-hero-asi-fullbleed.md` (AC-1..AC-7)
**Metode:** build produksi (`npm run build` + `node .output/server/index.mjs`) lalu **Chrome headless via CDP** (bukan Playwright — Playwright tidak terpasang). Skrip harness sementara di `%TEMP%` (`qa-visual.mjs`, `qa-runtime.mjs`, `qa-followup.mjs`, `qa-contrast.py`), **tidak ada perubahan kode aplikasi**.

---

## Status: **FAIL**

Satu acceptance criterion dilanggar: **AC-3 (kontras)** pada viewport **768×1024** — subline 16px turun ke **3.24:1** (syarat 4.5:1) di area awan terang. Semua kriteria lain PASS.

| Item | Hasil |
|---|---|
| Build | **PASS** |
| Visual & responsif | **PASS** |
| Kontras | **FAIL** (768×1024 subline) |
| Runtime browser | **PASS** |
| Regresi section bawah | **PASS** |

---

## 1. Build Check — PASS

- `npm run build` exit code **0**; Nuxt 3.21.8 / Nitro 2.13.4 / Vite 7.3.6, 174 modul, output total 2.22 MB (546 kB gzip).
- Server produksi: `node .output/server/index.mjs` → `Listening on http://[::]:3000`.
- **Warning yang sudah ada sebelumnya (bukan regresi):** `(node:19292) [DEP0155] DeprecationWarning: Use of deprecated trailing slash pattern mapping "./" in the "exports" field ... node_modules/@vue/shared/package.json` — berasal dari dependency, bukan kode hero. Tidak ada error/warning build lain.
- Audit CSS hasil build (Tailwind di-inline di HTML SSR): kelas scrim benar-benar ter-render —
  `.from-black/85` (#000000d9), `.from-black/55` (#0000008c), `.lg:from-black/70` (#000000b3), `.via-black/60` (#0009), `.sm:via-black/50` (#00000080), `.lg:via-black/55`, `.via-black/45`, `.lg:via-black/30`, `.via-black/25`, `.sm:via-black/50`, `.to-black/45`. Tidak ada kelas opacity `/82`/`/62` yang dibuang. Computed style di browser: top `linear-gradient(rgba(0,0,0,.55), rgba(0,0,0,.25), transparent)`, kiri `linear-gradient(to right, rgba(0,0,0,.85), rgba(0,0,0,.55), transparent)`, bawah `linear-gradient(to top, rgba(0,0,0,.7), rgba(0,0,0,.3), transparent)`.

## 2. Metode & Cakupan Playwright Check — Chrome CDP headless

- Chrome `C:\Program Files\Google\Chrome\Application\chrome.exe` mode `--headless=new --remote-debugging-port=9333/9334/9335`, WebSocket `ws` bawaan `node_modules`/global; tanpa dependency baru.
- **Viewport screenshot & metrik:** 1440×900, 1280×800, 1024×768, 768×1024, 390×844, 375×667, 360×800, 320×568 (deviceScaleFactor 1, overlay `LoadingScreen` `z-[60]` ditunggu hilang sebelum capture).
- **Skenario:** render awal, 3 state header (atas hero / masih di hero / setelah hero), klik CTA hero, 5 anchor nav, link WhatsApp, menu mobile (buka/Escape/klik link), reduced-motion 1440+375, parallax scroll manual via `Input.dispatchMouseEvent` wheel, section setelah hero, audit a11y/pointer-events, focus ring, kontras piksel, cek semua `<img>` (`naturalWidth`).
- **Kontras:** screenshot kedua dengan teks di-`visibility:hidden` (layout tetap) → background di bawah glyph diambil piksel-per-piksel, teks alpha dikompositkan ke background aktual, rasio WCAG dihitung per piksel glyph (worst, p5, mean). Validasi silang: screenshot builder `docs/qa-evidence/hero-fullbleed/tablet-768.png` **byte-identical** (SHA-256 sama) dengan hasil capture QA → hasil reproducible.

## 3. Bukti Visual

Semua di `docs/qa-evidence/hero-fullbleed/qa/`:

| File | Isi |
|---|---|
| `desktop-1440x900.png`, `laptop-1280x800.png`, `tablet-1024x768.png`, `tablet-portrait-768x1024.png`, `mobile-390x844.png`, `mobile-375x667.png`, `mobile-360x800.png`, `mobile-320x568.png` | screenshot viewport baru (8 ukuran) |
| `bg-<viewport>.png` (8 file) | background-only untuk sampling kontras |
| `runtime-desktop-1440x900.png`, `runtime-mobile-390x844.png` | state runtime awal |
| `runtime-header-over-hero-1440x900.png`, `runtime-header-scrolled-1440x900.png` | header transparan vs putih solid |
| `runtime-cta-target-tipe-1440x900.png` | hasil klik CTA hero → `#tipe` |
| `runtime-mobile-menu-open-390x844.png`, `runtime-mobile-menu-target-390x844.png` | menu mobile buka & setelah pilih nav |
| `runtime-cta-focus-hero-1440x900.png` | focus ring CTA (hero terlihat) |
| `runtime-reduced-motion-1440x900.png`, `runtime-reduced-motion-375x667.png` | prefers-reduced-motion |
| `runtime-parallax-1440-scroll~150.png`, `runtime-parallax-1440-scroll~450.png` | scroll manual wheel |
| `runtime-sections-after-hero-1440x900.png` | TrustStrip + KprPartnerStrip |
| `header-bg-1440x900.png`, `header-bg-375x667.png` | background-only header (kontras) |
| `recheck-768x1024.png`, `bg-recheck-768x1024.png` | reproduksi temuan kontras 768 |
| `debug-768-subline-text.png`, `debug-768-subline-bg.png` | crop teks vs background area gagal |
| `metrics-visual.json`, `runtime-report.json`, `followup-report.json`, `contrast-report.json` | data mentah pengukuran |

## 4. Hasil per Acceptance Criteria

### AC-1 Komposisi full-bleed — PASS
- 1440×900: hero tepat **1440×900** (= `100dvh`), sky `inset-x-0 -top-10% h-120%` menutup 0..900 tanpa celah; rumah `x=432 w=1368 bottom=940` (bleed `-bottom-10`), mask kiri `linear-gradient(90deg, transparent 0, black 22%)` ter-render.
- Tidak ada tipografi raksasa di langit: font terbesar di hero = **82.08px** (headline 3 baris, `font-family: Poppins, sans-serif`); `giantTitle` tidak dipakai.
- Screenshot `desktop-1440x900.png` menunjukkan foto penuh, rumah kanan-bawah, copy kiri.

### AC-2 Kolom kiri lengkap — PASS
Urutan & geometri 1440×900: badge y162.9 (`Alam Sanggar Indah`, 11px, border white/45) → h1 y229.4 (3 baris: HUNIAN ASRI / DI JANTUNG / PARAHYANGAN, 82.08px/600, leading 73.87) → subline y474.9 (22.32px, max-w 480) → CTA y575.6 tinggi **62px** `href="#tipe"` → divider y685.6 (`border-top 1px`) + label "Mitra Pembiayaan KPR Resmi" y706.6 + logo **BCA** & **BTN** (`naturalWidth=300`, tinggi kotak 40/46, dipisah garis `1px white/30`).
Di 390×844 urutan sama, logo bank bottom 672 < 844 (tidak terpotong).

### AC-3 Keterbacaan — **FAIL** (satu kondisi)
Syarat: teks kecil ≥4.5:1, headline besar ≥3:1. Hasil (worst / p5 / mean):

| Elemen | 1440 | 1280 | 1024 | **768×1024** | 390 | 375 | 360 | 320 | Threshold |
|---|---|---|---|---|---|---|---|---|---|
| Badge 11px | 9.30 | 9.79 | 8.15 | 7.78 | 7.10 | 7.51 | 6.79 | 8.09 | 4.5 ✓ |
| Headline L1 | 5.21 | 6.52 | 6.66 | 7.42 | 7.95 | 6.02 | 8.33 | 7.23 | 3.0 ✓ |
| Headline L2 | 5.39 | 6.75 | 7.32 | 7.13 | 6.69 | 6.82 | 6.62 | 5.93 | 3.0 ✓ |
| Headline L3 "PARAHYANGAN" | 4.45 | 5.63 | 4.85 | 6.34 | 5.59 | 5.86 | 5.76 | 5.56 | 3.0 ✓ |
| Subline 16–22px | 7.56 | 8.82 | 6.68 | **3.24** | 6.16 | 5.39 | 5.19 | 5.46 | 4.5 ✗ (768) |
| Label KPR 10–11px | 14.00 | 14.25 | 13.98 | 12.62 | 11.22 | 11.22 | 11.12 | 11.12 | 4.5 ✓ |
| CTA (analitik #141a1c/#fff) | 17.58 | — | — | — | — | — | — | — | 4.5 ✓ |
| Header nav 12px (atas hero) | 8.38–14.37 | — | — | — | — | — | — | — | 4.5 ✓ |
| Header logo text | 17.66 | — | — | — | — | — | — | — | 4.5 ✓ |

**Temuan #1 (blocking):** di **768×1024**, subline `Temukan kenyamanan hunian modern yang berpadu dengan suasana alam.` menimpa pita awan terang. Titik terburuk pada ekor baris 1 ("yang berpadu"), sekitar **x≈434–443, y≈503–509**: background `rgb(155,129,91)`, teks `rgba(255,255,255,.9)` → **3.24:1** (p5 4.35, mean 9.81). Diperlukan 4.5:1 untuk teks 16px. Scrim kiri di titik itu hanya ~40% (gradien berakhir di 86% lebar) dan scrim bawah mulai di y≈532 (52% tinggi), sehingga awan terang lolos. Terukur dua kali (run visual awal dan run recheck terpisah) dengan angka identik → bukan artefak sesaat. Bukti: `tablet-portrait-768x1024.png`, `bg-recheck-768x1024.png`, `debug-768-subline-text.png`, `debug-768-subline-bg.png`.

Catatan: area headline "PARAHYANGAN" yang menimpa dinding/atap rumah di 1440 tetap **4.45:1** (≥3:1 untuk headline 82px) → lolos.

### AC-4 Responsif — PASS
- `document.documentElement.scrollWidth == window.innerWidth` di **semua 8 viewport** (1440/1280/1024/768/390/375/360/320) → nol horizontal scroll. `offenders` hanya elemen dekoratif di dalam hero `overflow-hidden` (rumah `w=152vh` dll.).
- Headline selalu tepat **3 baris** (`min-h` 100dvh + clamp), ukuran mengecil 82.08→33.6px.
- CTA tidak terpotong: right 400.3 (1440), 273.7 (320); tinggi tetap 62px.
- Logo bank sejajar (`flex items-center`), BCA h40 / BTN h46, bottom 546.8 di 320 (< viewport 568).
- Badge tidak tertutup header: jarak badge-top − header-bottom = 19px (320), 64.4px (375), 85.9px (1440).
- Catatan minor: di 320×568 tinggi hero **586.77px** (18.8px > viewport) karena content + padding; seluruh teks/logo tetap terlihat (bottom 546.8), hanya padding bawah yang di bawah fold. Tidak melanggar AC-4.

### AC-5 Konten terpusat — PASS
Teks render **persis** sama dengan `data/content.js`: badge `Alam Sanggar Indah`, headline `["HUNIAN ASRI","DI JANTUNG","PARAHYANGAN"]`, subline, CTA `Jelajahi Pilihan Rumah`, label `Mitra Pembiayaan KPR Resmi`, alt `Logo Bank BCA`/`Logo Bank BTN`. Tidak ada teks jualan hardcode baru di template hero (semua via `content.*`); `hero.giantTitle` tidak direferensikan.

### AC-6 Header tetap adaptif — PASS / tidak regresi
- Di atas hero: `rgba(0,0,0,0.15)` + `backdrop-blur(4px)`, teks putih / `white/90`, border `white/25`; kontras nav **8.38–14.37**, logo **17.66**.
- Setelah scroll melewati hero (scrollY 1150): `bg rgb(255,255,255)`, teks `rgb(17,17,17)`, border `black/10`, shadow aktif → tetap berubah putih solid seperti sebelumnya.

### AC-7 Build & runtime bersih — PASS
- Browser (8 viewport + semua skenario): **console errors 0, console warnings 0, pageErrors 0, request 404/failed 0**.
- Semua aset hero & bank termuat: `hero-bg.jpg` nw=1376, `hero.webp` nw=1920, `bca-white.png` nw=300, `btn-white.png` nw=300 (tidak ada broken image). File lazy di bawah fold yang belum ter-decode terverifikasi ada di disk (bukan 404).
- Reduced-motion: `matchMedia` true, badge/h1/subline/CTA/blok KPR `opacity:1`, `transform:none`, tanpa inline style animasi, konten tampil penuh (1440 & 375).
- Catatan: stdout server produksi memuat `[Vue Router warn] No match found for location with path "/_nuxt/"` dan `/.well-known/appspecific/com.chrome.devtools.json` — berasal dari probe DevTools Chrome, bukan aset halaman; tidak ada request 404/gagal di log jaringan browser dan tidak terkait kode hero.

## 5. Uji Interaksi & Runtime lain — PASS

- **CTA hero:** klik nyata pada `Jelajahi Pilihan Rumah` → `location.hash === "#tipe"`, `scrollY 2038`, posisi `#tipe` top **88.4px** (di bawah header 77px) dan terlihat → scroll ke section Pilihan Rumah yang benar.
- **Nav header:** Tentang Kami → `#tentang` (top 88.0), Pilihan Rumah → `#tipe` (88.4), Simulasi KPR → `#kalkulator` (167.6; `scroll-mt` 80 + offset), Keunggulan → `#keunggulan` (87.6), FAQ → `#faq` (168.1). Semua hash cocok, target terlihat, bukan header.
- **Kontak:** `https://wa.me/6282118992299?text=Halo%2C%20saya%20lihat%20informasi%20di%20website...`, `target="_blank"`, `rel="noopener noreferrer"` (desktop & menu mobile).
- **Menu mobile:** buka → `display:block` tinggi 334px, header `z-50` > z maksimum hero `30`, link pertama bisa di-hit (tidak tertutup hero); Escape menutup (`display:none`, `aria-expanded=false`); klik "Pilihan Rumah" menutup menu + scroll ke `#tipe` (top 87.9).
- **Parallax scroll manual (wheel, bukan programatik):** scrollY 150/300/450 → sky `y=14.4/28.8/43.2`, rumah `y=-4.33/-8.67/-13`, konten `y=5/10/15` (monoton, tanpa lompatan). Tepi bawah hero selalu menyatu dengan TrustStrip: heroBottom == trustTop (750/750, 600/600, 450/450) → **tidak ada celah**.
- **A11y:** `<h1>` total = **1** (di hero), gambar dekoratif `alt=""` + ancestor `aria-hidden=true`, logo bank punya `alt`; CTA fokusable dengan `:focus-visible` true → outline **2px solid rgb(83,107,85) offset 2px** terlihat (screenshot); semua layer dekoratif hero `pointer-events:none`; `elementFromPoint` di CTA/badge/logo/nav mengembalikan elemen yang benar.
- **Regresi section bawah:** pada scrollY 860 (hero bottom 40): TrustStrip top **40** (menyambung, tanpa gap), tinggi 78, opacity 1, teks utuh; KprPartnerStrip top 118, tinggi 50, opacity 1, logo bank tampil. Tidak tertimpa/terpotong.

## 6. Temuan

**Blocking (FAIL):**
1. `HeroSection.vue` — subline di **768×1024** kontras **3.24:1** (<4.5:1) karena menimpa awan terang; terjadi di baris 1 ekor "yang berpadu" (x≈434–443, y≈503–509). Melanggar **AC-3**. Hanya viewport ini; 1024 ke atas dan 390 ke bawah subline ≥5.19:1.

**Minor (tidak memblokir):**
2. Tinggi efektif logo bank: BCA ≈**36.6px** (kotak 40px, tinta 97/106), BTN ≈**39.9px** (kotak 46px, tinta 163/188) — spec §3.2 menyebut BCA ≈40 / BTN ≈36, jadi BTN ~9% lebih tinggi dari BCA (kebalikan angka target), meski secara visual seimbang di screenshot. AC-2 tetap PASS (keduanya ada, berurutan, tidak terpotong).
3. 320×568: hero 586.77px (≈18.8px lebih tinggi dari viewport) akibat content+padding; tidak ada konten terpotong, hanya padding bawah di bawah fold.
4. FAB WhatsApp (komponen lama, out of scope) di 320px berimpit sangat dekat logo BTN: FAB x 244–299 vs BTN right 242.6 → **~1.4px** jarak, tidak menimpa; terasa sesak.
5. Deviasi visual vs mockup `hero-asi-reference.jpg` (tidak melanggar AC): langit/rumah lebih gelap & hangat (warm grade + scrim), subjek/framing rumah dari aset existing berbeda dari mockup, dan padding kiri memakai `.container` (132px @1440 ≈9.2%) vs mockup ~6% — deviasi ini sudah didokumentasikan di spec §3.2. Ukuran headline 82.08px @1440 (mockup proporsional ~91px) sedikit lebih kecil.

## 7. Rekomendasi Perbaikan (untuk frontend-agent)

1. **Perkuat scrim area copy pada breakpoint `sm`/tablet** (perbaikan utama AC-3): opsi termurah — ubah scrim kiri agar tetap pekat sampai melewati lebar kolom copy, mis. di `sm` jadikan `sm:w-[100%] sm:via-black/60 sm:to-black/25` (atau tambah stop ketiga `sm:to-black/15`), **atau** tingkatkan scrim bawah `h-[48%]` → mulai lebih tinggi / `via-black/55` khusus `sm` saja, **atau** beri scrim lokal tipis di belakang blok copy (`bg-gradient-to-r from-black/45 to-transparent` pada wrapper copy) agar tidak mengubah komposisi desktop. Target: subline 16px ≥ **4.7:1** (margin aman) di 768×1024 tanpa membuat sky kanan ikut gelap.
   - Verifikasi ulang: 768×1024 + 700×900/820×1180 (tablet portrait lain) — karena masalah muncul saat sky framing bergeser.
2. Opsional (minor): sesuaikan `bankHeightClass` BTN 46px → ~42px bila ingin tinggi efektif BTN < BCA sesuai spec; tidak wajib secara visual.
3. Opsional (minor): beri `pb`/`min-h` yang tidak memaksa hero > viewport di 320×568 (mis. kurangi `pt` mobile atau `pb`), atau biarkan karena tidak memotong konten.

---

**Bukti ringkas yang bisa dicek ulang:** `metrics-visual.json` (8 viewport), `contrast-report.json` (semua angka kontras), `runtime-report.json` (header/nav/CTA/menu/parallax/a11y/sections), `followup-report.json` (kontras header, focus, recheck 768), dan screenshot di direktori ini.
