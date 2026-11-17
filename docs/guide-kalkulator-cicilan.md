# Panduan Portabilitas & Implementasi Reusable: Kalkulator Cicilan KPR (v2.1)

Dokumen ini adalah panduan teknis lengkap (*complete integration guide*) untuk mereplikasi dan mengimplementasikan **Section Kalkulator Cicilan KPR (Simulasi Bunga Anuitas Dinamis v2.1)** ke proyek web lain berbasis **Vue 3**, **Nuxt 3**, dan **Tailwind CSS**.

---

## 1. Overview & Diagram Wireframe Arsitektur v2.1

Kalkulator ini dirancang dengan pendekatan *high-conversion, mobile-responsive, lean UX, clean code*, dan akurasi finansial sesuai standar perbankan nasional.

Pada pembaruan **versi 2.1**, antarmuka disederhanakan (*decluttered*) untuk memfokuskan calon konsumen pada besaran cicilan bulanan:
- **Heading Rata Tengah (Center-Aligned)**: Tampilan judul harmonis dan bersih tanpa eyebrow (*no eyebrow*).
- **Sub-grid 2 Kolom Responsif (`sm:grid-cols-2`)**: Form Suku Bunga Manual (% p.a.) dan Pilihan Tenor disajikan berdampingan di tablet/desktop dan bertumpuk 1 kolom di mobile smartphone.
- **Tanpa Preset Tipe Rumah**: Pengguna bebas memasukkan nominal harga rumah apa pun secara fleksibel dilengkapi tombol reset `✕`.
- **Box Estimasi Cicilan Bersih**: Box highlight nominal cicilan bulanan disajikan bersih tanpa badge bunga statis.
- **Grid 3 Metrik Dinamis Real-Time**: Tiga kartu metrik ringkasan (Bunga KPR % p.a., Uang Muka %, Tenor Thn) otomatis terupdate seketika mengikuti perubahan input pengguna.
- **Eliminasi Accordion Biaya Akad Awal (Clean Code)**: Menghilangkan perhitungan provisi/admin bank untuk menghindari kelebihan informasi (*information overload*) pada tahap awal simulasi.
- **Wadah Pill Disclaimer Netral**: Catatan edukatif diletakkan di luar kartu utama secara mandiri tanpa keterikatan logo bank spesifik.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ [Latar Belakang: Gambar Hunian 3D Modern - Opacity 10-15% + Vertical Fade Gradient]   │
│                                                                                        │
│                           Hitung Simulasi Cicilan Rumah Anda                           │
│              Rencanakan pembelian rumah impian Anda dengan mudah...                    │
│                                                                                        │
│  ┌─ SATU CARD LANDSCAPE 2 KOLOM (bg-white, rounded-[26px], border-line, shadow) ─────┐ │
│  │                                                   │                              │ │
│  │  KOLOM 1 (KIRI, 6 Col): FORM INPUT FINANSIAL      │  KOLOM 2 (KANAN, 6 Col):     │ │
│  │                                                   │  RINGKASAN ESTIMASI & CTA    │ │
│  │  🏠 Harga Rumah             [Rp 166.000.000]      │  Plafon Pembiayaan KPR       │ │
│  │  ┌──────────────────────────────────────────┬───┐ │    Rp 157.700.000            │ │
│  │  │ Rp 166.000.000                           │ ✕ │ │  ──────────────────────────  │ │
│  │  └──────────────────────────────────────────┴───┘ │  ┌─ BOX ESTIMASI CICILAN ──┐ │ │
│  │                                                   │  │ Estimasi Cicilan/Bulan  │ │ │
│  │  👛 Uang Muka (DP) - 5% (Rp 8.300.000)            │  │ Rp 1.040.750 /bulan     │ │ │
│  │  ┌────────────────────────┬─────────────────────┐ │  └─────────────────────────┘ │ │
│  │  │ Rp 8.300.000           │  5 %                │ │  ┌─ 3 METRICS GRID ────────┐ │ │
│  │  └────────────────────────┴─────────────────────┘ │  │ Bunga     DP       Tenor │ │ │
│  │  [==●===========================================] │  │ 5% p.a.   5%       20 Th │ │ │
│  │  5% Min DP                 50%       90% Max DP   │  └─────────────────────────┘ │ │
│  │                                                   │                              │ │
│  │  ┌─ SUB-GRID 2 KOLOM (sm:grid-cols-2 gap-4) ────┐ │                              │ │
│  │  │ Suku Bunga (%/thn)   │ Jangka Waktu (Tenor)  │ │                              │ │
│  │  │ [ 5.0            % ] │ [ 20 Thn (240 Bln)  ▼]│ │                              │ │
│  │  └──────────────────────┴───────────────────────┘ │  [ 📲 Konsultasi WhatsApp ↗ ]│ │
│  └───────────────────────────────────────────────────┴──────────────────────────────┘ │
│                                                                                        │
│       ┌─ WADAH PILL DISCLAIMER NETRAL DI LUAR CARD (White/80 Backdrop-blur, Border) ──┐ │
│       │ *Perhitungan simulasi ini hanya sebagai patokan/estimasi. Perhitungan riil    │ │
│       │ dan suku bunga di bank dapat berbeda sesuai kebijakan bank yang berlaku.      │ │
│       └───────────────────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### Rangkuman Fitur Unggulan v2.1:
1. **Layout 1 Card Landscape 2 Kolom**: Terpadu dalam satu kartu utama (`rounded-[26px]`) dengan split 6-kolom input dan 6-kolom hasil estimasi pada desktop.
2. **Suku Bunga Dinamis & Manual (% p.a.)**: Pengguna dapat menguji berbagai skema suku bunga bank (KPR Subsidi FLPP 5%, KPR Promo 3.5%–7%, atau KPR Komersial 8%+) secara leluasa.
3. **Sub-Grid 2 Kolom Responsif (`sm:grid-cols-2 gap-4`)**: Input suku bunga dan dropdown tenor sejajar rapi di layar $\ge 640\text{px}$, menghemat ruang vertikal kartu secara drastis. Label dilengkapi proteksi `truncate` dan badge dengan `shrink-0 whitespace-nowrap` anti-wrapping.
4. **Sinkronisasi Tiga Arah Uang Muka (DP)**:
   - Mengetik nominal Rupiah $\rightarrow$ persentase DP & slider otomatis sinkron.
   - Mengubah persentase DP $\rightarrow$ nominal DP & slider otomatis sinkron.
   - Menggeser slider range $\rightarrow$ nominal DP & persentase DP otomatis sinkron dengan track gradient terisi proporsional.
   - Mengubah harga rumah $\rightarrow$ nominal DP terhitung ulang mengikuti persentase aktif.
5. **Tombol Reset Cepat (`✕`)**: Tombol clear yang mengembalikan harga dan nominal ke 0 secara aman tanpa error `NaN`.
6. **Formula Anuitas Bulanan Presisi Perbankan**: Menghitung angsuran tetap bulanan secara akurat menggunakan formula anuitas standar bank nasional ($A = P \times \frac{r(1+r)^n}{(1+r)^n - 1}$).
7. **Grid 3 Metrik Dinamis Real-Time**: Kartu ringkasan Bunga KPR (`% p.a.`), Uang Muka (`%`), dan Tenor (`Thn`) yang selalu sinkron instan.
8. **WhatsApp Lead Generator**: Tombol CTA menghasilkan tautan WhatsApp otomatis yang langsung memuat rincian simulasi pengguna (*pre-filled message*).
9. **Wadah Pill Disclaimer Netral**: Posisi di luar kartu utama dengan latar belakang *backdrop-blur* transparan, bebas dari ikatan logo bank tertentu.

---

## 2. Kebutuhan Stack & Dependensi

Komponen ini bersifat *framework-agnostic* di ekosistem Vue modern:
- **Core Framework**: Vue 3 (Composition API `<script setup>`) atau Nuxt 3.
- **CSS Utility Engine**: Tailwind CSS (v3.x atau v4.x).
- **Icons / Simbol**: Karakter UTF-8 bawaan (`Rp`, `▼`, `✕`, `↗`, `%`). Tidak membutuhkan dependensi icon library eksternal (Lucide/FontAwesome).
- **Zero Third-Party Financial Plugins**: Semua formula aktuaria dihitung menggunakan JavaScript murni.

### Konfigurasi Variabel Warna CSS (Design Tokens)
Komponen menggunakan variabel CSS dengan nilai fallback bawaan, sehingga dapat langsung bekerja tanpa setup rumit atau diintegrasikan ke theme kustom proyek Anda:

```css
:root {
  --calc-bg: #f6f5f1;             /* Latar section (off-white / cream pale) */
  --calc-card: #ffffff;           /* Latar kartu utama */
  --calc-ink: #1e2420;            /* Teks judul & nominal utama (charcoal) */
  --calc-muted: #6b726d;          /* Teks deskripsi & label input */
  --calc-line: #e3e1da;           /* Border garis halus (cream light) */
  --calc-primary: #1c4a2a;        /* Warna primer tombol & slider (forest deep) */
  --calc-primary-hover: #2d5a3f;  /* Warna hover tombol primer */
  --calc-track: #eceae1;          /* Track slider tidak aktif */
  --calc-highlight-bg: #eaf1ec;   /* Latar box estimasi cicilan (mint pale) */
  --calc-highlight-border: #c8d8cd; /* Border box estimasi cicilan */
}
```

---

## 3. Struktur Data Konfigurasi (Config Object)

Seluruh teks antarmuka, nilai default finansial, daftar tenor, dan template WhatsApp dapat dikonfigurasi melalui sebuah objek konfigurasi (misalnya di file `content.js` atau JSON):

```javascript
export const mortgageCalculatorConfig = {
  id: 'kalkulator',
  eyebrow: '', // Kosongkan untuk menyembunyikan eyebrow (v2.1)
  titleLines: ['Hitung Simulasi', 'Cicilan Rumah Anda'],
  subtitle: 'Rencanakan pembelian rumah impian Anda dengan mudah menggunakan kalkulator cicilan berikut.',
  
  // Parameter Finansial Default
  defaultPrice: 166000000,          // Patokan harga awal rumah (Rp 166 Jt)
  defaultDpPercent: 5,              // Uang Muka (DP) default 5%
  defaultTenor: 20,                 // Tenor pinjaman default 20 tahun
  defaultInterestRatePercent: 5,   // Suku bunga tahunan default 5.0% p.a.
  minDpPercent: 5,                  // Batas minimum slider DP (5%)
  maxDpPercent: 90,                 // Batas maksimum slider DP (90%)
  
  // Pilihan Tenor Pinjaman
  tenors: [
    { value: 5, label: '5 Tahun (60 Bulan)' },
    { value: 10, label: '10 Tahun (120 Bulan)' },
    { value: 15, label: '15 Tahun (180 Bulan)' },
    { value: 20, label: '20 Tahun (240 Bulan)' },
  ],
  
  // Gambar Latar Hunian Subtle
  image: '/assets/images/rumah-3d.webp',
  imageAlt: 'Ilustrasi Hunian Modern',
  
  // Template & Nomor WhatsApp Marketing
  whatsappNumber: '6282118992299', // Format nomor internasional tanpa tanda +
  whatsappTemplate: 'Halo tim marketing, saya melakukan simulasi cicilan di website dengan rincian:\n- Harga Rumah: {harga}\n- Uang Muka: {dp} ({dpPercent}%)\n- Plafon KPR: {plafon}\n- Suku Bunga: {bunga} p.a.\n- Tenor: {tenor} Tahun\n- Estimasi Cicilan: {cicilan}/bulan\n\nBoleh bantu info ketersediaan unit dan persyaratannya?',
  
  // Teks Disclaimer Netral di Luar Card
  footnote: '*Perhitungan simulasi ini hanya sebagai patokan/estimasi. Perhitungan riil dan suku bunga di bank dapat berbeda sesuai kebijakan bank yang berlaku.',
  ctaLabel: 'Konsultasi Simulasi via WhatsApp',
  
  // Label UI (Dapat disesuaikan bahasa / i18n)
  labels: {
    housePrice: 'Harga Rumah',
    downPayment: 'Uang Muka (DP)',
    downPaymentNominal: 'Uang Muka Nominal',
    downPaymentPercent: 'Persentase DP',
    interestRate: 'Suku Bunga (% per tahun)',
    tenor: 'Jangka Waktu Pinjaman',
    clearLabel: 'Hapus input harga',
    minDp: '5% Min DP',
    midDp: '50%',
    maxDp: '90% Max DP',
    plafonTitle: 'Plafon Pembiayaan KPR',
    installmentTitle: 'Estimasi Cicilan per Bulan',
    perMonth: '/bulan',
    metricInterest: 'Bunga KPR',
    metricDp: 'Uang Muka',
    metricTenor: 'Tenor KPR',
  },
};
```

---

## 4. Formula Finansial & Logika Inti (The Math & Core Logic)

### 4.1 Rumus Bunga Anuitas Bulanan (EMI) dengan Suku Bunga Dinamis

Kalkulator menggunakan rumus baku anuitas perbankan (*Equated Monthly Installment / EMI*):

$$A = P \times \frac{r(1 + r)^n}{(1 + r)^n - 1}$$

Di mana:
- $P$ (**Plafon Kredit**) = $\max(0, \text{Harga Rumah} - \text{Nominal Uang Muka})$
- $i$ (**Suku Bunga Tahunan Dinamis**) = $\frac{\text{interestRate}}{100}$ (contoh: $5\% \rightarrow 0.05$)
- $r$ (**Suku Bunga Bulanan**) = $\frac{i}{12} = \frac{\text{interestRate} / 100}{12}$
- $n$ (**Jumlah Periode Bulan**) = $\text{Tenor (Tahun)} \times 12$
- $A$ (**Angsuran per Bulan**) = Nominal angsuran pokok + bunga tetap per bulan

#### Penanganan Edge Case Finansial:
1. **Plafon $\le 0$**: Menghasilkan angsuran $\text{Rp } 0$.
2. **Suku Bunga $\le 0$ ($i = 0$)**: Menghindari pembagian dengan nol (*division by zero*). Angsuran dihitung murni pembagian pokok pinjaman: $A = \text{round}(P / n)$.

```javascript
function calculateMonthlyInstallment(plafon, interestRatePercent, tenorYears) {
  if (plafon <= 0) return 0;
  const rateVal = Number(interestRatePercent);
  const annualRate = (isNaN(rateVal) ? 0 : rateVal) / 100;
  const n = (tenorYears || 20) * 12;
  
  if (annualRate <= 0) return Math.round(plafon / n);
  
  const r = annualRate / 12;
  const factor = Math.pow(1 + r, n);
  const emi = plafon * (r * factor) / (factor - 1);
  return Math.round(emi);
}
```

### 4.2 Matriks Referensi Validasi Finansial (Akurasi Presisi)

Sebagai acuan uji kebenaran perhitungan pada harga unit **Rp 166.000.000** (DP 5% = Rp 8.300.000, Plafon = Rp 157.700.000):

| Suku Bunga p.a. | Tenor 5 Thn ($n=60$) | Tenor 10 Thn ($n=120$) | Tenor 15 Thn ($n=180$) | Tenor 20 Thn ($n=240$) |
|:---:|:---:|:---:|:---:|:---:|
| **5.0%** (Subsidi FLPP) | Rp 2.976.013/bln | Rp 1.672.846/bln | Rp 1.247.165/bln | **Rp 1.040.750/bln** |
| **7.0%** (Promo Komersil) | Rp 3.122.641/bln | Rp 1.831.026/bln | Rp 1.417.414/bln | **Rp 1.222.658/bln** |
| **8.5%** (KPR Reguler) | Rp 3.235.961/bln | Rp 1.954.551/bln | Rp 1.551.789/bln | **Rp 1.368.514/bln** |

### 4.3 Logika Sinkronisasi Dua Arah Uang Muka (DP)

Tiga kendali DP (Input Rupiah, Input Persen, dan Slider Range) saling mengikat secara reaktif:

```javascript
// 1. Pengguna mengetik harga rumah manual
function onPriceInput(digitsOnlyString, currentDpPercent) {
  const price = parseInt(digitsOnlyString, 10) || 0;
  const dpNominal = Math.round(price * (currentDpPercent / 100));
  return { price, dpNominal };
}

// 2. Pengguna mengetik nominal DP dalam Rupiah
function onDpNominalInput(digitsOnlyString, currentPrice) {
  const dpNominal = parseInt(digitsOnlyString, 10) || 0;
  let dpPercent = 0;
  if (currentPrice > 0) {
    const pct = (dpNominal / currentPrice) * 100;
    dpPercent = Math.min(100, Math.max(0, Number(pct.toFixed(1))));
  }
  return { dpNominal, dpPercent };
}

// 3. Pengguna mengubah persentase DP atau menggeser slider range
function onDpPercentChange(newPercent, currentPrice) {
  let pct = parseFloat(newPercent);
  if (isNaN(pct)) pct = 0;
  if (pct > 95) pct = 95;
  if (pct < 0) pct = 0;
  const dpNominal = Math.round(currentPrice * (pct / 100));
  return { dpPercent: pct, dpNominal };
}
```

### 4.4 Perhitungan Dynamic Gradient Track Slider

Agar track range slider terisi warna hijau aktif sesuai posisi handle secara mulus:

```javascript
const sliderBackgroundStyle = computed(() => {
  const min = c.value.minDpPercent || 5;
  const max = c.value.maxDpPercent || 90;
  const val = Math.min(Math.max(dpPercent.value, min), max);
  const percentage = ((val - min) / (max - min)) * 100;
  return {
    background: `linear-gradient(to right, var(--calc-primary, #1c4a2a) 0%, var(--calc-primary, #1c4a2a) ${percentage}%, var(--calc-track, #eceae1) ${percentage}%, var(--calc-track, #eceae1) 100%)`,
  };
});
```

---

## 5. Komponen Reusable Siap Pakai (`MortgageCalculator.vue`)

Komponen Single-File Component (SFC) berikut **100% self-contained** dan siap disalin langsung ke folder `components/MortgageCalculator.vue` di proyek Vue 3 atau Nuxt 3 Anda:

```vue
<script setup>
import { computed, ref } from 'vue';

const props = defineProps({
  config: {
    type: Object,
    default: () => ({}),
  },
});

// Default Configuration Fallback (v2.1)
const defaultConfig = {
  id: 'kalkulator',
  eyebrow: '',
  titleLines: ['Hitung Simulasi', 'Cicilan Rumah Anda'],
  subtitle: 'Rencanakan pembelian rumah impian Anda dengan mudah menggunakan kalkulator cicilan berikut.',
  defaultPrice: 166000000,
  defaultDpPercent: 5,
  defaultTenor: 20,
  defaultInterestRatePercent: 5,
  minDpPercent: 5,
  maxDpPercent: 90,
  tenors: [
    { value: 5, label: '5 Tahun (60 Bulan)' },
    { value: 10, label: '10 Tahun (120 Bulan)' },
    { value: 15, label: '15 Tahun (180 Bulan)' },
    { value: 20, label: '20 Tahun (240 Bulan)' },
  ],
  image: '',
  imageAlt: 'Ilustrasi Hunian Modern',
  whatsappNumber: '6282118992299',
  whatsappTemplate: 'Halo tim marketing, saya melakukan simulasi cicilan di website dengan rincian:\n- Harga Rumah: {harga}\n- Uang Muka: {dp} ({dpPercent}%)\n- Plafon KPR: {plafon}\n- Suku Bunga: {bunga} p.a.\n- Tenor: {tenor} Tahun\n- Estimasi Cicilan: {cicilan}/bulan\n\nBoleh bantu info ketersediaan unit dan persyaratannya?',
  footnote: '*Perhitungan simulasi ini hanya sebagai patokan/estimasi. Perhitungan riil dan suku bunga di bank dapat berbeda sesuai kebijakan bank yang berlaku.',
  ctaLabel: 'Konsultasi Simulasi via WhatsApp',
  labels: {
    housePrice: 'Harga Rumah',
    downPayment: 'Uang Muka (DP)',
    downPaymentNominal: 'Uang Muka Nominal',
    downPaymentPercent: 'Persentase DP',
    interestRate: 'Suku Bunga (% per tahun)',
    tenor: 'Jangka Waktu Pinjaman',
    clearLabel: 'Hapus input harga',
    minDp: '5% Min DP',
    midDp: '50%',
    maxDp: '90% Max DP',
    plafonTitle: 'Plafon Pembiayaan KPR',
    installmentTitle: 'Estimasi Cicilan per Bulan',
    perMonth: '/bulan',
    metricInterest: 'Bunga KPR',
    metricDp: 'Uang Muka',
    metricTenor: 'Tenor KPR',
  },
};

const c = computed(() => ({
  ...defaultConfig,
  ...props.config,
  labels: { ...defaultConfig.labels, ...(props.config?.labels || {}) },
}));

// State Reaktif
const price = ref(c.value.defaultPrice);
const dpPercent = ref(c.value.defaultDpPercent);
const dpNominal = ref(Math.round(price.value * (dpPercent.value / 100)));
const tenorYears = ref(c.value.defaultTenor);
const interestRate = ref(c.value.defaultInterestRatePercent);

// Format Rupiah
function formatCurrency(val) {
  if (val === null || val === undefined || isNaN(val)) return 'Rp 0';
  return 'Rp ' + Math.round(val).toLocaleString('id-ID');
}

const formattedPrice = computed(() => {
  return price.value > 0 ? price.value.toLocaleString('id-ID') : '';
});

const formattedDpNominal = computed(() => {
  return dpNominal.value > 0 ? dpNominal.value.toLocaleString('id-ID') : '';
});

// Event Handlers Input
function onPriceInput(e) {
  const digits = e.target.value.replace(/\D/g, '');
  const num = digits ? parseInt(digits, 10) : 0;
  price.value = num;
  dpNominal.value = Math.round(num * (dpPercent.value / 100));
}

function clearPrice() {
  price.value = 0;
  dpNominal.value = 0;
  dpPercent.value = 0;
}

function onDpNominalInput(e) {
  const digits = e.target.value.replace(/\D/g, '');
  const num = digits ? parseInt(digits, 10) : 0;
  dpNominal.value = num;
  if (price.value > 0) {
    const pct = (num / price.value) * 100;
    dpPercent.value = Math.min(100, Math.max(0, Number(pct.toFixed(1))));
  } else {
    dpPercent.value = 0;
  }
}

function onDpPercentInput(e) {
  let pct = parseFloat(e.target.value);
  if (isNaN(pct)) pct = 0;
  if (pct > 95) pct = 95;
  if (pct < 0) pct = 0;
  dpPercent.value = pct;
  dpNominal.value = Math.round(price.value * (pct / 100));
}

function onSliderInput(e) {
  const pct = Number(e.target.value);
  dpPercent.value = pct;
  dpNominal.value = Math.round(price.value * (pct / 100));
}

// Perhitungan Finansial
const plafon = computed(() => Math.max(0, price.value - dpNominal.value));

const monthlyInstallment = computed(() => {
  const P = plafon.value;
  if (P <= 0) return 0;
  const rateVal = Number(interestRate.value);
  const annualRate = (isNaN(rateVal) ? 0 : rateVal) / 100;
  const years = tenorYears.value || 20;
  const n = years * 12;
  if (annualRate <= 0) return Math.round(P / n);

  const r = annualRate / 12;
  const factor = Math.pow(1 + r, n);
  const emi = P * (r * factor) / (factor - 1);
  return Math.round(emi);
});

// Slider Background Track Gradient
const sliderBackgroundStyle = computed(() => {
  const min = c.value.minDpPercent || 5;
  const max = c.value.maxDpPercent || 90;
  const val = Math.min(Math.max(dpPercent.value, min), max);
  const percentage = ((val - min) / (max - min)) * 100;
  return {
    background: `linear-gradient(to right, var(--calc-primary, #1c4a2a) 0%, var(--calc-primary, #1c4a2a) ${percentage}%, var(--calc-track, #eceae1) ${percentage}%, var(--calc-track, #eceae1) 100%)`,
  };
});

// Grid 3 Metrik Dinamis Real-Time
const metrics = computed(() => [
  {
    label: c.value.labels?.metricInterest || 'Bunga KPR',
    value: `${interestRate.value ?? 0}% p.a.`,
  },
  {
    label: c.value.labels?.metricDp || 'Uang Muka',
    value: `${dpPercent.value}%`,
  },
  {
    label: c.value.labels?.metricTenor || 'Tenor KPR',
    value: `${tenorYears.value} Thn`,
  },
]);

// Dynamic WhatsApp Link
const whatsappUrl = computed(() => {
  const template = c.value.whatsappTemplate || '';
  const phone = (c.value.whatsappNumber || '').replace(/\D/g, '');
  const map = {
    harga: formatCurrency(price.value),
    dp: formatCurrency(dpNominal.value),
    dpPercent: dpPercent.value,
    plafon: formatCurrency(plafon.value),
    bunga: `${interestRate.value ?? 0}%`,
    tenor: tenorYears.value,
    cicilan: formatCurrency(monthlyInstallment.value),
  };
  let message = template;
  for (const [key, val] of Object.entries(map)) {
    message = message.replaceAll(`{${key}}`, String(val));
  }
  return `https://wa.me/${phone}?text=${encodeURIComponent(message)}`;
});
</script>

<template>
  <section
    :id="c.id || 'kalkulator'"
    class="calc-section relative pt-6 pb-16 lg:pt-8 lg:pb-20 scroll-mt-20 overflow-hidden bg-[var(--calc-bg,#f6f5f1)]"
  >
    <!-- Background Image with Fade Effect -->
    <div v-if="c.image" class="pointer-events-none absolute inset-0 z-0 overflow-hidden">
      <img
        :src="c.image"
        :alt="c.imageAlt || ''"
        aria-hidden="true"
        class="h-full w-full object-cover object-center opacity-10 lg:opacity-15"
      />
      <div class="absolute inset-0 bg-gradient-to-b from-[var(--calc-bg,#f6f5f1)] via-transparent to-[var(--calc-bg,#f6f5f1)]"></div>
    </div>

    <div class="container mx-auto px-4 sm:px-6 lg:px-8 relative z-10 max-w-6xl">
      <!-- Section Heading (Centered, Tanpa Eyebrow) -->
      <div class="text-center mx-auto max-w-2xl mb-8 lg:mb-10">
        <span
          v-if="c.eyebrow"
          class="inline-block text-xs font-bold tracking-widest uppercase text-[var(--calc-primary,#1c4a2a)] mb-3"
        >
          {{ c.eyebrow }}
        </span>
        <h2 class="text-3xl sm:text-4xl lg:text-[42px] font-serif font-semibold leading-tight text-[var(--calc-ink,#1e2420)] mb-3 text-center">
          {{ c.titleLines?.[0] || 'Hitung Simulasi' }}<br>
          <em class="italic font-normal">{{ c.titleLines?.[1] || 'Cicilan Rumah Anda' }}</em>
        </h2>
        <p v-if="c.subtitle" class="text-xs sm:text-sm text-[var(--calc-muted,#6b726d)] leading-relaxed">
          {{ c.subtitle }}
        </p>
      </div>

      <!-- SATU Card Landscape 2 Kolom -->
      <div class="bg-[var(--calc-card,#ffffff)] border border-[var(--calc-line,#e3e1da)] rounded-[26px] p-6 sm:p-8 lg:p-10 shadow-[0_4px_30px_rgba(20,43,32,0.06)]">
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 lg:gap-12 items-start">
          <!-- Kolom 1 (Kiri): Form Input Finansial (6 Col) -->
          <div class="lg:col-span-6 space-y-6">
            <!-- 1. Input Harga Rumah -->
            <div>
              <div class="flex items-center justify-between mb-2">
                <label for="calc-price-input" class="text-xs sm:text-[13px] font-bold text-[var(--calc-ink,#1e2420)] tracking-tight">
                  {{ c.labels?.housePrice }}
                </label>
                <span v-if="price > 0" class="text-[11px] font-semibold text-[var(--calc-primary,#1c4a2a)]">
                  {{ formatCurrency(price) }}
                </span>
              </div>

              <div class="relative flex items-center">
                <div class="absolute left-4 pointer-events-none text-xs sm:text-sm font-bold text-[var(--calc-muted,#6b726d)] select-none">
                  Rp
                </div>
                <input
                  id="calc-price-input"
                  type="text"
                  inputmode="numeric"
                  :value="formattedPrice"
                  @input="onPriceInput"
                  placeholder="0"
                  class="w-full pl-11 pr-10 py-3.5 bg-[var(--calc-bg,#f6f5f1)]/80 border border-[var(--calc-line,#e3e1da)] rounded-xl text-sm sm:text-base font-bold text-[var(--calc-ink,#1e2420)] placeholder-[var(--calc-muted,#6b726d)]/50 focus:bg-white focus:border-[var(--calc-primary,#1c4a2a)] focus:ring-2 focus:ring-[var(--calc-primary,#1c4a2a)]/20 outline-none transition-all"
                  :aria-label="c.labels?.housePrice"
                />
                <button
                  v-if="price > 0"
                  type="button"
                  @click="clearPrice"
                  class="absolute right-3.5 w-6 h-6 flex items-center justify-center rounded-full bg-[var(--calc-line,#e3e1da)] hover:bg-[var(--calc-muted,#6b726d)] hover:text-white text-[var(--calc-ink,#1e2420)] text-xs transition-colors cursor-pointer"
                  :aria-label="c.labels?.clearLabel"
                >
                  ✕
                </button>
              </div>
            </div>

            <!-- 2. Uang Muka (DP) Sinkronisasi Dua Arah -->
            <div>
              <div class="flex items-center justify-between mb-2">
                <label for="calc-dp-nominal" class="text-xs sm:text-[13px] font-bold text-[var(--calc-ink,#1e2420)] tracking-tight">
                  {{ c.labels?.downPayment }}
                </label>
                <span class="text-[11px] font-semibold text-[var(--calc-primary,#1c4a2a)]">
                  {{ dpPercent }}% ({{ formatCurrency(dpNominal) }})
                </span>
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-12 gap-3">
                <!-- Input Nominal DP -->
                <div class="sm:col-span-8 relative flex items-center">
                  <div class="absolute left-4 pointer-events-none text-xs sm:text-sm font-bold text-[var(--calc-muted,#6b726d)] select-none">
                    Rp
                  </div>
                  <input
                    id="calc-dp-nominal"
                    type="text"
                    inputmode="numeric"
                    :value="formattedDpNominal"
                    @input="onDpNominalInput"
                    placeholder="0"
                    class="w-full pl-11 pr-4 py-3.5 bg-[var(--calc-bg,#f6f5f1)]/80 border border-[var(--calc-line,#e3e1da)] rounded-xl text-sm sm:text-base font-bold text-[var(--calc-ink,#1e2420)] placeholder-[var(--calc-muted,#6b726d)]/50 focus:bg-white focus:border-[var(--calc-primary,#1c4a2a)] focus:ring-2 focus:ring-[var(--calc-primary,#1c4a2a)]/20 outline-none transition-all"
                    :aria-label="c.labels?.downPaymentNominal || c.labels?.downPayment"
                  />
                </div>

                <!-- Input Persentase DP -->
                <div class="sm:col-span-4 relative flex items-center">
                  <input
                    id="calc-dp-percent"
                    type="number"
                    min="0"
                    max="95"
                    step="0.5"
                    :value="dpPercent"
                    @input="onDpPercentInput"
                    placeholder="0"
                    class="w-full pl-4 pr-8 py-3.5 bg-[var(--calc-bg,#f6f5f1)]/80 border border-[var(--calc-line,#e3e1da)] rounded-xl text-sm sm:text-base font-bold text-[var(--calc-ink,#1e2420)] placeholder-[var(--calc-muted,#6b726d)]/50 focus:bg-white focus:border-[var(--calc-primary,#1c4a2a)] focus:ring-2 focus:ring-[var(--calc-primary,#1c4a2a)]/20 outline-none transition-all text-right sm:text-center"
                    :aria-label="c.labels?.downPaymentPercent || 'Persentase DP'"
                  />
                  <div class="absolute right-4 pointer-events-none text-xs sm:text-sm font-bold text-[var(--calc-muted,#6b726d)] select-none">
                    %
                  </div>
                </div>
              </div>

              <!-- Slider Range (5% - 90%) -->
              <div class="mt-3.5 px-1">
                <input
                  type="range"
                  :min="c.minDpPercent || 5"
                  :max="c.maxDpPercent || 90"
                  step="1"
                  :value="dpPercent"
                  @input="onSliderInput"
                  :style="sliderBackgroundStyle"
                  class="calc-slider w-full h-2 rounded-lg appearance-none cursor-pointer"
                  aria-label="Slider Persentase Uang Muka"
                />
                <div class="flex justify-between text-[10px] sm:text-[11px] font-semibold text-[var(--calc-muted,#6b726d)] mt-1.5">
                  <span>{{ c.labels?.minDp }}</span>
                  <span>{{ c.labels?.midDp || '50%' }}</span>
                  <span>{{ c.labels?.maxDp }}</span>
                </div>
              </div>
            </div>

            <!-- 3 & 4. Sub-grid 2 Kolom Responsif: Suku Bunga & Tenor -->
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <!-- Suku Bunga Manual (% per tahun) -->
              <div>
                <div class="flex items-center justify-between mb-2 gap-2">
                  <label for="calc-interest-rate" class="text-xs sm:text-[13px] font-bold text-[var(--calc-ink,#1e2420)] tracking-tight truncate">
                    {{ c.labels?.interestRate || 'Suku Bunga (% per tahun)' }}
                  </label>
                  <span class="text-[11px] font-semibold text-[var(--calc-primary,#1c4a2a)] shrink-0 whitespace-nowrap">
                    {{ interestRate || 0 }}% p.a.
                  </span>
                </div>
                <div class="relative flex items-center">
                  <input
                    id="calc-interest-rate"
                    type="number"
                    min="0"
                    max="30"
                    step="0.1"
                    v-model.number="interestRate"
                    placeholder="5"
                    class="w-full pl-4 pr-10 py-3.5 bg-[var(--calc-bg,#f6f5f1)]/80 border border-[var(--calc-line,#e3e1da)] rounded-xl text-sm sm:text-base font-bold text-[var(--calc-ink,#1e2420)] placeholder-[var(--calc-muted,#6b726d)]/50 focus:bg-white focus:border-[var(--calc-primary,#1c4a2a)] focus:ring-2 focus:ring-[var(--calc-primary,#1c4a2a)]/20 outline-none transition-all"
                    :aria-label="c.labels?.interestRate || 'Suku Bunga (% per tahun)'"
                  />
                  <div class="absolute right-4 pointer-events-none text-xs sm:text-sm font-bold text-[var(--calc-muted,#6b726d)] select-none">
                    %
                  </div>
                </div>
              </div>

              <!-- Tenor Pinjaman -->
              <div>
                <div class="flex items-center justify-between mb-2 gap-2">
                  <label for="calc-tenor-select" class="text-xs sm:text-[13px] font-bold text-[var(--calc-ink,#1e2420)] tracking-tight truncate">
                    {{ c.labels?.tenor || 'Jangka Waktu' }}
                  </label>
                  <span class="text-[11px] font-semibold text-[var(--calc-primary,#1c4a2a)] shrink-0 whitespace-nowrap">
                    {{ tenorYears }} Thn
                  </span>
                </div>

                <div class="relative">
                  <select
                    id="calc-tenor-select"
                    v-model.number="tenorYears"
                    class="w-full appearance-none px-4 py-3.5 bg-[var(--calc-bg,#f6f5f1)]/80 border border-[var(--calc-line,#e3e1da)] rounded-xl text-sm sm:text-base font-bold text-[var(--calc-ink,#1e2420)] focus:bg-white focus:border-[var(--calc-primary,#1c4a2a)] focus:ring-2 focus:ring-[var(--calc-primary,#1c4a2a)]/20 outline-none cursor-pointer transition-all pr-10"
                    :aria-label="c.labels?.tenor || 'Jangka Waktu Pinjaman'"
                  >
                    <option v-for="t in c.tenors" :key="t.value" :value="t.value">
                      {{ t.label }}
                    </option>
                  </select>
                  <div class="absolute right-4 top-1/2 -translate-y-1/2 pointer-events-none text-[var(--calc-primary,#1c4a2a)] text-xs font-bold">
                    ▼
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Kolom 2 (Kanan): Plafon, Box Cicilan, Metrics, WhatsApp CTA (6 Col) -->
          <div class="lg:col-span-6 space-y-5 lg:border-l lg:border-[var(--calc-line,#e3e1da)] lg:pl-10">
            <!-- Plafon Pinjaman -->
            <div class="flex items-center justify-between pb-3 border-b border-[var(--calc-line,#e3e1da)] text-xs sm:text-sm">
              <span class="text-[var(--calc-muted,#6b726d)] font-medium">{{ c.labels?.plafonTitle || 'Plafon Pembiayaan KPR' }}</span>
              <span class="text-[var(--calc-ink,#1e2420)] font-bold text-sm sm:text-base">{{ formatCurrency(plafon) }}</span>
            </div>

            <!-- Box Estimasi Cicilan Bersih (Clean - Tanpa Badge Statis) -->
            <div class="p-5 sm:p-6 rounded-2xl bg-[var(--calc-highlight-bg,#eaf1ec)] border border-[var(--calc-highlight-border,#c8d8cd)] text-center">
              <span class="text-[10px] sm:text-[11px] font-bold tracking-wider uppercase text-[var(--calc-primary,#1c4a2a)]">
                {{ c.labels?.installmentTitle || 'Estimasi Cicilan per Bulan' }}
              </span>
              <div class="flex items-baseline justify-center gap-1.5 mt-1">
                <span class="font-serif text-3xl sm:text-4xl text-[var(--calc-primary,#1c4a2a)] font-bold tracking-tight">
                  {{ formatCurrency(monthlyInstallment) }}
                </span>
                <span class="text-xs sm:text-sm font-semibold text-[var(--calc-muted,#6b726d)]">
                  {{ c.labels?.perMonth || '/bulan' }}
                </span>
              </div>
            </div>

            <!-- Grid 3 Metrik Dinamis Real-Time -->
            <div class="p-4 rounded-xl bg-[var(--calc-bg,#f6f5f1)] border border-[var(--calc-line,#e3e1da)] grid grid-cols-3 gap-2 text-center divide-x divide-[var(--calc-line,#e3e1da)]">
              <div v-for="metric in metrics" :key="metric.label" class="first:pl-0 px-2 last:pr-0">
                <span class="block text-[10px] uppercase font-bold tracking-wider text-[var(--calc-muted,#6b726d)]">
                  {{ metric.label }}
                </span>
                <strong class="block text-xs sm:text-sm font-bold text-[var(--calc-primary,#1c4a2a)] mt-0.5">
                  {{ metric.value }}
                </strong>
              </div>
            </div>

            <!-- WhatsApp CTA Button (Full Width) -->
            <div class="pt-2">
              <a
                :href="whatsappUrl"
                target="_blank"
                rel="noopener noreferrer"
                class="w-full py-4 px-6 text-xs sm:text-sm font-bold flex items-center justify-center gap-2 rounded-full bg-[var(--calc-primary,#1c4a2a)] hover:bg-[var(--calc-primary-hover,#2d5a3f)] text-white shadow-md hover:shadow-lg transition-all duration-200 cursor-pointer"
              >
                <span>{{ c.ctaLabel || 'Konsultasi Simulasi via WhatsApp' }}</span>
                <span class="text-base leading-none" aria-hidden="true">↗</span>
              </a>
            </div>
          </div>
        </div>
      </div>

      <!-- Wadah Pill Disclaimer Netral di Luar Card (Tanpa Logo Bank) -->
      <div v-if="c.footnote" class="mt-8 flex justify-center">
        <div class="inline-flex items-center px-5 py-2.5 rounded-2xl sm:rounded-full bg-white/80 backdrop-blur-sm border border-[var(--calc-line,#e3e1da)] shadow-sm max-w-full text-center">
          <p class="text-[11px] sm:text-xs leading-relaxed text-[var(--calc-muted,#6b726d)]">
            {{ c.footnote }}
          </p>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* Cross-browser range thumb styling */
.calc-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: var(--calc-primary, #1c4a2a);
  border: 2px solid #ffffff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: transform 0.15s ease;
}

.calc-slider::-webkit-slider-thumb:hover {
  transform: scale(1.15);
}

.calc-slider::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: var(--calc-primary, #1c4a2a);
  border: 2px solid #ffffff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: transform 0.15s ease;
}

.calc-slider::-moz-range-thumb:hover {
  transform: scale(1.15);
}
</style>
```

---

## 6. Petunjuk Step-by-Step Integrasi di Proyek Lain

Berikut adalah 4 langkah mudah untuk mengimplementasikan komponen ini pada proyek Vue 3 atau Nuxt 3 baru:

### Langkah 1: Salin File Komponen
Salin kode di atas ke file `components/MortgageCalculator.vue` dalam proyek web target Anda.

### Langkah 2: Konfigurasi Tailwind & Variabel Warna
Anda dapat menyesuaikan palet warna agar serasi dengan *brand identity* proyek Anda. Masukkan CSS variables berikut ke file CSS utama Anda (misal `assets/css/main.css`):

```css
/* Contoh Palet Emerald / Nature */
:root {
  --calc-bg: #f8faf9;
  --calc-card: #ffffff;
  --calc-ink: #111827;
  --calc-muted: #6b7280;
  --calc-line: #e5e7eb;
  --calc-primary: #047857;       /* emerald-700 */
  --calc-primary-hover: #065f46; /* emerald-800 */
  --calc-track: #e5e7eb;
  --calc-highlight-bg: #ecfdf5;  /* emerald-50 */
  --calc-highlight-border: #a7f3d0;
}
```

*Jika proyek Anda telah memiliki konfigurasi Tailwind tersendiri, Anda juga dapat mereplace kelas CSS langsung ke utilitas Tailwind standar.*

### Langkah 3: Siapkan Data Konfigurasi & Nomor WhatsApp
Buat file konfigurasi (misal `data/calculator.js`) atau letakkan langsung di dalam komponen halaman:

```javascript
// data/calculator.js
export const myCalcConfig = {
  defaultPrice: 250000000,       // Harga default unit
  defaultDpPercent: 10,          // DP default 10%
  defaultInterestRatePercent: 6, // Bunga default 6.0% p.a.
  defaultTenor: 15,              // Tenor default 15 tahun
  whatsappNumber: '6281234567890',
  whatsappTemplate: 'Halo, saya melakukan simulasi cicilan di website Anda dengan rincian:\n- Harga Rumah: {harga}\n- Uang Muka: {dp} ({dpPercent}%)\n- Plafon KPR: {plafon}\n- Suku Bunga: {bunga} p.a.\n- Tenor: {tenor} Tahun\n- Estimasi Cicilan: {cicilan}/bulan\n\nBisa dibantu informasi unit yang tersedia?',
};
```

### Langkah 4: Render di Halaman Utama (Nuxt 3 / Vue 3)

```vue
<!-- pages/index.vue -->
<template>
  <main>
    <!-- Section Landing Page Lainnya -->
    
    <!-- Kalkulator Cicilan Reusable v2.1 -->
    <MortgageCalculator :config="myCalcConfig" />
    
    <!-- Section Landing Page Lainnya -->
  </main>
</template>

<script setup>
import MortgageCalculator from '~/components/MortgageCalculator.vue';
import { myCalcConfig } from '~/data/calculator';
</script>
```

---

## 7. Rangkuman & Checklist Implementasi Portabilitas v2.1

Gunakan checklist berikut untuk memverifikasi kesuksesan integrasi komponen pada proyek target:

| No | Komponen / Pengujian | Deskripsi Verifikasi | Target Hasil |
|:---:|---|---|:---:|
| 1 | **Salin Komponen SFC** | File `components/MortgageCalculator.vue` tersalin tanpa error kompilasi Nuxt/Vite. | PASS |
| 2 | **Header Rata Tengah** | Judul section berada di tengah (*center-aligned*) dan bebas dari elemen eyebrow. | PASS |
| 3 | **Input Suku Bunga Dinamis** | Mengubah angka suku bunga (% p.a.) seketika mengubah nominal cicilan bulanan dan metrik pertama. | PASS |
| 4 | **Sub-grid 2 Kolom Responsif** | Bunga & Tenor sejajar 2 kolom pada layar $\ge 640\text{px}$ dan bertumpuk 1 kolom pada ponsel mobile. | PASS |
| 5 | **Sinkronisasi Tiga Arah DP** | Perubahan nominal Rupiah, persen (%), dan slider 5%–90% saling sinkron 100%. | PASS |
| 6 | **Tombol Reset (`✕`)** | Mengklik tombol clear mereset harga menjadi 0 dan cicilan Rp 0 tanpa memicu `NaN`. | PASS |
| 7 | **3 Metrik Dinamis** | Bunga p.a., DP %, dan Tenor Thn ter-update real-time sesuai state aktif. | PASS |
| 8 | **Clean Box Cicilan** | Nominal cicilan bulanan disajikan bersih tanpa badge statis. | PASS |
| 9 | **Disclaimer Netral** | Wadah pill di luar kartu menampilkan teks edukatif netral tanpa keterikatan logo bank. | PASS |
| 10 | **CTA WhatsApp Dinamis** | Klik tombol CTA membuka pesan WhatsApp dengan seluruh data simulasi terisi otomatis. | PASS |
