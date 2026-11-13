# Spec-Lite: Redesign Header & Hero — Alam Sanggar Indah

**Tanggal:** 2026-11-13 · **Status:** Selesai (`PASS`) · **Agent:** `doc-agent`

---

## 1. Latar Belakang & Tujuan

Proyek landing page direbranding dari identitas sebelumnya ke **Alam Sanggar Indah** (`logo.webp`, tagline *"Hunian Asri di Jantung Parahyangan"*). Header dan Hero lama belum menonjolkan karakter arsitektural kawasan maupun identitas merek baru secara kuat di *above-the-fold*.

**Tujuan:**
- Menghadirkan identitas visual **Alam Sanggar Indah** pada navigasi atas dan Hero *full-viewport*.
- Meningkatkan keterbacaan navigasi di atas foto Hero melalui header transparan berbatas garis tipis yang otomatis beralih menjadi putih solid saat pengguna menggulir melewati Hero.
- Menciptakan komposisi Hero berlapis (4-layer) yang memadukan foto latar, tipografi judul raksasa di area langit, siluet bangunan foreground transparan, headline + CTA kiri bawah, serta panel Mitra Pembiayaan KPR Resmi (`BCA` & `BTN`) di kanan bawah.

---

## 2. Arsitektur & Komponen

1. **`components/TheHeader.vue` (Header Adaptif 2-State):**
   - Posisi `fixed top-0`, menggunakan `IntersectionObserver` pada `#beranda` (`rootMargin: '-80px 0px 0px 0px'`, `threshold: 0`, dengan *scroll fallback*).
   - **State di atas Hero (`!isScrolled`):** Latar transparan gelap tipis dengan blur (`bg-black/15 backdrop-blur-[4px]`), teks putih, dan border bawah `border-white/25`.
   - **State melewati Hero (`isScrolled`):** Latar putih solid (`bg-white`), teks `#111111`, border bawah `border-black/10`, dan `shadow-sm`.
   - **Layout Desktop (`lg:grid-cols-[1fr_auto_1fr]`):** Kiri logo `logo.webp` + nama proyek; tengah 5 tautan navigasi; kanan tombol pill CTA `Kontak` (`wa('konsultasi')`) dan tombol hamburger mobile (`#mobile-nav-menu` berlatar putih solid, mendukung tutup via tombol `Escape`).
2. **`components/HeroSection.vue` (Hero 4-Layer + GSAP Entrance & Parallax, `#beranda`, `min-h-[100dvh]`):**
   - **Layer 1 (`z-0` & `z-[5]`):** Wrapper `bgParallaxRef` (`-top-[16%] h-[118%]` sebagai bleed parallax) memuat foto latar `hero-bg.jpg` + *top/bottom gradient scrim*.
   - **Layer 2 (`z-10`):** Wrapper `titleParallaxRef` + inner `titleInnerRef` memuat tipografi raksasa `ALAM SANGGAR INDAH` (`content.hero.giantTitle`, `font-display`), skala responsif `text-[clamp(3rem,13vw,5.25rem)]` (2 baris di mobile) dan `md:text-[clamp(3.2rem,9.2vw,9.25rem)] md:whitespace-nowrap` (1 baris di desktop).
   - **Layer 3 (`z-20` & `z-[25]`):** Wrapper `houseParallaxRef` dengan bottom bleed `-bottom-16` (`h-[calc(68dvh+4rem)]` s/d `lg:h-[calc(90dvh+4rem)]`) + inner `houseInnerRef` memuat `hero.webp` (atap rumah menumpuk di depan teks raksasa) + *bottom gradient scrim* (`from-black/80 via-black/40`).
   - **Layer 4 (`z-30`):** Overlay UI bawah (kiri atas tetap bersih): **kiri bawah** (`uiCopyRef`) memuat `headlineLines` (*"HUNIAN ASRI / DI JANTUNG / PARAHYANGAN"*) + CTA split ke `#tipe` (`Jelajahi Pilihan Rumah »`); **kanan bawah** (`uiBankRef`) memuat panel glassmorphism Mitra Pembiayaan KPR Resmi (`content.hero.bankInfo`) dengan logo putih `BCA` & `BTN` (`bank.imageWhite`).
   - **Animasi GSAP & ScrollTrigger** (dalam `gsap.context()` dengan `ctx?.revert()` saat unmount, dilewati jika `prefers-reduced-motion: reduce)` aktif):
     - *Entrance:* `titleInnerRef` (`y: 44`, `duration: 1.0`, `delay: 0.2`), `houseInnerRef` (`y: 80`, `scale: 1.04`, `duration: 1.15`, `delay: 0.35`), dan `uiCopyRef` + `uiBankRef` (`y: 28`, `stagger: 0.12`, `delay: 0.55`).
     - *ScrollTrigger Parallax* (`start: 'top top'`, `end: 'bottom top'`, `scrub: 0.6` pada wrapper luar terpisah): `bgParallaxRef` (`yPercent: 12`), `titleParallaxRef` (`y: 165` turun masuk ke belakang atap), dan `houseParallaxRef` (`y: -48` naik halus di foreground).
3. **Data Layer & Konfigurasi:**
   - `data/content.js`: Terpusat pada `meta` (`projectName`, `logo`, `tagline`), `hero` (`bgImage`, `image`, `giantTitle`, `headlineLines`, `primaryCta`, `bankInfo`), dan `banks` (`imageWhite` untuk BCA & BTN).
   - `composables/useContent.js`: Menormalisasi seluruh path aset lewat `withBase()` dan menyusun tautan WhatsApp lewat `wa()`.
   - `pages/index.vue`: Menyelaraskan deskripsi SEO dan fallback `ogImage` ke `content.hero.bgImage` (`/assets/images/hero-bg.jpg`).
   - `tailwind.config.js`: Menghapus override `white: 'var(--white)'` di `theme.extend.colors` agar utilitas `white` kembali menggunakan warna bawaan `#ffffff` dan mendukung modifier opasitas (`bg-white/90`, `border-white/25`, `text-white/95`, dll.).

---

## 3. Acceptance Criteria (Given / When / Then)

- **AC-1 (Header Awal & Transisi Scroll):** *Given* pengguna membuka halaman di posisi atas (`#beranda`), *When* Hero berada di viewport, *Then* header tampil transparan berbatas `border-white/25` dengan teks/logo putih; *When* halaman digulir melewati batas Hero (`-80px`), *Then* header berubah menjadi putih solid (`bg-white`) dengan teks gelap (`#111111`) dan tombol CTA hijau.
- **AC-2 (Navigasi Mobile):** *Given* viewport mobile/tablet (`< lg`), *When* tombol hamburger ditekan, *Then* `#mobile-nav-menu` terbuka dengan latar putih solid, dan tertutup saat tautan diklik atau tombol `Escape` ditekan.
- **AC-3 (Komposisi Hero 4-Layer & Skala Tipografi):** *Given* section `#beranda` dimuat, *When* dirender di mobile maupun desktop (`>= md`), *Then* `ALAM SANGGAR INDAH` tampil 2 baris di mobile (`clamp(3rem,13vw,5.25rem)`) dan 1 baris di desktop (`clamp(3.2rem,9.2vw,9.25rem)`, `whitespace-nowrap`), tersusun bersama `hero-bg.jpg`, `hero.webp`, headline 3 baris + CTA ke `#tipe`, serta panel Mitra KPR `BCA` & `BTN` secara proporsional.
- **AC-4 (GSAP Entrance, Parallax & Reduced Motion):** *Given* preferensi gerak standar, *When* Hero dimuat dan digulir, *Then* animasi entrance berjalan bertahap dan parallax (`scrub: 0.6`) menggerakkan latar (`yPercent: 12`), judul (`y: 165` masuk ke belakang atap), dan rumah (`y: -48`) tanpa celah di tepi bawah (`-bottom-16`); *When* `prefers-reduced-motion: reduce` aktif, *Then* seluruh animasi GSAP dilewati.
- **AC-5 (Token Opasitas Putih):** *Given* class utilitas `white` dengan modifier opasitas (mis. `border-white/25`, `text-white/80`, `bg-white/90`), *When* CSS dikompilasi Tailwind, *Then* seluruh modifier opasitas terender valid.

---

## 4. Out of Scope

- Redesign visual maupun pembaruan copywriting/aset pada section di bawah Hero (`TrustStrip`, `KprPartnerStrip`, `AboutSection`, `ProductsSection`, `MortgageCalculatorSection`, `WhySection`, `LocationSection`, `FaqSection`, `FinalCtaSection`, dan `TheFooter`) belum dilakukan pada tahap ini.
- Penambahan atau perubahan rute halaman di luar `pages/index.vue`.
