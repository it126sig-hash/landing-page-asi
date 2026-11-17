<script setup>
import { computed, ref } from 'vue';
import { useContent } from '~/composables/useContent';

const { content, houseTypes } = useContent();

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
  <section id="tipe" class="bg-[var(--sage-pale)] py-20 lg:py-28 scroll-mt-20">
    <div class="container">
      <div class="grid grid-cols-1 lg:grid-cols-[0.34fr_0.66fr] gap-8 lg:gap-12 items-center">
        <!-- Kiri: Heading + Copy + CTA Outlined Pill -->
        <div class="reveal">
          <div class="mb-4 h-0.5 w-10 rounded-full bg-[var(--green)]" aria-hidden="true" />
          <h2 class="font-sans text-3xl sm:text-4xl font-bold tracking-[-0.02em] leading-[1.18] text-[var(--ink)]">
            {{ content.products.title }}
          </h2>
          <p class="mt-4 text-sm sm:text-[15px] leading-[1.7] text-[var(--muted)]">
            {{ content.products.subtitle }}
          </p>
          <div class="mt-7">
            <a
              :href="content.products.ctaHref || '#kalkulator'"
              class="btn btn-outline"
            >
              <span>{{ content.products.ctaLabel }}</span>
              <span class="arrow" aria-hidden="true">→</span>
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
              <div class="aspect-[16/10] w-full overflow-hidden bg-[var(--paper-deep)]">
                <img
                  :src="t.thumbnail || t.gallery?.[0]"
                  :alt="t.name"
                  loading="lazy"
                  class="h-full w-full object-cover transition-transform duration-500 group-hover:scale-105"
                />
              </div>

              <div class="p-5 flex-1 flex flex-col justify-between">
                <h3 class="font-sans text-lg font-bold tracking-tight text-[var(--ink)]">
                  {{ t.name }}
                </h3>

                <div class="mt-3 flex items-center justify-between gap-2">
                  <span class="inline-flex items-center gap-1.5 text-xs font-medium text-[var(--muted)]">
                    <svg
                      class="w-3.5 h-3.5 shrink-0 text-[var(--green)]"
                      viewBox="0 0 24 24"
                      fill="none"
                      stroke="currentColor"
                      stroke-width="2"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      aria-hidden="true"
                    >
                      <path d="M3 10.5 12 3l9 7.5" />
                      <path d="M5 9.5V20h14V9.5" />
                    </svg>
                    <span>LB {{ t.lb }}m² · LT {{ t.lt }}m² · {{ t.tag }}</span>
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
