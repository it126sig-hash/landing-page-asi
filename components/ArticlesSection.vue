<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';
import { useContent } from '~/composables/useContent';

const { content, wa } = useContent();

const articles = computed(() => content.articlesSection?.items || []);
const activeArticle = ref(null);

function openArticle(article) {
  if (!article) return;
  activeArticle.value = article;
}

function openFirstArticle() {
  if (articles.value.length > 0) {
    openArticle(articles.value[0]);
  }
}

function closeArticle() {
  activeArticle.value = null;
}

function onKeydown(e) {
  if (e.key === 'Escape' && activeArticle.value) {
    closeArticle();
  }
}

watch(activeArticle, (val) => {
  if (typeof document === 'undefined') return;
  document.body.style.overflow = val ? 'hidden' : '';
});

onMounted(() => {
  window.addEventListener('keydown', onKeydown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', onKeydown);
  if (typeof document !== 'undefined') {
    document.body.style.overflow = '';
  }
});
</script>

<template>
  <section
    v-if="content.articlesSection"
    id="artikel"
    class="bg-[var(--sage-pale)] py-20 lg:py-28 scroll-mt-20"
  >
    <div class="container">
      <!-- Header Row -->
      <div class="flex flex-col sm:flex-row sm:items-end justify-between gap-4 mb-8 lg:mb-10 reveal">
        <div>
          <div class="mb-4 h-0.5 w-10 rounded-full bg-[var(--green)]" aria-hidden="true" />
          <h2 class="font-sans text-3xl sm:text-4xl font-bold tracking-[-0.02em] leading-[1.18] text-[var(--ink)]">
            {{ content.articlesSection.title }}
          </h2>
          <p class="mt-2.5 text-sm sm:text-[15px] leading-relaxed text-[var(--muted)] max-w-[540px]">
            {{ content.articlesSection.subtitle }}
          </p>
        </div>

        <button
          v-if="articles.length"
          type="button"
          class="btn btn-outline shrink-0 self-start sm:self-auto"
          @click="openFirstArticle"
        >
          <span>{{ content.articlesSection.ctaLabel }}</span>
          <span class="arrow" aria-hidden="true">→</span>
        </button>
      </div>

      <!-- Grid 3 Kartu Artikel -->
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <article
          v-for="article in articles"
          :key="article.id"
          role="button"
          tabindex="0"
          :aria-label="`Baca artikel ${article.title}`"
          class="group rounded-2xl bg-white border border-[var(--line)] shadow-sm overflow-hidden flex flex-col cursor-pointer hover:-translate-y-1 hover:shadow-[var(--shadow)] transition-all duration-300 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[var(--green)] reveal"
          @click="openArticle(article)"
          @keydown.enter.prevent="openArticle(article)"
          @keydown.space.prevent="openArticle(article)"
        >
          <div class="aspect-[16/10] w-full overflow-hidden bg-[var(--paper-deep)]">
            <img
              :src="article.image"
              :alt="article.title"
              loading="lazy"
              class="h-full w-full object-cover transition-transform duration-500 group-hover:scale-105"
            />
          </div>

          <div class="p-5 flex-1 flex flex-col justify-between">
            <div>
              <div class="flex items-center gap-1.5 text-xs font-medium text-[var(--muted)] mb-2.5">
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
                  <rect x="3" y="4" width="18" height="18" rx="2" />
                  <path d="M16 2v4M8 2v4M3 10h18" />
                </svg>
                <time>{{ article.date }}</time>
              </div>

              <h3 class="font-sans text-base sm:text-[17px] font-bold leading-snug text-[var(--ink)] line-clamp-2 group-hover:text-[var(--green)] transition-colors">
                {{ article.title }}
              </h3>

              <p class="mt-2 text-xs sm:text-[13px] leading-relaxed text-[var(--muted)] line-clamp-2">
                {{ article.excerpt }}
              </p>
            </div>

            <div class="mt-4 flex justify-end">
              <button
                type="button"
                class="w-9 h-9 rounded-full bg-[var(--sage-pale)] text-[var(--green)] grid place-items-center group-hover:bg-[var(--green)] group-hover:text-white transition-colors duration-200 cursor-pointer"
                :aria-label="`Baca selengkapnya: ${article.title}`"
                @click.stop="openArticle(article)"
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
    </div>

    <!-- Modal Pratinjau Artikel -->
    <Teleport to="body">
      <div
        v-if="activeArticle"
        class="fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6 bg-[#17253b]/60 backdrop-blur-sm"
        role="dialog"
        aria-modal="true"
        :aria-label="activeArticle.title"
        @click.self="closeArticle"
      >
        <div class="relative w-full max-w-2xl max-h-[90vh] overflow-y-auto rounded-3xl bg-white shadow-2xl border border-[var(--line)]">
          <button
            type="button"
            class="absolute top-4 right-4 z-10 w-10 h-10 rounded-full bg-white/95 text-[var(--ink)] border border-[var(--line)] shadow-sm grid place-items-center hover:bg-[var(--sage-pale)] hover:text-[var(--green-deep)] transition-colors cursor-pointer"
            aria-label="Tutup artikel"
            @click="closeArticle"
          >
            <span class="text-base font-bold leading-none" aria-hidden="true">✕</span>
          </button>

          <div class="aspect-[16/9] w-full overflow-hidden bg-[var(--paper-deep)]">
            <img
              :src="activeArticle.image"
              :alt="activeArticle.title"
              class="h-full w-full object-cover"
            />
          </div>

          <div class="p-6 sm:p-8">
            <div class="inline-flex items-center gap-1.5 rounded-full bg-[var(--sage-pale)] px-3 py-1 text-xs font-semibold text-[var(--green-deep)]">
              <svg
                class="w-3.5 h-3.5 shrink-0"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
                aria-hidden="true"
              >
                <rect x="3" y="4" width="18" height="18" rx="2" />
                <path d="M16 2v4M8 2v4M3 10h18" />
              </svg>
              <time>{{ activeArticle.date }}</time>
            </div>

            <h3 class="mt-4 font-sans text-xl sm:text-2xl font-bold leading-snug text-[var(--ink)]">
              {{ activeArticle.title }}
            </h3>

            <p class="mt-3 text-sm font-medium leading-relaxed text-[var(--ink)]/85">
              {{ activeArticle.excerpt }}
            </p>

            <p class="mt-4 text-sm leading-[1.75] text-[var(--muted)]">
              {{ activeArticle.content }}
            </p>

            <div class="mt-7 pt-5 border-t border-[var(--line)] flex flex-wrap items-center justify-between gap-3">
              <a
                :href="wa('konsultasi')"
                target="_blank"
                rel="noopener noreferrer"
                class="btn btn-primary"
              >
                <span>{{ content.faqSection?.ctaLabel || 'Konsultasi Tim Kami' }}</span>
                <span aria-hidden="true">→</span>
              </a>

              <button
                type="button"
                class="btn btn-outline"
                @click="closeArticle"
              >
                Tutup
              </button>
            </div>
          </div>
        </div>
      </div>
    </Teleport>
  </section>
</template>
