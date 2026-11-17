# Spec-Lite: Redesign Hero Full-Bleed — Alam Sanggar Indah

**Tanggal:** 2026-11-14 · **Status:** Disetujui User — dikerjakan · **Agent:** `frontend-agent` → `qa-agent` → `doc-agent`

Referensi visual:
- Mockup target (dari User): [`assets/hero-asi-reference.jpg`](assets/hero-asi-reference.jpg)
- Preview komposit aset existing: [`assets/hero-composite-preview.jpg`](assets/hero-composite-preview.jpg)

**Keputusan User (2026-11-14):**
1. Latar hero memakai **komposit CSS dari aset existing** (`hero-bg.jpg` + cutout `hero.webp`), tanpa aset baru.
2. Headline memakai **Poppins SemiBold** (`font-body`), bukan menambah font baru.

---

## 1. Latar Belakang & Tujuan

Hero versi `2026-11-13` memakai komposisi 4-layer: tipografi raksasa di langit, siluet rumah di foreground, headline + CTA split di kiri bawah, dan panel glassmorphism Mitra KPR di kanan bawah. User meminta arah visual baru: **satu foto full-bleed sinematik**, seluruh copy tersusun satu kolom di kiri, dan logo bank tampil polos tanpa panel.

**Tujuan:**
- Hero satu layar penuh (`100dvh`) dengan foto kawasan (langit senja + deretan rumah) sebagai latar tunggal.
- Kolom kiri: badge outline, headline 3 baris, subheadline, CTA putih pill lembut, lalu blok Mitra KPR dengan logo BCA & BTN.
- Menghapus tipografi raksasa di langit, panel glassmorphism, dan CTA split.

---

## 2. Lingkup Perubahan

| File | Perubahan |
|---|---|
| `components/HeroSection.vue` | Ditulis ulang (template + GSAP). |
| `data/content.js` | Tambah `hero.subline`. Field lain tetap (`giantTitle` tidak lagi dipakai di hero, tidak dihapus). |
| `components/TheHeader.vue` | **Tidak diubah** (sudah sesuai mockup: logo + nama, 5 nav, pill `Kontak`). |
| `pages/index.vue`, section lain | **Tidak diubah**. |

---

## 3. Komposisi Visual

### 3.1 Latar (3 layer, tanpa aset baru)

1. **Sky** — `content.hero.bgImage` (`/assets/images/hero-bg.jpg`), `object-cover`, `warm grade` via CSS filter (`saturate/sepia/contrast`) agar senada dengan rumah. Parallax `yPercent: 8`.
2. **Rumah** — `content.hero.image` (`/assets/images/hero.webp`, PNG/WebP **beralfa** = cutout langit transparan). Ditempel `absolute bottom-0`, tinggi otomatis mengikuti lebar, rasio 16:9:
   - Desktop (`lg`): mulai ±`30%` dari kiri, melebar sampai `-6%` di kanan (rumah mengisi 2/3 kanan, terpotong halus di tepi kanan).
   - Tablet/mobile: lebih lebar (`left` ≈ 2–8%), tetap di bawah; area copy cukup gelap.
   - Tepi kiri diberi **mask fade** (`linear-gradient(to right, transparent 0%, #000 16–20%)`) supaya tidak terlihat sebagai potongan gambar.
   - Parallax `y: -26` (terpisah dari wrapper entrance).
3. **Scrim** — top `h-36 from-black/55` (legibilitas header), kiri `w-[72%] from-black/85 via-black/45 to-transparent` (legibilitas copy), bawah `h-[42%] from-black/70` (menyatukan ground line). Opsional: glow hangat `soft-light` tipis di kanan-tengah.

### 3.2 Kolom Copy (kiri, `.container`)

Acuan ukuran dari mockup 1672×941 (CSS px), diterjemahkan ke `clamp()`:

| Elemen | Ukuran |
|---|---|
| Badge | `rounded-full`, border `1px white/45`, `px-5 py-2.5`, teks `10–11px` uppercase `tracking-[0.22em]`, teks = `content.meta.projectName` |
| Headline | `font-body font-semibold uppercase`, `text-[clamp(2.5rem,5.7vw,6rem)]`, `leading-[0.9]`, `tracking-[-0.02em]`, 3 baris dari `content.hero.headlineLines` |
| Subline | `text-[clamp(1rem,1.55vw,1.4rem)]`, `leading-[1.45]`, `text-white/90`, `max-w-[30rem]`, dari `content.hero.subline` (baru) |
| CTA | `rounded-xl` (≈12px, **bukan** pill), `bg-white`, `px-7 py-[18px]`, teks `15–16px font-semibold` warna `#141a1c`, label `content.hero.primaryCta` + ikon panah →, `href="#tipe"`, total tinggi ≈ 62px, hover: `bg-white/95` + panah `translate-x-0.5` |
| Blok KPR | `border-t border-white/25`, `pt-5`, `max-w-[22rem]`, margin atas ≈ 40–48px; label `10–11px uppercase tracking-[0.2em] text-white/85` = `content.hero.bankInfo.label`; logo `content.banks` pakai `imageWhite`, tinggi efektif BCA ≈ 40px & BTN ≈ 36px (disesuaikan visual), dipisah garis vertikal `1px white/30`; `desc` tidak ditampilkan |

Posisi vertikal: kolom copy ± di tengah area di bawah header (`items-center`), jarak atas `pt-[112px]`, bawah `pb-16`; di viewport 941px dimulai ≈ 170–200px dari atas.

Catatan deviasi: mockup memakai padding tepi 100px pada 1672px (≈6vw). Implementasi tetap memakai `.container` (1180px) agar sejajar dengan section lain di halaman.

### 3.3 Tipografi

Mockup memakai grotesque normal-width. Implementasi memakai **`font-body` (Poppins) weight 600** — font yang sudah dimuat situs, paling dekat tanpa menambah aset font. `font-display` (Dream Orphans) tidak dipakai lagi di hero.

---

## 4. Motion (GSAP)

- Entrance bertahap: badge (y 14, delay .1) → headline (y 34, delay .2) → subline (y 24, delay .35) → CTA (y 20, delay .45) → blok KPR (y 20, delay .55); rumah (y 40, opacity 0→1, delay .1). Semua di dalam `gsap.context()` + `ctx.revert()` saat unmount.
- ScrollTrigger parallax (`start: 'top top'`, `end: 'bottom top'`, `scrub: 0.6`): sky `yPercent: 8`, rumah `y: -26`, konten `y: 30`.
- Hormati `prefers-reduced-motion: reduce` (semua animasi dilewati, konten tetap tampil).

---

## 5. Acceptance Criteria (Given/When/Then)

- **AC-1 Komposisi full-bleed:** *Given* Hero dimuat di desktop 1440×900, *Then* foto kawasan memenuhi 100dvh tanpa celah di tepi, rumah tampil di kanan-bawah dengan tepi kiri menyatu (mask), dan tidak ada teks raksasa di langit.
- **AC-2 Kolom kiri lengkap:** *Given* konten termuat, *Then* badge, headline 3 baris, subline, CTA putih (`#tipe`), dan blok Mitra KPR (divider + label + logo BCA & BTN) tampil berurutan dengan spasi sesuai §3.2.
- **AC-3 Keterbacaan:** *Given* foto latar terang (area langit), *Then* seluruh teks terbaca (kontras ≥ 4.5:1 untuk teks kecil, scrim menutup area copy) di desktop maupun mobile.
- **AC-4 Responsif:** *Given* viewport 375×667, 390×844, 768×1024, 1280×800, 1440×900, *Then* tidak ada horizontal scroll, CTA tidak terpotong, headline maksimal 3 baris dengan ukuran mengecil, logo bank tetap sejajar.
- **AC-5 Konten terpusat:** *Given* render, *Then* **nol teks jualan hardcode** — badge/headline/subline/CTA/label KPR berasal dari `useContent()`.
- **AC-6 Header tetap adaptif:** *Given* halaman di-scroll melewati hero, *Then* header berubah putih solid seperti sekarang (tidak regresi), dan saat di atas hero teks header putih tetap kontras.
- **AC-7 Build & runtime bersih:** *Given* `npm run build`, *Then* sukses tanpa error, tidak ada console error / aset 404, animasi hormati reduced-motion.

---

## 6. Out of Scope

- Redesign `TheHeader.vue` (sudah sesuai) dan section di bawah hero (`TrustStrip`, `KprPartnerStrip`, `AboutSection`, dst.).
- Penggantian/penambahan file foto hero baru (jika User menyediakan foto final, cukup ganti `hero.bgImage`/`hero.image` di `data/content.js`).
- Penambahan library/font baru.
