# Progress — Landing Page Sanggar Indah Parahyangan

Acuan status pengerjaan. Detail task per fase ada di `docs/fase-N-*.md`. Update kolom Status saat mulai/selesai fase.

Status: `belum` · `jalan` · `selesai`

| # | Fase | Status | Dokumen |
|---|------|--------|---------|
| 1 | Setup Project (Nuxt 3, Tailwind, struktur folder) | selesai | [fase-1-setup.md](fase-1-setup.md) |
| 2 | Content & Data Layer (`content.js`, `useContent.js`) | selesai | [fase-2-konten.md](fase-2-konten.md) |
| 3 | Komponen UI (layout, primitive, section) | selesai | [fase-3-komponen.md](fase-3-komponen.md) |
| 4 | SEO, Performance & Analytics | belum | [fase-4-seo-analytics.md](fase-4-seo-analytics.md) |
| 5 | Deployment (GitHub Pages + Actions) | belum | [fase-5-deployment.md](fase-5-deployment.md) |
| 6 | QA & Acceptance | belum | [fase-6-qa.md](fase-6-qa.md) |

## Urutan kerja
Fase 1 → 2 → 3 wajib berurutan (komponen butuh data layer, data layer butuh project). Fase 4 & 5 bisa paralel setelah Fase 3 selesai. Fase 6 selalu terakhir.

## Fitur Ekstensi & Spesifikasi (Superpowers Specs)

| Fitur | Status | Dokumen Desain / Spec | Deskripsi Ringkas |
|---|---|---|---|
| Galeri Fasad & Thumbnail Persona | selesai | [2026-08-27-tipe-rumah-galeri-fasad-design.md](superpowers/specs/2026-08-27-tipe-rumah-galeri-fasad-design.md) | Redesign fasad 16:9, swipe gallery di modal tipe rumah |
| Redesign Sanggar Indah Palastri | selesai | [2026-10-30-redesign-palastri.md](superpowers/specs/2026-10-30-redesign-palastri.md) | Transformasi visual Natural Scandinavian × Modern Editorial |
| Section Kalkulator Cicilan KPR | selesai | [2026-11-11-kalkulator-cicilan-design.md](superpowers/specs/2026-11-11-kalkulator-cicilan-design.md) | Simulasi cicilan KPR anuitas 5%, 1 Card Landscape 2 Kolom, default Rp 166 Jt/DP 5%/20 Thn (Rp 1.040.750/bln), pill logo Bank BCA & disclaimer ringkas di luar card, sync DP 2-arah, slider & WA CTA dinamis |
| Panduan Portabilitas Kalkulator KPR | selesai | [guide-kalkulator-cicilan.md](guide-kalkulator-cicilan.md) | Panduan lengkap implementasi kalkulator KPR (Vue 3 / Nuxt 3 / Tailwind) standalone siap salin |
| Redesign Header & Hero Alam Sanggar Indah | selesai | [2026-11-13-redesign-header-hero-asi-design.md](superpowers/specs/2026-11-13-redesign-header-hero-asi-design.md) | Rebranding `logo.webp`, Header adaptif transparan-ke-putih solid saat scroll, Hero 4-layer (`hero-bg.jpg`, teks raksasa, `hero.webp`, panel KPR BCA & BTN), serta perbaikan token `white` di `tailwind.config.js` |

## Referensi
- PRD: `docs/PRD_Landing_Sanggar_Indah_Parahyangan.md`
- Design system: `docs/design/DESIGN.md`
- Migrasi Backend (Supabase, Admin CMS & Blog SEO): [backend/progress.md](backend/progress.md)
- Redesign Sanggar Indah Palastri (Branch `redesign-palastri`): [superpowers/specs/progress-palastri.md](superpowers/specs/progress-palastri.md)
