<script setup>
import { computed, ref } from 'vue';
import { useContent } from '~/composables/useContent';

const { content, houseTypes } = useContent();

const watermarkWords = computed(() =>
  (content.meta?.projectName || 'ALAM SANGGAR INDAH').trim().split(/\s+/)
);

const selectedTypeId = ref(null);
const modalOpen = ref(false);
const selectedType = computed(() => houseTypes.find((t) => t.id === selectedTypeId.value) || null);

function openType(id) {
  selectedTypeId.value = id;
  modalOpen.value = true;
}

function closeModal() {
  modalOpen.value = false;
}
</script>

<template>
  <section id="tipe" class="relative overflow-hidden bg-[var(--sage-pale)] py-20 lg:py-28 scroll-mt-20">
    <!-- Background Watermark (3 baris per kata di pojok kiri bawah) -->
    <div
      class="pointer-events-none select-none absolute bottom-0 left-0 z-0 flex flex-col items-start pl-4 pb-3 sm:pl-8 sm:pb-5"
      aria-hidden="true"
    >
      <span
        v-for="(word, idx) in watermarkWords"
        :key="idx"
        class="block font-display font-extrabold uppercase leading-[0.88] tracking-[0.06em] text-[clamp(2.6rem,7.5vw,5.75rem)] text-[color:var(--green-deep)] opacity-[0.055] whitespace-nowrap"
      >
        {{ word }}
      </span>
    </div>

    <div class="container relative z-10">
      <div class="grid grid-cols-1 lg:grid-cols-[0.34fr_0.66fr] gap-8 lg:gap-12 items-center">
        <!-- Kiri: Eyebrow + Heading + Copy + CTA Outlined Pill -->
        <div class="reveal">
          <span v-if="content.products.eyebrow" class="eyebrow mb-4">
            {{ content.products.eyebrow }}
          </span>
          <h2 class="font-sans text-3xl sm:text-4xl font-bold tracking-[-0.02em] leading-[1.18] text-[var(--ink)]">
            <template v-if="content.products.titleLines?.length >= 2">
              {{ content.products.titleLines[0] }}
              <br />
              <em class="serif italic font-normal text-[var(--green)]">{{ content.products.titleLines[1] }}</em>
            </template>
            <template v-else>
              {{ content.products.title }}
            </template>
          </h2>
          <p class="mt-4 text-sm sm:text-[15px] leading-[1.7] text-[var(--muted)]">
            {{ content.products.subtitle }}
          </p>
          <div class="mt-7">
            <a
              :href="content.products.ctaHref || '#kalkulator'"
              class="btn btn-outline cursor-pointer"
            >
              <span>{{ content.products.ctaLabel }}</span>
              <svg
                class="w-4 h-4 shrink-0 transition-transform duration-200 group-hover:translate-x-1"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2.2"
                stroke-linecap="round"
                stroke-linejoin="round"
                aria-hidden="true"
              >
                <path d="M5 12h14" />
                <path d="m13 6 6 6-6 6" />
              </svg>
            </a>
          </div>
        </div>

        <!-- Kanan: Grid Kartu Produk -->
        <div class="reveal">
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-5">
            <article
              v-for="t in houseTypes"
              :key="t.id"
              role="button"
              tabindex="0"
              :aria-label="`Buka detail ${t.name}`"
              class="group rounded-2xl bg-white border border-[var(--line)] shadow-sm overflow-hidden flex flex-col hover:-translate-y-1 hover:shadow-[var(--shadow)] transition-all duration-300 cursor-pointer focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[var(--green)]"
              @click="openType(t.id)"
              @keydown.enter.prevent="openType(t.id)"
              @keydown.space.prevent="openType(t.id)"
            >
              <div class="relative aspect-[16/10] w-full overflow-hidden bg-[var(--paper-deep)]">
                <img
                  :src="t.thumbnail || t.gallery?.[0]"
                  :alt="t.name"
                  loading="lazy"
                  class="h-full w-full object-cover transition-transform duration-500 group-hover:scale-105"
                />
                <span
                  v-if="t.tag"
                  class="absolute top-3 left-3 rounded-full bg-white/95 backdrop-blur-sm border border-[var(--line)] px-3 py-1 text-[10px] font-bold uppercase tracking-wider text-[var(--green-deep)] shadow-xs"
                >
                  {{ t.tag }}
                </span>
              </div>

              <div class="p-5 flex-1 flex flex-col justify-between gap-4">
                <div>
                  <h3 class="font-sans text-lg font-bold tracking-tight text-[var(--ink)]">
                    {{ t.name }}
                  </h3>
                  <p v-if="t.headline" class="mt-1 text-xs sm:text-[13px] leading-relaxed text-[var(--muted)]">
                    {{ t.headline }}
                  </p>

                  <!-- Ringkasan LB, LT, KT, KM dengan ikon SVG -->
                  <div class="mt-3.5 grid grid-cols-2 gap-2 text-[11px] font-semibold text-[var(--ink)]/85">
                    <span class="inline-flex items-center gap-1.5 rounded-lg bg-[var(--sage-pale)]/80 px-2.5 py-1.5 tabular-nums">
                      <svg class="w-3.5 h-3.5 shrink-0 text-[var(--green)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                        <path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z" />
                        <polyline points="9 22 9 12 15 12 15 22" />
                      </svg>
                      LB {{ t.lb }} m²
                    </span>
                    <span class="inline-flex items-center gap-1.5 rounded-lg bg-[var(--sage-pale)]/80 px-2.5 py-1.5 tabular-nums">
                      <svg class="w-3.5 h-3.5 shrink-0 text-[var(--green)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                        <rect width="18" height="18" x="3" y="3" rx="2" />
                        <path d="M3 9h18M9 21V9" />
                      </svg>
                      LT {{ t.lt }} m²
                    </span>
                    <span class="inline-flex items-center gap-1.5 rounded-lg bg-[var(--sage-pale)]/80 px-2.5 py-1.5 tabular-nums">
                      <svg class="w-3.5 h-3.5 shrink-0 text-[var(--green)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                        <path d="M2 4v16M2 8h18a2 2 0 0 1 2 2v10M2 17h20M6 8v9" />
                      </svg>
                      {{ t.kt }} Kamar Tidur
                    </span>
                    <span class="inline-flex items-center gap-1.5 rounded-lg bg-[var(--sage-pale)]/80 px-2.5 py-1.5 tabular-nums">
                      <svg class="w-3.5 h-3.5 shrink-0 text-[var(--green)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                        <path d="M4 12h16a1 1 0 0 1 1 1v3a4 4 0 0 1-4 4H7a4 4 0 0 1-4-4v-3a1 1 0 0 1 1-1Z" />
                        <path d="M6 12V5a2 2 0 0 1 2-2h1" />
                      </svg>
                      {{ t.km }} Kamar Mandi
                    </span>
                  </div>
                </div>

                <div class="pt-3 border-t border-[var(--line)] flex items-center justify-between gap-2">
                  <span class="text-xs font-bold text-[var(--green)] group-hover:text-[var(--green-deep)] transition-colors">
                    Lihat Detail &amp; Denah
                  </span>

                  <button
                    type="button"
                    class="w-9 h-9 shrink-0 rounded-full bg-[var(--sage-pale)] text-[var(--green)] grid place-items-center group-hover:bg-[var(--green)] group-hover:text-white transition-colors duration-200 cursor-pointer"
                    :aria-label="`Lihat detail ${t.name}`"
                    @click.stop="openType(t.id)"
                  >
                    <svg
                      class="w-4 h-4"
                      viewBox="0 0 24 24"
                      fill="none"
                      stroke="currentColor"
                      stroke-width="2.2"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      aria-hidden="true"
                    >
                      <path d="M5 12h14" />
                      <path d="m13 6 6 6-6 6" />
                    </svg>
                  </button>
                </div>
              </div>
            </article>
          </div>

          <p v-if="content.products.footnote" class="mt-4 text-[11px] leading-relaxed text-[var(--muted)]">
            {{ content.products.footnote }}
          </p>
        </div>
      </div>
    </div>

    <!-- Modal Detail Unit -->
    <HouseTypeModal :type="modalOpen ? selectedType : null" @close="closeModal" />
  </section>
</template>
