<script setup>
import { computed, ref } from 'vue';
import { useContent } from '~/composables/useContent';

const { content, wa } = useContent();

const calc = computed(() => content.calculator || {});
const labels = computed(() => calc.value.labels || {});

// State & Defaults
const price = ref(calc.value.defaultPrice ?? 166000000);
const dpPercent = ref(calc.value.defaultDpPercent ?? 5);
const dpNominal = ref(Math.round(price.value * (dpPercent.value / 100)));
const tenorYears = ref(calc.value.defaultTenor ?? 20);
const showBreakdown = ref(false);

// Format Rupiah
function formatCurrency(val) {
  if (val === null || val === undefined || isNaN(val)) return 'Rp 0';
  return 'Rp ' + Math.round(val).toLocaleString('id-ID');
}

// Formatted strings for display in inputs
const formattedPrice = computed(() => {
  return price.value > 0 ? price.value.toLocaleString('id-ID') : '';
});

const formattedDpNominal = computed(() => {
  return dpNominal.value > 0 ? dpNominal.value.toLocaleString('id-ID') : '';
});

// Event Handlers for 2-Way Sync
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

// Calculations
const plafon = computed(() => {
  return Math.max(0, price.value - dpNominal.value);
});

const monthlyInstallment = computed(() => {
  const P = plafon.value;
  if (P <= 0) return 0;
  const annualRate = calc.value.annualInterestRate ?? 0.05;
  const years = tenorYears.value || 20;
  const n = years * 12;
  if (annualRate <= 0) {
    return Math.round(P / n);
  }
  const r = annualRate / 12;
  const factor = Math.pow(1 + r, n);
  const emi = P * (r * factor) / (factor - 1);
  return Math.round(emi);
});

const provisionFee = computed(() => {
  const rate = calc.value.provisionRate ?? 0.005;
  return Math.round(plafon.value * rate);
});

const adminFee = computed(() => {
  return plafon.value > 0 ? (calc.value.adminFee ?? 500000) : 0;
});

const totalInitialFee = computed(() => {
  if (plafon.value <= 0) return 0;
  return monthlyInstallment.value + provisionFee.value + adminFee.value;
});

// Dynamic WhatsApp link
const waConsultLink = computed(() => {
  return wa('simulasiKpr', {
    harga: formatCurrency(price.value),
    dp: formatCurrency(dpNominal.value),
    dpPercent: dpPercent.value,
    plafon: formatCurrency(plafon.value),
    tenor: tenorYears.value,
    cicilan: formatCurrency(monthlyInstallment.value),
  });
});
</script>

<template>
  <section class="calc-section relative py-20 lg:py-28 scroll-mt-20 bg-white" id="kalkulator">
    <div class="container relative z-10">
      <!-- Section Heading -->
      <div class="section-top mb-10 lg:mb-12 reveal">
        <div class="section-heading max-w-[680px]">
          <div class="mb-4 h-0.5 w-10 rounded-full bg-[var(--green)]" aria-hidden="true" />
          <h2 class="font-sans font-bold text-3xl sm:text-4xl tracking-[-0.02em] leading-[1.18] text-[var(--ink)] mb-3">
            {{ calc.title || (calc.titleLines || []).join(' ') }}
          </h2>
          <p class="text-sm sm:text-[15px] text-[var(--muted)] leading-relaxed">
            {{ calc.subtitle }}
          </p>
        </div>
      </div>

      <!-- SATU Card Landscape 2 Kolom -->
      <div class="bg-[var(--white)] border border-[var(--line)] rounded-[26px] p-6 sm:p-8 lg:p-10 shadow-[var(--shadow)] reveal">
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 lg:gap-12 items-start">
          <!-- Kolom 1 (Kiri): Input Data -->
          <div class="lg:col-span-6 space-y-6">
            <!-- 1. Input Harga Rumah -->
            <div>
              <div class="flex items-center justify-between mb-2">
                <label for="calc-price-input" class="text-xs sm:text-[13px] font-bold text-[var(--ink)] tracking-tight">
                  {{ labels.price || 'Harga Rumah' }}
                </label>
                <span v-if="price > 0" class="text-[11px] font-semibold text-[var(--green)]">
                  {{ formatCurrency(price) }}
                </span>
              </div>

              <div class="relative flex items-center">
                <div class="absolute left-4 pointer-events-none text-xs sm:text-sm font-bold text-[var(--muted)] select-none">
                  Rp
                </div>
                <input
                  id="calc-price-input"
                  type="text"
                  inputmode="numeric"
                  :value="formattedPrice"
                  @input="onPriceInput"
                  placeholder="0"
                  class="w-full pl-11 pr-10 py-3.5 bg-[var(--paper)]/50 border border-[var(--line)] rounded-xl text-sm sm:text-base font-bold text-[var(--ink)] placeholder-[var(--muted)]/50 focus:bg-white focus:border-[var(--green)] focus:ring-2 focus:ring-[var(--green)]/20 outline-none transition-all"
                  :aria-label="labels.price || 'Harga Rumah'"
                />
                <button
                  v-if="price > 0"
                  type="button"
                  @click="clearPrice"
                  class="absolute right-3.5 w-6 h-6 flex items-center justify-center rounded-full bg-[var(--line)] hover:bg-[var(--muted)] hover:text-white text-[var(--ink)] text-xs transition-colors cursor-pointer"
                  :aria-label="labels.clearPrice || 'Hapus input harga'"
                >
                  ✕
                </button>
              </div>
            </div>

            <!-- 2. Uang Muka (DP) Sinkronisasi Dua Arah -->
            <div>
              <div class="flex items-center justify-between mb-2">
                <label for="calc-dp-nominal" class="text-xs sm:text-[13px] font-bold text-[var(--ink)] tracking-tight">
                  {{ labels.dp || 'Uang Muka (DP)' }}
                </label>
                <span class="text-[11px] font-semibold text-[var(--green)]">
                  {{ dpPercent }}% ({{ formatCurrency(dpNominal) }})
                </span>
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-12 gap-3">
                <!-- Input Nominal DP (sm:col-span-8) -->
                <div class="sm:col-span-8 relative flex items-center">
                  <div class="absolute left-4 pointer-events-none text-xs sm:text-sm font-bold text-[var(--muted)] select-none">
                    Rp
                  </div>
                  <input
                    id="calc-dp-nominal"
                    type="text"
                    inputmode="numeric"
                    :value="formattedDpNominal"
                    @input="onDpNominalInput"
                    placeholder="0"
                    class="w-full pl-11 pr-4 py-3.5 bg-[var(--paper)]/50 border border-[var(--line)] rounded-xl text-sm sm:text-base font-bold text-[var(--ink)] placeholder-[var(--muted)]/50 focus:bg-white focus:border-[var(--green)] focus:ring-2 focus:ring-[var(--green)]/20 outline-none transition-all"
                    :aria-label="labels.dp || 'Uang Muka Nominal'"
                  />
                </div>

                <!-- Input Persentase DP (sm:col-span-4) -->
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
                    class="w-full pl-4 pr-8 py-3.5 bg-[var(--paper)]/50 border border-[var(--line)] rounded-xl text-sm sm:text-base font-bold text-[var(--ink)] placeholder-[var(--muted)]/50 focus:bg-white focus:border-[var(--green)] focus:ring-2 focus:ring-[var(--green)]/20 outline-none transition-all text-right sm:text-center"
                    :aria-label="labels.dpPercent || 'Persentase DP'"
                  />
                  <div class="absolute right-4 pointer-events-none text-xs sm:text-sm font-bold text-[var(--muted)] select-none">
                    %
                  </div>
                </div>
              </div>

              <!-- Slider Range (5% - 90%) -->
              <div class="mt-3.5 px-1">
                <input
                  type="range"
                  min="5"
                  max="90"
                  step="1"
                  :value="dpPercent"
                  @input="onSliderInput"
                  class="w-full h-2 bg-[var(--paper-deep)] rounded-lg appearance-none cursor-pointer accent-[var(--green)]"
                  aria-label="Slider Persentase Uang Muka"
                />
                <div class="flex justify-between text-[10px] sm:text-[11px] font-semibold text-[var(--muted)] mt-1.5">
                  <span>{{ labels.minDp || '5% Min DP' }}</span>
                  <span>50%</span>
                  <span>{{ labels.maxDp || '90% Max DP' }}</span>
                </div>
              </div>
            </div>

            <!-- 3. Tenor Kredit -->
            <div>
              <div class="flex items-center justify-between mb-2">
                <label for="calc-tenor-select" class="text-xs sm:text-[13px] font-bold text-[var(--ink)] tracking-tight">
                  {{ labels.tenor || 'Jangka Waktu Pinjaman' }}
                </label>
                <span class="text-[11px] font-semibold text-[var(--green)]">
                  {{ tenorYears }} Tahun ({{ tenorYears * 12 }} Bulan)
                </span>
              </div>

              <div class="relative">
                <select
                  id="calc-tenor-select"
                  v-model.number="tenorYears"
                  class="w-full appearance-none px-4 py-3.5 bg-[var(--paper)]/50 border border-[var(--line)] rounded-xl text-sm sm:text-base font-bold text-[var(--ink)] focus:bg-white focus:border-[var(--green)] focus:ring-2 focus:ring-[var(--green)]/20 outline-none cursor-pointer transition-all pr-10"
                  :aria-label="labels.tenor || 'Jangka Waktu Pinjaman'"
                >
                  <option
                    v-for="t in calc.tenors"
                    :key="t.value"
                    :value="t.value"
                  >
                    {{ t.label }}
                  </option>
                </select>
                <div class="absolute right-4 top-1/2 -translate-y-1/2 pointer-events-none text-[var(--green)] text-xs font-bold">
                  ▼
                </div>
              </div>
            </div>
          </div>

          <!-- Kolom 2 (Kanan): Plafon, Box Cicilan, Metrics, Collapsible Biaya, WhatsApp CTA -->
          <div class="lg:col-span-6 space-y-5 lg:border-l lg:border-[var(--line)] lg:pl-10">
            <!-- Plafon Kredit Row -->
            <div class="flex items-center justify-between pb-3 border-b border-[var(--line)] text-xs sm:text-sm">
              <span class="text-[var(--muted)] font-medium">{{ labels.plafon || 'Plafon Pembiayaan KPR' }}</span>
              <span class="text-[var(--ink)] font-bold text-sm sm:text-base">{{ formatCurrency(plafon) }}</span>
            </div>

            <!-- Highlight Estimasi Cicilan -->
            <div class="p-5 sm:p-6 rounded-2xl bg-[var(--sage-pale)]/70 border border-[var(--green-soft)] text-center sm:text-left sm:flex sm:items-center sm:justify-between gap-4">
              <div>
                <span class="text-[10px] sm:text-[11px] font-bold tracking-wider uppercase text-[var(--green-deep)]">
                  {{ labels.installment || 'Estimasi Cicilan per Bulan' }}
                </span>
                <div class="flex items-baseline justify-center sm:justify-start gap-1.5 mt-1">
                  <span class="font-sans text-3xl sm:text-4xl text-[var(--green-deep)] font-bold tracking-tight">
                    {{ formatCurrency(monthlyInstallment) }}
                  </span>
                  <span class="text-xs sm:text-sm font-semibold text-[var(--muted)]">
                    {{ labels.perMonth || '/bulan' }}
                  </span>
                </div>
              </div>
              <div class="mt-3 sm:mt-0 flex justify-center">
                <span class="inline-flex items-center gap-1.5 px-3.5 py-1.5 rounded-full text-[11px] font-bold bg-white text-[var(--green-deep)] shadow-sm">
                  <span>{{ labels.rateBadge || 'Bunga 5% Anuitas BCA' }}</span>
                </span>
              </div>
            </div>

            <!-- Metrics Grid (Bunga KPR, Uang Muka, Tenor KPR) -->
            <div
              v-if="calc.metrics && calc.metrics.length"
              class="p-4 rounded-xl bg-[var(--paper)]/80 border border-[var(--line)] grid grid-cols-3 gap-2 text-center divide-x divide-[var(--line)]"
            >
              <div v-for="metric in calc.metrics" :key="metric.label" class="first:pl-0 px-2 last:pr-0">
                <span class="block text-[10px] uppercase font-bold tracking-wider text-[var(--muted)]">{{ metric.label }}</span>
                <strong class="block text-xs sm:text-sm font-bold text-[var(--green-deep)] mt-0.5">{{ metric.value }}</strong>
              </div>
            </div>

            <!-- Collapsible Biaya Akad Awal -->
            <div>
              <button
                type="button"
                @click="showBreakdown = !showBreakdown"
                class="inline-flex items-center gap-1.5 text-xs font-bold text-[var(--green)] hover:text-[var(--green-deep)] transition-colors py-1 cursor-pointer"
                :aria-expanded="showBreakdown"
              >
                <span>{{ showBreakdown ? (labels.breakdownClose || 'Tutup Rincian Biaya Akad Awal ▴') : (labels.breakdownOpen || 'Lihat Rincian Biaya Akad Awal ▾') }}</span>
              </button>

              <div
                v-show="showBreakdown"
                class="mt-3 p-4 rounded-xl bg-[var(--paper)]/70 border border-[var(--line)] space-y-2.5 text-xs transition-all"
              >
                <div class="flex justify-between items-center text-[var(--muted)]">
                  <span>{{ labels.provisionFee || 'Provisi Bank (0.5%)' }}</span>
                  <span class="font-semibold text-[var(--ink)]">{{ formatCurrency(provisionFee) }}</span>
                </div>
                <div class="flex justify-between items-center text-[var(--muted)]">
                  <span>{{ labels.adminFee || 'Administrasi Bank' }}</span>
                  <span class="font-semibold text-[var(--ink)]">{{ formatCurrency(adminFee) }}</span>
                </div>
                <div class="flex justify-between items-center pt-2.5 border-t border-[var(--line)] font-bold text-[var(--ink)] text-xs sm:text-sm">
                  <span>{{ labels.totalInitialFee || 'Total Estimasi Biaya Pertama' }}</span>
                  <span class="text-[var(--green-deep)] text-sm sm:text-base">{{ formatCurrency(totalInitialFee) }}</span>
                </div>
                <p v-if="labels.initialFeeNote" class="text-[10.5px] text-[var(--muted)] leading-relaxed pt-1">
                  *{{ labels.initialFeeNote }}
                </p>
              </div>
            </div>

            <!-- WhatsApp CTA Button -->
            <div class="pt-2">
              <a
                :href="waConsultLink"
                target="_blank"
                rel="noopener noreferrer"
                class="btn btn-primary w-full py-4 text-xs sm:text-sm font-bold flex items-center justify-center gap-2 rounded-full shadow-lg"
              >
                <span>{{ calc.ctaLabel }}</span>
                <span class="arrow text-lg leading-none">↗</span>
              </a>
            </div>
          </div>
        </div>
      </div>

      <!-- Disclaimer & Logo Bank BCA di Bawah Card Utama -->
      <div v-if="calc.footnote" class="mt-8 flex justify-center reveal">
        <div class="inline-flex flex-col sm:flex-row items-center gap-2.5 sm:gap-3.5 px-5 py-2.5 sm:py-2 rounded-2xl sm:rounded-full bg-[var(--white)]/80 backdrop-blur-sm border border-[var(--line)] shadow-sm max-w-[820px] text-center sm:text-left">
          <div v-if="calc.bankPartner?.logo" class="shrink-0 flex items-center justify-center">
            <img
              :src="calc.bankPartner.logo"
              :alt="calc.bankPartner.name || 'Bank BCA'"
              class="h-5 sm:h-6 w-auto object-contain"
              loading="lazy"
            />
          </div>
          <span v-if="calc.bankPartner?.logo" class="hidden sm:block h-3.5 w-px bg-[var(--line)] shrink-0" aria-hidden="true"></span>
          <p class="text-[11px] sm:text-xs leading-relaxed text-[var(--muted)]">
            {{ calc.footnote }}
          </p>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.calc-section {
  position: relative;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: var(--green);
  border: 2px solid #fff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: transform 0.15s ease;
}

input[type="range"]::-webkit-slider-thumb:hover {
  transform: scale(1.15);
}

input[type="range"]::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: var(--green);
  border: 2px solid #fff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: transform 0.15s ease;
}

input[type="range"]::-moz-range-thumb:hover {
  transform: scale(1.15);
}
</style>
