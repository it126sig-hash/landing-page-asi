# Spec-Lite: Redesign Alam Sanggar Indah v2 & Audit UI/UX Pro Max

**Tanggal:** 2026-11-17 · **Status:** `selesai` (Lulus `npx nuxi generate` & Playwright QA pada 4 viewport) · **Agent:** `doc-agent`

---

## 1. Latar Belakang & Tujuan

Setelah implementasi tema *light-blue & crisp white* pada iterasi sebelumnya (`2026-11-17-redesign-landing-page-asi.md`), diperlukan penyempurnaan menyeluruh agar halaman utama **Alam Sanggar Indah** tampil lebih kuat secara visual, bebas dari sisa identitas proyek sebelumnya, lebih fokus pada alur konversi inti, dan memenuhi standar audit `/ui-ux-pro-max`.

**Tujuan:**
- Menghadirkan **Hero full-bleed** (`HeroSection.vue`) bernuansa cerah (`#5BA4E6` → `#9CD2F7` → `#E3F3FD`) dengan gambar deretan rumah membentang penuh kiri-ke-kanan layar tanpa fade, **overlay hitam transparan ringan**, serta **parallax GSAP `ScrollTrigger`** tanpa celah bawah, sehingga tampilan tetap cerah sambil menjaga keterbacaan teks putih dan logo bank mitra KPR.
- Mengadaptasi **`TheHeader.vue`** agar selaras dengan Hero saat di puncak halaman (`scrollY <= 24`: transparan dengan teks putih) dan bertransisi ke putih solid dengan teks `--ink` saat halaman di-scroll.
- Melakukan **rebranding penuh Alam Sanggar Indah** serta menyelaraskan lokasi proyek ke **Citapen, Cihampelas, Kabupaten Bandung Barat** di seluruh copy, metadata, formulir, localStorage key, dan aset visual.
- Memperbarui penawaran unit menjadi **Tipe 36/72** dan **Tipe 31/72** (keduanya berlabel `Komersial` / non-subsidi) dengan **9 spesifikasi teknis identik dari `defaultSpecs`** (`data/content.js` via `composables/useContent.js` & `HouseTypeModal.vue`).
- Menyesuaikan **harga default kalkulator KPR** di `data/content.js` menjadi **`Rp 300.000.000`** (`defaultPrice: 300000000`) agar merepresentasikan kisaran harga unit komersial Alam Sanggar Indah.
- Menambahkan **watermark background `"ALAM SANGGAR INDAH"`** bertumpuk 3 baris per kata (`ALAM` / `SANGGAR` / `INDAH`) di pojok kiri bawah section `#tipe` (`ProductsSection.vue`) dan `#fasilitas` (`FacilitiesSection.vue`) untuk memperkuat identitas visual kawasan.
- **Menghapus permanen `ArticlesSection.vue`** beserta data/navigasi `#artikel` terkait untuk menjaga fokus alur konversi landing page.
- Menyelesaikan temuan **audit `/ui-ux-pro-max`** (konsistensi `cursor-pointer`, hierarki `.eyebrow` + heading *serif italic*, penomoran kartu, kontras tombol & banner, perbaikan alignment teks FAQ, dan ritme latar bergantian antar-section).

---

## 2. Arsitektur & Komponen

### A. Data Layer, Composable & Aset
1. **`data/content.js`:**
   - **Navigasi & Footer:** Menghapus tautan `#artikel` dari `meta.nav` dan `footer.quickLinks`; menautkan `#kalkulator` (`Simulasi KPR`) dan `#faq` (`FAQ`) langsung pada navigasi utama & footer.
   - **Lokasi Citapen:** Memperbarui `locationSection.address`, `mapUrl`, `embedMapUrl`, `footer.mainAddress` ke `Citapen, Cihampelas, Kabupaten Bandung Barat`, serta menyelaraskan `why.items` dan `projects.items`.
   - **Tipe Rumah Non-Subsidi (`houseTypes`):** Menyediakan 2 tipe komersial (`36-72` / `Tipe 36/72` dan `31-72` / `Tipe 31/72`), keduanya dengan `tag: 'Komersial'`, `category: 'Pilihan komersial'`, `specsOverride: []`, serta aset di `/assets/images/rumah/` (`3672.webp`, `3672 denah.webp`, `3175.webp`, `3175 denah.webp`). Opsi `contactCta.options` ikut diperbarui ke `Tipe 36/72 Komersial` dan `Tipe 31/72 Komersial`.
   - **Spesifikasi Identik (`defaultSpecs`):** 9 item spesifikasi standar (`Pondasi`, `Struktur`, `Dinding`, `Lantai`, `Atap`, `Kusen`, `Pintu`, `Listrik: 1300 VA`, `Air`) digunakan seragam untuk kedua tipe.
   - **Kalkulator KPR (`calculator`):** Memperbarui `defaultPrice: 300000000` (Rp 300.000.000) dengan default DP `5%`, tenor `20` tahun, dan suku bunga `5%`.
2. **`composables/useContent.js`:**
   - Memperkuat helper `withBase(path)` dengan `encodeURI(path)` (untuk path gambar yang mengandung spasi seperti `3672 denah.webp`) dan pass-through URL eksternal (`https?://`).
   - Menghapus pemetaan `articlesSection`.
   - Mempertahankan `mergeSpecs(content.defaultSpecs, t.specsOverride)` untuk menghasilkan `specs` identik pada setiap item `houseTypes`.
3. **`composables/useConsultationForm.js` & Aset Publik:**
   - Memperbarui fallback `source` payload menjadi `'landing-page-asi'`.
   - Menambahkan aset tipe rumah di `public/assets/images/rumah/`, memperbarui `public/favicon.ico`, serta menghapus aset lama `public/assets/images/logo-palastri.png` dan `public/assets/images/palastri/rumah/3172-denah.webp`.

### B. Komponen UI yang Berubah & Dihapus
1. **`components/HeroSection.vue`:**
   - Full-bleed viewport (`h-[100dvh] min-h-[600px] sm:min-h-[640px]`) bernuansa langit biru siang yang lebih cerah (`bg-[linear-gradient(180deg,#5BA4E6_0%,#9CD2F7_58%,#E3F3FD_100%)]`) dengan layer langit (`hero-bg.jpg` + tint `#75BDF2` tanpa gradien gelap di bawah), glow matahari lembut kanan-atas, visual deretan rumah (`hero.webp`) membentang penuh kiri-ke-kanan layar (`inset-x-0 -bottom-14 w-full`, `object-cover object-bottom`) **tanpa fade** (`mask-image` dihapus), serta **overlay hitam transparan lebih ringan** (`bg-gradient-to-r from-black/55 via-black/28 to-transparent sm:from-black/50 sm:via-black/25 sm:to-transparent`, top gradient `from-black/30`, bottom gradient `from-black/35`).
   - Kolom kiri berisi badge pill `Alam Sanggar Indah`, headline 3 baris uppercase tebal putih, subline, tombol CTA putih `Jelajahi Pilihan Rumah` (`#tipe`), dan blok Mitra Pembiayaan KPR Resmi (`Bank BCA` & `Bank BTN` menggunakan `imageWhite` tanpa kotak latar).
   - Scroll hint `#tentang` di kanan-bawah (desktop/tablet `md:block`).
   - Animasi entrance timeline & parallax **GSAP + `ScrollTrigger`** (`skyRef` `yPercent: 18`, `houseRef` `y: -52, scale: 1.02, transformOrigin: 'bottom center'` dipadukan dengan posisi `-bottom-14` agar tidak menimbulkan celah bawah saat di-scroll, `contentRef` `y: 56`) dengan guard `prefers-reduced-motion: reduce`.
2. **`components/TheHeader.vue`:**
   - Adaptif terhadap Hero kontras-gelap: saat `scrollY <= 24`, header transparan dengan teks putih (`text-white`), logo dalam wadah `bg-white/95`, tombol CTA putih (`bg-white text-[var(--ink)]`), dan tombol hamburger beraksen putih transparan; saat `scrollY > 24`, bertransisi menjadi `bg-white/95 backdrop-blur-md text-[var(--ink)]` dengan CTA biru (`bg-[var(--green)] text-white`). Dropdown menu mobile (`#mobile-nav-menu`) selalu berlatar putih solid.
3. **`components/ProductsSection.vue` (`#tipe`) & `components/FacilitiesSection.vue` (`#fasilitas`):**
   - Menambahkan **watermark background `"ALAM SANGGAR INDAH"`** yang dipecah per kata dari `content.meta.projectName` menjadi **3 baris bertumpuk (`ALAM` / `SANGGAR` / `INDAH`)** di **pojok kiri bawah** (`bottom-0 left-0`, `pl-4 pb-3 sm:pl-8 sm:pb-5`) dengan tipografi *uppercase* `clamp(2.6rem,7.5vw,5.75rem)`, `leading-[0.88]`, `opacity-[0.055]`, `pointer-events-none select-none`, `aria-hidden="true"` di `z-0`, sementara grid konten berada di `relative z-10`.
   - `ProductsSection.vue`: tombol aksi & kartu produk memakai `.eyebrow`, heading dengan baris kedua `.serif italic`, serta `cursor-pointer`.
   - `FacilitiesSection.vue`: 5 kartu ikon fasilitas horizontal (`Ruang Terbuka Hijau`, `Taman Bermain`, `Keamanan 24 Jam`, `Masjid`, `Fasilitas Olahraga`) dilengkapi nomor urut `01`–`05`.
4. **`components/HouseTypeModal.vue`:**
   - Menghapus `defaultSpecsDefinition` yang sebelumnya ter-hardcode di komponen; `normalizedSpecs` kini membaca langsung dari `props.type.specs` (`content.defaultSpecs` via `useContent()`), memastikan Tipe 36/72 dan Tipe 31/72 menampilkan 9 spesifikasi teknis identik secara konsisten.
5. **Perbaikan Audit `/ui-ux-pro-max` pada Komponen Pendukung & Rebranding:**
   - `components/AboutSection.vue`: menambahkan `.eyebrow`, aksen *serif italic* pada `headlineLines[1]`, badge `Dikembangkan oleh Sanggar Indah Group` di sudut foto, dan penomoran `01`–`04` pada `whyCards`.
   - `components/LocationSection.vue`: menambahkan `.eyebrow` dan dukungan `titleHtml` dengan aksen *serif italic*.
   - `components/FaqSection.vue` & `components/FaqItem.vue`: mengubah latar `#faq` menjadi `bg-[var(--sage-pale)]` agar ritme latar antar-section bergantian rapi, menambahkan `.eyebrow` + *serif italic*, mengubah perataan paragraf dari `text-justify` menjadi `text-left text-pretty`, dan memberi border tegas pada tombol `+`.
   - `components/FinalCtaSection.vue`: memperdalam gradien banner menjadi `from-[var(--green-deep)] via-[var(--green)] to-[#255CA8]` dan mengganti karakter teks `✕` pada tombol tutup modal dengan ikon SVG.
   - `components/Lightbox.vue`: mengganti kelas lama `bg-charcoal/95` menjadi `bg-[var(--ink)]/95` dan menambahkan `cursor-pointer` pada kontrol zoom, tutup, prev/next.
   - `components/KprTermsModal.vue`, `components/MemberGetMemberModal.vue`, `components/ReferralWelcomeModal.vue`, `components/WhatsAppFloatingButton.vue`: menambahkan `cursor-pointer` pada seluruh tombol/tab interaktif serta rebranding `source: 'landing-page-asi'` dan `seenKey` `asi:referral-welcome:...`.
6. **Penghapusan Permanen (`pages/index.vue` & `components/ArticlesSection.vue`):**
   - File `components/ArticlesSection.vue` **dihapus permanen** dari repositori, dan pemanggilan `<ArticlesSection />` dihapus dari `pages/index.vue`. Urutan section final pada `pages/index.vue`: `HeroSection` → `AboutSection` → `ProductsSection` → `LocationSection` → `FacilitiesSection` → `MortgageCalculatorSection` → `FaqSection` → `FinalCtaSection` → `TheFooter`.

---

## 3. Acceptance Criteria & Ringkasan Verifikasi QA

### Acceptance Criteria (Given / When / Then)
- **AC-1 (Hero Full-Bleed, Overlay, Parallax & Header Adaptif):** *Given* pengguna membuka halaman di posisi atas (`scrollY <= 24`), *When* `TheHeader.vue` dan `HeroSection.vue` dirender, *Then* Hero tampil full-bleed `100dvh` berlatar langit cerah (`#5BA4E6` → `#9CD2F7` → `#E3F3FD`) dengan gambar rumah membentang penuh kiri-ke-kanan tanpa fade (`mask-image` dihapus), overlay hitam transparan ringan (`from-black/55 via-black/28`), logo KPR Bank BCA & BTN putih terbaca jelas, animasi parallax GSAP berjalan halus tanpa celah bawah saat di-scroll (nonaktif saat `prefers-reduced-motion: reduce`), serta Header tampil transparan berteks putih lalu bertransisi menjadi putih solid berteks gelap ketika di-scroll (`scrollY > 24`).
- **AC-2 (Rebranding, Tipe Non-Subsidi & Harga Default Kalkulator):** *Given* pengguna menelusuri seluruh halaman dan membuka `HouseTypeModal` maupun kalkulator `#kalkulator`, *When* data dimuat dari `data/content.js` dan `composables/useContent.js`, *Then* seluruh identitas dan lokasi merujuk ke **Alam Sanggar Indah — Citapen, Cihampelas, Kab. Bandung Barat**, hanya **Tipe 36/72** dan **Tipe 31/72** komersial yang tampil dengan 9 spesifikasi teknis identik dari `defaultSpecs`, serta kalkulator menampilkan harga default **`Rp 300.000.000`**.
- **AC-3 (Watermark, Penghapusan ArticlesSection & Audit `/ui-ux-pro-max`):** *Given* pengguna memeriksa `#tipe`, `#fasilitas`, `#faq`, modal, dan elemen interaktif di seluruh viewport, *When* halaman dirender, *Then* watermark `"ALAM SANGGAR INDAH"` tampil 3 baris per kata (`ALAM` / `SANGGAR` / `INDAH`) di pojok kiri bawah (`opacity-[0.055]`) pada `#tipe` dan `#fasilitas`, `ArticlesSection.vue` serta tautan `#artikel` tidak ada lagi, seluruh elemen interaktif memiliki `cursor-pointer`, dan ritme latar antar-section bergantian konsisten tanpa *horizontal overflow*.
- **AC-4 (Build Statis & QA Multi-Viewport):** *Given* seluruh perubahan v2 selesai, *When* dijalankan `npx nuxi generate` dan pengujian Playwright pada 4 viewport, *Then* build statis berhasil tanpa error dan inspeksi multi-viewport dinyatakan **PASS**.

### Ringkasan Hasil Verifikasi QA
- **Build Statis:** `npx nuxi generate` lulus (**PASS**).
- **Playwright QA:** Lulus (**PASS**) pada 4 viewport (`1440x900`, `1280x800`, `768x1024`, `375x667`).
- **Bukti Screenshot (`docs/qa-evidence/redesign-v2/`):**
  - Hero Desktop & Mobile: `hero-1440.png`, `hero-375.png`
  - Section Produk & Fasilitas (Watermark & Audit): `products-facilities-1440.png`, `facilities-1440.png`
  - Modal Tipe Rumah (Spek Identik `defaultSpecs`): `modal-3672.png`, `modal-3175.png`
  - Tangkapan Layar Penuh Desktop & Mobile: `full-1440.png`, `full-375.png`

---

## 4. Out of Scope

- Pembuatan halaman blog/artikel terpisah (`/blog`, `/blog/[slug]`) serta integrasi backend/CMS Supabase berada di luar cakupan redesign frontend v2 ini dan mengikuti roadmap `docs/backend/progress.md`.
- Komponen lama yang sudah tidak dipanggil di `pages/index.vue` (`TrustStrip.vue`, `KprPartnerStrip.vue`, `WhySection.vue`) tidak diubah atau dihapus dalam iterasi ini (hanya `ArticlesSection.vue` yang dihapus permanen).
- Tidak ada perubahan logika perhitungan anuitas pada `MortgageCalculatorSection.vue` di luar penyesuaian nilai `defaultPrice` dari `data/content.js`.
