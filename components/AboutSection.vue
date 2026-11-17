<script setup>
import { useContent } from '~/composables/useContent';

const { content } = useContent();
</script>

<template>
  <section id="tentang" class="bg-white py-20 lg:py-28 scroll-mt-20 overflow-x-clip">
    <div class="container">
      <div class="grid grid-cols-1 lg:grid-cols-[0.95fr_1.05fr] gap-10 lg:gap-16 items-center">
        <!-- Kiri: Foto Utama dengan Aksen Pale-Blue -->
        <div class="relative mx-auto w-full max-w-[520px] lg:max-w-none reveal">
          <div
            class="pointer-events-none absolute -top-4 -left-4 h-44 w-44 rounded-[36px] bg-[var(--sage-pale)] sm:h-56 sm:w-56"
            aria-hidden="true"
          />
          <div
            class="pointer-events-none absolute -bottom-4 -right-4 h-36 w-36 rounded-[32px] bg-[var(--green-soft)]/60 sm:h-48 sm:w-48"
            aria-hidden="true"
          />
          <div class="relative overflow-hidden rounded-[28px] bg-[var(--sage-pale)] shadow-[var(--shadow)] aspect-[4/3]">
            <img
              :src="content.about.photos.main"
              :alt="`Lingkungan hunian ${content.meta.projectName}`"
              loading="lazy"
              class="h-full w-full object-cover"
            />
          </div>
        </div>

        <!-- Kanan: Copy + Grid 2x2 Kartu Fitur -->
        <div class="reveal">
          <div class="mb-4 h-0.5 w-10 rounded-full bg-[var(--green)]" aria-hidden="true" />
          <h2 class="font-sans text-3xl sm:text-4xl font-bold tracking-[-0.02em] leading-[1.18] text-[var(--ink)]">
            {{ content.about.title }}
          </h2>
          <p
            v-if="content.about.description?.[0]?.text"
            class="mt-4 text-sm sm:text-[15px] leading-[1.7] text-[var(--muted)] max-w-[540px]"
          >
            {{ content.about.description[0].text }}
          </p>

          <!-- Grid 2x2 Kartu Fitur Putih -->
          <div
            v-if="content.whyCards?.length"
            class="mt-7 grid grid-cols-1 sm:grid-cols-2 gap-3.5 sm:gap-4"
          >
            <div
              v-for="card in content.whyCards"
              :key="card.id"
              class="rounded-2xl bg-white border border-[var(--line)] p-4 sm:p-5 shadow-sm flex items-center gap-3.5 transition-all duration-200 hover:border-[var(--green-soft)] hover:shadow-md"
            >
              <div
                class="w-11 h-11 shrink-0 rounded-xl bg-[var(--sage-pale)] text-[var(--green)] grid place-items-center"
                aria-hidden="true"
              >
                <!-- Lokasi Strategis -->
                <svg v-if="card.iconKey === 'location'" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M12 21s-6.5-5.8-6.5-11a6.5 6.5 0 1 1 13 0c0 5.2-6.5 11-6.5 11Z" />
                  <circle cx="12" cy="10" r="2.3" />
                </svg>
                <!-- Lingkungan Asri -->
                <svg v-else-if="card.iconKey === 'nature'" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M12 20v-7" />
                  <path d="M7.5 13.5a4.5 4.5 0 0 1 1.4-8.7 4.7 4.7 0 0 1 6.2 0 4.5 4.5 0 0 1 1.4 8.7Z" />
                </svg>
                <!-- Kualitas Bangunan -->
                <svg v-else-if="card.iconKey === 'shield'" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M12 3 5 6v6c0 4.5 3 7.8 7 9 4-1.2 7-4.5 7-9V6l-7-3Z" />
                  <path d="m9.2 12.2 2 2 3.8-4" />
                </svg>
                <!-- Nilai Investasi -->
                <svg v-else class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M3 17l6-6 4 4 8-8" />
                  <path d="M15 7h6v6" />
                </svg>
              </div>
              <span class="text-[13px] sm:text-sm font-semibold leading-snug text-[var(--ink)]">
                {{ card.title }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
