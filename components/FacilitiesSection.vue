<script setup>
import { computed } from 'vue';
import { useContent } from '~/composables/useContent';

const { content } = useContent();

const watermarkWords = computed(() =>
  (content.meta?.projectName || 'ALAM SANGGAR INDAH').trim().split(/\s+/)
);

const headingParts = computed(() => {
  const title = content.facilities?.title || 'Fasilitas Lengkap';
  const words = title.trim().split(/\s+/);
  if (words.length <= 1) return { lead: title, accent: '' };
  const accent = words.pop();
  return { lead: words.join(' '), accent };
});
</script>

<template>
  <section
    v-if="content.facilities"
    id="fasilitas"
    class="relative overflow-hidden bg-[var(--sage-pale)] py-20 lg:py-28 scroll-mt-20"
  >
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
      <div class="grid grid-cols-1 lg:grid-cols-[0.44fr_0.56fr] gap-10 lg:gap-12 items-center">
        <!-- Kiri: Foto Fasilitas -->
        <div class="reveal">
          <div class="overflow-hidden rounded-2xl bg-white shadow-[var(--shadow)] border border-[var(--line)] aspect-[4/3]">
            <img
              :src="content.facilities.image"
              :alt="`${content.facilities.title} ${content.meta.projectName}`"
              loading="lazy"
              class="h-full w-full object-cover"
            />
          </div>
        </div>

        <!-- Kanan: Copy + 5 Kartu Ikon Fasilitas (Horizontal Icon-Left Lega) -->
        <div class="reveal">
          <span v-if="content.facilities.eyebrow" class="eyebrow mb-4">
            {{ content.facilities.eyebrow }}
          </span>
          <h2 class="font-sans text-3xl sm:text-4xl font-bold tracking-[-0.02em] leading-[1.18] text-[var(--ink)]">
            {{ headingParts.lead }}
            <em v-if="headingParts.accent" class="serif italic font-normal text-[var(--green)]">{{ headingParts.accent }}</em>
          </h2>
          <p class="mt-3.5 text-sm sm:text-[15px] leading-[1.7] text-[var(--muted)] max-w-[480px]">
            {{ content.facilities.subtitle }}
          </p>

          <div class="mt-7 grid grid-cols-1 sm:grid-cols-2 gap-3.5">
            <div
              v-for="(item, idx) in content.facilities.items"
              :key="item.id"
              class="rounded-2xl bg-white border border-[var(--line)] p-4 shadow-sm flex items-center justify-between gap-3 transition-all duration-200 hover:-translate-y-0.5 hover:border-[var(--green-soft)] hover:shadow-md sm:last:col-span-2"
            >
              <div class="flex items-center gap-3.5 min-w-0">
                <div
                  class="w-11 h-11 rounded-xl bg-[var(--sage-pale)] text-[var(--green)] grid place-items-center shrink-0"
                  aria-hidden="true"
                >
                  <!-- Ruang Terbuka Hijau -->
                  <svg v-if="item.iconKey === 'tree'" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M12 21v-6" />
                    <path d="M7 15h10l-3.5-5h2L12 4 8.5 10h2L7 15Z" />
                  </svg>
                  <!-- Taman Bermain -->
                  <svg v-else-if="item.iconKey === 'playground'" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M4 20h16" />
                    <path d="M7 20V8l10 12" />
                    <path d="M7 12h6" />
                    <circle cx="15" cy="7" r="2" />
                  </svg>
                  <!-- Keamanan 24 Jam -->
                  <svg v-else-if="item.iconKey === 'security'" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M12 3 5 6v6c0 4.5 3 7.8 7 9 4-1.2 7-4.5 7-9V6l-7-3Z" />
                    <path d="m9.2 12.2 2 2 3.8-4" />
                  </svg>
                  <!-- Masjid -->
                  <svg v-else-if="item.iconKey === 'mosque'" class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M4 20h16" />
                    <path d="M6 20v-7a6 6 0 0 1 12 0v7" />
                    <path d="M12 7V4" />
                    <path d="M10 20v-4a2 2 0 0 1 4 0v4" />
                  </svg>
                  <!-- Fasilitas Olahraga -->
                  <svg v-else class="w-5 h-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="14" cy="5.5" r="1.8" />
                    <path d="M6 12.5 10.5 10l3 2.5 3.5-1.5" />
                    <path d="m10.5 10-1 5.5-3.5 3" />
                    <path d="m13.5 12.5 2 4.5 2.5 2" />
                  </svg>
                </div>
                <span class="text-[13px] sm:text-sm font-semibold leading-snug text-[var(--ink)]">
                  {{ item.label }}
                </span>
              </div>

              <span class="text-[11px] font-bold tabular-nums text-[var(--muted)]/45 shrink-0" aria-hidden="true">
                {{ String(idx + 1).padStart(2, '0') }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
