<script setup>
import { useContent } from '~/composables/useContent';

const { content } = useContent();
</script>

<template>
  <section id="lokasi" class="bg-white py-20 lg:py-28 scroll-mt-20">
    <div class="container">
      <div class="grid grid-cols-1 lg:grid-cols-[0.42fr_0.58fr] gap-10 lg:gap-14 items-center">
        <!-- Copy Kiri -->
        <div class="reveal">
          <span v-if="content.locationSection.eyebrow" class="eyebrow mb-4">
            {{ content.locationSection.eyebrow }}
          </span>
          <h2
            class="font-sans text-3xl sm:text-4xl font-bold tracking-[-0.02em] leading-[1.18] text-[var(--ink)] [&_em]:font-serif [&_em]:italic [&_em]:font-normal [&_em]:text-[var(--green)]"
            v-html="content.locationSection.titleHtml || content.locationSection.title"
          />
          <p class="mt-4 text-sm sm:text-[15px] leading-[1.7] text-[var(--muted)] max-w-[440px]">
            {{ content.locationSection.desc }}
          </p>

          <div class="mt-5 flex items-start gap-2.5 text-sm font-medium text-[var(--ink)]">
            <span
              class="mt-0.5 grid h-6 w-6 shrink-0 place-items-center rounded-full bg-[var(--sage-pale)] text-[var(--green)]"
              aria-hidden="true"
            >
              <svg class="h-3.5 w-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M12 21s-6.5-5.8-6.5-11a6.5 6.5 0 1 1 13 0c0 5.2-6.5 11-6.5 11Z" />
                <circle cx="12" cy="10" r="2.2" />
              </svg>
            </span>
            <span>{{ content.locationSection.address }}</span>
          </div>

          <div class="mt-7 flex flex-wrap items-center gap-4">
            <a
              :href="content.locationSection.mapUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="btn btn-outline"
            >
              <span>{{ content.locationSection.primaryCta }}</span>
              <span class="arrow" aria-hidden="true">→</span>
            </a>
            <a
              href="#kontak"
              class="inline-flex items-center gap-1.5 px-2 py-2 text-xs sm:text-sm font-semibold text-[var(--green)] hover:text-[var(--green-deep)] transition-colors"
            >
              <span>{{ content.locationSection.secondaryCta }}</span>
              <span aria-hidden="true">→</span>
            </a>
          </div>
        </div>

        <!-- Peta Kanan -->
        <div class="relative rounded-2xl overflow-hidden shadow-[var(--shadow)] border border-[var(--line)] min-h-[340px] lg:min-h-[380px] bg-[var(--sage-pale)] reveal">
          <iframe
            :src="content.locationSection.embedMapUrl"
            class="w-full h-[340px] lg:h-[380px] border-0"
            loading="lazy"
            referrerpolicy="no-referrer-when-downgrade"
            :title="`Peta Lokasi ${content.meta.projectName}`"
          />

          <div class="pointer-events-none absolute left-4 bottom-4 px-3.5 py-2.5 rounded-xl bg-white/95 backdrop-blur-sm border border-[var(--line)] shadow-sm">
            <span class="block text-xs font-bold tracking-tight text-[var(--ink)]">
              {{ content.meta.projectName }}
            </span>
            <span class="block mt-0.5 text-[11px] font-medium text-[var(--muted)]">
              {{ content.locationSection.address }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
