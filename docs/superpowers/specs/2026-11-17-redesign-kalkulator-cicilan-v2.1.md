# Spec-Lite: Redesign Kalkulator Cicilan KPR v2.1

**Tanggal:** 2026-11-17 · **Status:** `selesai (PASS)` (Lulus `npx nuxi generate` & Playwright QA pada `1280x800`, `768x1024`, `375x667`) · **Agent:** `doc-agent`

---

## 1. Latar Belakang & Tujuan

Section kalkulator KPR sebelumnya (`2026-11-11-kalkulator-cicilan-design.md`) masih terikat pada skema bunga tetap 5% khusus Bank BCA, menampilkan accordion biaya akad awal, serta memakai pesan WhatsApp yang menyebut `Palastri`. Untuk menyelaraskan `components/MortgageCalculatorSection.vue` dan `data/content.js` dengan standar **Kalkulator Cicilan KPR v2.1** (`docs/guide-kalkulator-cicilan.md`) di atas tema aktif **Light-Blue & Crisp White** (`assets/css/tokens.css`), dilakukan penyederhanaan antarmuka (*lean UX*) menggunakan latar section `bg-white` polos tanpa gambar 3D (Opsi A) agar calon pembeli fokus pada simulasi cicilan dengan input suku bunga fleksibel.

---

## 2. Arsitektur & Komponen

### A. Data Layer (`data/content.js`)
- **`meta.whatsapp.templates.simulasiKpr`:** Dinetralkan agar generik menggunakan placeholder `{project}` dan `{bunga}` (`Suku Bunga: {bunga} p.a.`) tanpa penyebutan `Palastri` maupun `Bank BCA`.
- **Node `calculator`:**
  - Properti baru/diperbarui: `eyebrow: ''`, `defaultPrice: 166000000`, `defaultDpPercent: 5`, `defaultTenor: 20`, `defaultInterestRatePercent: 5`, `minDpPercent: 5`, `maxDpPercent: 90`, `tenors` (5, 10, 15, 20 tahun), `footnote` netral (*patokan/estimasi tanpa keterikatan bank tertentu*), serta objek `labels` v2.1 (`price`, `dp`, `dpPercent`, `interestRate`, `tenor`, `plafon`, `installment`, `perMonth`, `minDp`, `midDp`, `maxDp`, `metricInterest`, `metricDp`, `metricTenor`, `clearPrice`).
  - Pembersihan properti lama: menghapus `annualInterestRate`, `provisionRate`, `adminFee`, `presets`, `image`, `badge`, `metrics` statis, `bankPartner`, dan seluruh label biaya akad.

### B. Komponen UI (`components/MortgageCalculatorSection.vue`)
- **Heading & Latar Section:** Section `#kalkulator` memakai latar `bg-white` polos tanpa gambar latar 3D, dengan heading rata tengah (`text-center`) tanpa bar aksen dan tanpa eyebrow bawaan (`v-if="calc.eyebrow"`).
- **Kolom Input (Kiri — `lg:col-span-6`):**
  - Input Harga Rumah (`#calc-price-input`) dengan format ribuan `id-ID` dan tombol reset cepat `✕` (`clearPrice`).
  - Sinkronisasi DP 3-arah antara nominal Rupiah (`#calc-dp-nominal`), persentase (`#calc-dp-percent`), dan slider range (`5%`–`90%`) dengan *dynamic track gradient* (`sliderBackgroundStyle` berbasis `--green` & `--paper-deep`).
  - Sub-grid 2 kolom responsif (`grid-cols-1 sm:grid-cols-2 gap-4`) untuk input Suku Bunga manual (`#calc-interest-rate`, `% p.a.`, proteksi label `truncate` & badge `shrink-0 whitespace-nowrap`) dan pilihan Tenor (`#calc-tenor-select`).
- **Kolom Hasil & CTA (Kanan — `lg:col-span-6`) serta Pill Disclaimer:**
  - Ringkasan Plafon Pembiayaan KPR dan box estimasi cicilan bulanan (`monthlyInstallment`) yang bersih tanpa badge suku bunga statis.
  - Grid 3 metrik dinamis real-time (`Bunga KPR`, `Uang Muka`, `Tenor KPR`) yang otomatis mengikuti perubahan input.
  - Eliminasi accordion rincian biaya akad awal; tombol CTA WhatsApp *full-width* (`waConsultLink`) mengirim `{harga}`, `{dp}`, `{dpPercent}`, `{plafon}`, `{bunga}`, `{tenor}`, dan `{cicilan}`.
  - Wadah pill disclaimer netral di luar card (`bg-[var(--white)]/80 backdrop-blur-sm`) tanpa logo bank.

---

## 3. Acceptance Criteria & Ringkasan QA

### Acceptance Criteria (Given / When / Then)
- **AC-1 (Tampilan Awal & Default v2.1):** *Given* pengguna membuka `#kalkulator`, *When* komponen dirender, *Then* section tampil berlatar `bg-white` polos dengan heading rata tengah tanpa eyebrow, nilai awal `Rp 166.000.000` / DP `5%` (`Rp 8.300.000`) / Bunga `5% p.a.` / Tenor `20 Tahun`, estimasi `Rp 1.040.750 /bulan`, 3 metrik dinamis aktif, tanpa accordion biaya akad, dan pill disclaimer netral di bawah card.
- **AC-2 (Sinkronisasi DP & Suku Bunga Dinamis):** *Given* form simulasi aktif, *When* pengguna mengubah harga rumah, mengetik nominal/persen DP, menggeser slider, atau mengganti suku bunga `#calc-interest-rate` dan tenor `#calc-tenor-select`, *Then* ketiga kendali DP tersinkronisasi beserta gradien track slider, 3 metrik ringkasan dan cicilan anuitas terhitung ulang instan (termasuk kasus bunga `0%`), serta tautan CTA WhatsApp memuat parameter `{bunga}` terbaru.
- **AC-3 (Responsivitas & Build):** *Given* pengujian pada viewport desktop, tablet, dan mobile, *When* dilakukan build statis dan inspeksi browser, *Then* sub-grid `sm:grid-cols-2` tampil sejajar di `sm+` dan bertumpuk rapi di mobile tanpa *horizontal overflow* maupun console error.

### Ringkasan Hasil Verifikasi QA (`qa-agent`)
- **Status Keseluruhan:** `selesai (PASS)`
- **Build Statis:** `npx nuxi generate` lulus tanpa error.
- **Playwright QA:** **PASS** pada viewport `1280x800`, `768x1024`, dan `375x667` (`0` console errors, `0` horizontal overflow).
- **Bukti Screenshot:** `docs/qa-evidence/kalkulator-v2/desktop-1280.png` dan `docs/qa-evidence/kalkulator-v2/mobile-375.png`.

---

## 4. Out of Scope

- Tidak mengubah logika helper maupun pemetaan data pada `composables/useContent.js`.
- Tidak mengubah token warna maupun tipografi global pada `assets/css/tokens.css`.
