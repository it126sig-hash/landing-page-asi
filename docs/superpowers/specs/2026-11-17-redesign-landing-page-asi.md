# Spec-Lite: Redesign Landing Page Alam Sanggar Indah (Light-Blue & Crisp White)

**Tanggal:** 2026-11-17 · **Status:** `selesai` (Lulus `npx nuxi generate` & Playwright QA pada `1440x900`, `1280x800`, `768x1024`, `375x667`) · **Agent:** `doc-agent`

---

## 1. Latar Belakang & Tujuan

Tampilan landing page sebelumnya masih menggabungkan nuansa editorial bernada hangat dengan blok transisi seperti `TrustStrip`, `KprPartnerStrip`, serta scrim gelap pada bagian Hero. Untuk menghadirkan pengalaman visual yang lebih cerah, lapang, terstruktur, dan modern sesuai acuan layout referensi **light-blue & crisp white**, dilakukan redesign menyeluruh pada halaman utama **Alam Sanggar Indah** (`pages/index.vue`) beserta sistem token dan section-section pendukungnya.

**Tujuan:**
- Mengaplikasikan estetika **light-blue & crisp white** melalui ritme latar bergantian antara putih bersih (`#ffffff`) dan biru pucat (`--sage-pale: #eef5fc`) dengan warna tinta biru gelap (`--ink: #17253b`) dan aksen biru cerah (`--green: #2b6cc4`).
- Menyederhanakan alur informasi pada `pages/index.vue` menjadi 9 section utama yang fokus pada konversi: Hero cerah dengan kurva gelombang putih, Tentang Kami (+ grid keunggulan 2×2), Pilihan Rumah asimetris, Lokasi (+ Google Maps), Fasilitas Lengkap, Kalkulator KPR, Artikel & Berita, FAQ, dan Banner CTA Konsultasi.
- Menambahkan section baru **Fasilitas Lengkap** (`FacilitiesSection.vue`) dan **Berita & Artikel Terbaru** (`ArticlesSection.vue` beserta modal pratinjau) serta memperbarui **TheFooter.vue** dengan layout 4 kolom, input newsletter WhatsApp, dan tombol Back-to-Top.

---

## 2. Arsitektur & Komponen

### A. Token Global & Data Layer
1. **`assets/css/tokens.css` & `assets/css/base.css`:**
   - Memperbarui palet `:root` menjadi nuansa *light-blue & crisp white*: `--paper: #f4f8fc`, `--paper-deep: #e8f0fa`, `--white: #ffffff`, `--ink: #17253b`, `--muted: #5a667a`, `--green: #2b6cc4`, `--green-deep: #173c72`, `--green-soft: #d2e4fa`, `--sage-pale: #eef5fc`, `--sand: #2b6cc4`, `--olive: #2b6cc4`, `--line: rgba(23, 37, 59, 0.10)`, `--radius: 22px`, dan `--shadow: 0 20px 50px rgba(23, 37, 59, 0.08)`.
   - Menyelaraskan tombol `.btn-primary` dan `.btn-outline` pada `assets/css/base.css` dengan bayangan dan border aksen biru `#2b6cc4` / `#173c72`, serta mempertahankan `.eyebrow`, `.serif`, `.container`, dan animasi scroll `.reveal`.

2. **`data/content.js` & `composables/useContent.js`:**
   - Menambahkan item navigasi `#artikel` pada `meta.nav` dan memperbarui `footer.quickLinks` (mencakup `beranda`, `tentang`, `tipe`, `lokasi`, `fasilitas`, `artikel`).
   - Memperbarui `hero` (`headlineLines`, `subline`, `primaryCta: 'Jelajahi Pilihan Rumah'`, `scrollLabel: 'Scroll untuk menjelajah'`).
   - Menambahkan skema data baru: `whyCards` (4 poin keunggulan untuk grid 2×2 di About), `facilities` (judul, subjudul, gambar utama, dan 5 item fasilitas beserta `iconKey`), `articlesSection` (judul, subjudul, `ctaLabel`, dan 3 item artikel dengan `id`, `date`, `title`, `excerpt`, `content`, `image`), serta properti tambahan pada `locationSection` (`address`, `embedMapUrl`), `contactCta` (`bannerTitle`, `bannerDesc`, `bannerCtaLabel`), dan `footer.newsletter`.
   - Memperbarui `composables/useContent.js` agar memetakan path gambar melalui `withBase()` untuk `facilities.image` dan `articlesSection.items[].image`.

### B. Komponen Layout & Section (`pages/index.vue`)
1. **Header & Hero:**
   - `components/TheHeader.vue`: Header fixed yang transparan dengan teks gelap (`text-[var(--ink)]`) saat di puncak Hero (`scrollY <= 24`), lalu bertransisi menjadi putih solid (`bg-white/95 backdrop-blur-md`) dengan border bawah dan shadow saat di-scroll. Memuat logo + nama proyek di kiri, 5 tautan navigasi di tengah, tombol CTA WhatsApp di kanan, dan menu dropdown mobile berlatar putih solid.
   - `components/HeroSection.vue`: Hero setinggi tepat satu layar (`h-[100dvh] min-h-[580px] sm:min-h-[640px]`) berlatar gradien langit biru cerah (`#CBE3F8` → `#E6F2FC` → `#FFFFFF`), layer gambar langit berwarna biru cerah tanpa scrim gelap, glow matahari lembut di kanan-atas, haze kiri untuk kontras teks gelap, visual deretan rumah (`content.hero.image`) di kanan-bawah, kurva gelombang putih SVG di tepi bawah Hero, copy kiri + CTA pill `#tipe`, blok Mitra Pembiayaan KPR Resmi (`content.hero.bankInfo.label` beserta kartu logo & teks `Bank BCA` dan `Bank BTN` dari `content.banks`) di bawah CTA Hero, serta *scroll hint* `#tentang` (tablet/desktop). Didukung animasi entrance (`houseInnerRef`, `headlineRef`, `sublineRef`, `ctaRef`, `bankRef`, `scrollHintRef`) & parallax GSAP `ScrollTrigger` yang menghormati `prefers-reduced-motion`.

2. **Section Konten Utama:**
   - `components/AboutSection.vue`: Layout 2 kolom (`lg:grid-cols-[0.95fr_1.05fr]`) berlatar putih dengan foto utama `aspect-[4/3]` beraksen bingkai pale-blue di kiri, serta heading, deskripsi, dan grid 2×2 `content.whyCards` (4 kartu putih dengan ikon SVG) di kanan.
   - `components/ProductsSection.vue`: Berlatar `--sage-pale` dengan layout horizontal asimetris (`lg:grid-cols-[0.34fr_0.66fr]`) — kolom kiri berisi judul, subjudul, dan tombol `.btn-outline`, sedangkan kolom kanan berisi grid kartu produk `houseTypes` (2 kolom di `sm+`) yang membuka `components/HouseTypeModal.vue` saat diklik atau diaktifkan via keyboard.
   - `components/LocationSection.vue`: Layout 2 kolom terbuka (`lg:grid-cols-[0.42fr_0.58fr]`) berlatar putih — kolom kiri berisi copy, alamat proyek dengan ikon pin, serta CTA `Buka Google Maps` dan `Atur kunjungan` (`#kontak`); kolom kanan menampilkan `<iframe>` Google Maps embed (`embedMapUrl`) beserta kartu badge alamat mengambang.
   - `components/FacilitiesSection.vue` *(baru)*: Berlatar `--sage-pale`, layout 2 kolom (`lg:grid-cols-[0.42fr_0.58fr]`) dengan foto kawasan `aspect-[4/3]` di kiri dan 5 kartu ikon fasilitas (`Ruang Terbuka Hijau`, `Taman Bermain`, `Keamanan 24 Jam`, `Masjid`, `Fasilitas Olahraga`) di kanan.
   - `components/MortgageCalculatorSection.vue`: Section berlatar putih bersih (`bg-white`) yang memuat satu card landscape 2 kolom untuk simulasi cicilan anuitas 5% khusus KPR Bank BCA (default harga Rp 166 Jt, DP 5%, tenor 20 tahun), sinkronisasi DP 2-arah + slider 5%–90%, rincian biaya akad awal (*collapsible*), CTA WhatsApp dinamis, serta pill disclaimer & logo Bank BCA di bawah card.
   - `components/ArticlesSection.vue` *(baru)*: Berlatar `--sage-pale`, menampilkan header section dengan tombol `Lihat Semua`, grid 3 kartu artikel (`md:grid-cols-3`), serta modal pratinjau artikel (`Teleport` ke `body`) lengkap dengan gambar 16:9, tanggal, konten artikel, penutup tombol `✕` / `Tutup` / backdrop / `Escape`, dan CTA konsultasi WhatsApp.
   - `components/FaqSection.vue`: Layout 2 kolom (`lg:grid-cols-[0.72fr_1.28fr]`) berlatar putih dengan intro *sticky* di kiri dan daftar accordion `FaqItem` di kanan (`f1` default terbuka).
   - `components/FinalCtaSection.vue`: Banner CTA biru gradien *rounded-3xl* (`from-[#2D75D8] to-[#448EEB]`) dengan siluet deretan rumah SVG dan tombol pill putih `Hubungi Kami` yang membuka modal form konsultasi (`Teleport` ke `body`). Modal memanfaatkan composable `useConsultationForm` (validasi nama & nomor WA, integrasi Google reCAPTCHA, state sukses/error/`notConfigured`, serta fallback WhatsApp).

3. **Footer & Pembersihan Section Lama:**
   - `components/TheFooter.vue`: Berlatar `--green-deep` dengan padding bawah ekstra (`pb-24`) agar aman dari `WhatsAppFloatingButton`. Terdiri atas 4 kolom (`Brand`, `Menu`, `Ikuti Kami` dengan 4 ikon sosial media, dan `Dapatkan Informasi Terbaru` berupa form input WhatsApp/email yang mengarahkan ke `wa.me`), ditambah bar bawah copyright dan tombol *Back-to-Top* (`scrollToTop`).
   - **Pembersihan `pages/index.vue`:** Komponen `TrustStrip`, `KprPartnerStrip`, dan `WhySection` tidak lagi dipanggil di `pages/index.vue`.

---

## 3. Acceptance Criteria & Ringkasan Verifikasi QA

### Acceptance Criteria (Given / When / Then)
- **AC-1 (Header & Hero Light-Blue):** *Given* pengguna membuka landing page di posisi paling atas (`scrollY = 0`), *When* `TheHeader.vue` dan `HeroSection.vue` dirender, *Then* header tampil transparan dengan teks gelap yang kontras di atas langit biru cerah Hero setinggi `100dvh` (`h-[100dvh] min-h-[580px] sm:min-h-[640px]`) tanpa scrim gelap, blok mitra bank KPR (`Bank BCA` & `Bank BTN`) tampil terbaca di bawah CTA Hero, dan header berubah menjadi putih solid saat halaman di-scroll (`scrollY > 24`).
- **AC-2 (Alur Section & Konten Baru):** *Given* pengguna menelusuri `pages/index.vue` dari atas ke bawah, *When* seluruh section dimuat, *Then* urutan section mengikuti `Hero` → `About` (2×2 `whyCards`) → `Products` (asimetris + `HouseTypeModal`) → `Location` (embed Google Maps) → `Facilities` (5 kartu ikon) → `MortgageCalculator` (white clean) → `Articles` (3 kartu + modal pratinjau) → `FAQ` → `FinalCta` (banner biru + modal form reCAPTCHA) → `TheFooter` (4 kolom + newsletter WA + Back-to-Top), tanpa merender `TrustStrip`, `KprPartnerStrip`, dan `WhySection`.
- **AC-3 (Interaktivitas Modal & Responsivitas):** *Given* pengguna berinteraksi pada viewport desktop, tablet, dan mobile, *When* modal tipe rumah, modal artikel, modal konsultasi, maupun menu hamburger mobile dibuka/ditutup, *Then* seluruh overlay tampil terbaca dengan kontras baik, tidak terjadi *horizontal overflow*, dan footer memiliki ruang bawah (`pb-24`) yang tidak bertabrakan dengan tombol WhatsApp mengambang.
- **AC-4 (Build Statis & QA Multi-Viewport):** *Given* seluruh implementasi redesign selesai, *When* dijalankan `npx nuxi generate` dan pemeriksaan Playwright QA pada `1440x900`, `1280x800`, `768x1024`, serta `375x667`, *Then* proses generate berhasil tanpa error dan seluruh pemeriksaan visual serta interaksi dinyatakan **PASS**.

### Ringkasan Hasil Verifikasi QA
- **Build Statis:** `npx nuxi generate` lulus (PASS).
- **Playwright QA Multi-Viewport:** Lulus (PASS) pada viewport `1440x900`, `1280x800`, `768x1024`, dan `375x667`.
- **Bukti Screenshot (`docs/qa-evidence/redesign/`):**
  - Hero Refinement (`1440x900`, `1280x800`, `768x1024`, `375x667`): `hero-1440.png`, `hero-1280.png`, `hero-768.png`, `hero-375.png`.
  - Desktop (`1440x900` & `1280x800`): `desktop-1440-top.png`, `desktop-1440-full.png`, `desktop-1440-footer.png`, `desktop-1280-top.png`, `reverify-1280-footer.png`.
  - Tablet (`768x1024`): `tablet-768-top.png`, `tablet-768-footer.png`.
  - Mobile (`375x667`): `mobile-375-top.png`, `mobile-375-menu.png`, `mobile-375-full.png`, `reverify-375-footer.png`.
  - Modal & Accordion Interaktif: `modal-house-type.png`, `modal-article.png`, `modal-consultation.png`, `modal-kpr-terms.png`, `faq-projects.png`.

---

## 4. Out of Scope

- Halaman arsip & detail artikel penuh (`/blog` dan `/blog/[slug]`) serta integrasi CMS/database dinamis Supabase tetap berada di luar cakupan redesign frontend ini dan masuk ke roadmap migrasi backend terpisah (`docs/backend/progress.md`).
- Penghapusan file komponen lama (`TrustStrip.vue`, `KprPartnerStrip.vue`, `WhySection.vue`) dari direktori `components/` belum dilakukan dalam cakupan redesign ini (hanya dibersihkan dari pemakaian di `pages/index.vue`).
