<script setup>
import { ref } from 'vue';
import { useContent } from '~/composables/useContent';

const { content, wa } = useContent();

// Item pertama 'f1' default terbuka sesuai arahan editorial
const openFaqId = ref('f1');

function toggle(id) {
  openFaqId.value = openFaqId.value === id ? null : id;
}

const consultasiLink = wa('konsultasi');
</script>

<template>
  <section id="faq" class="scroll-mt-20 relative bg-white py-20 lg:py-28">
    <div class="container faq-grid grid grid-cols-1 lg:grid-cols-[0.72fr_1.28fr] gap-10 lg:gap-[88px] items-start">
      <!-- Kiri: Intro Sticky -->
      <div class="faq-intro reveal lg:sticky lg:top-28">
        <div class="mb-4 h-0.5 w-10 rounded-full bg-[var(--green)]" aria-hidden="true" />
        <h2
          class="font-sans font-bold text-3xl sm:text-4xl lg:text-[42px] tracking-[-0.02em] leading-[1.18] text-[var(--ink)] mb-4 [&_em]:not-italic [&_em]:font-bold [&_em]:text-[var(--green)]"
          v-html="content.faqSection.titleHtml || content.faqSection.title"
        />
        <p class="text-sm sm:text-[15px] text-[var(--muted)] leading-relaxed mb-7 max-w-[340px]">
          {{ content.faqSection.subtitle }}
        </p>
        <a
          :href="consultasiLink"
          target="_blank"
          rel="noopener noreferrer"
          class="btn btn-outline"
        >
          <span>{{ content.faqSection.ctaLabel }}</span>
          <span class="arrow" aria-hidden="true">→</span>
        </a>
      </div>

      <!-- Kanan: Accordion List -->
      <div class="faq-list border-t border-[var(--line)] reveal">
        <FaqItem
          v-for="faq in content.faqs"
          :key="faq.id"
          :faq="faq"
          :open="openFaqId === faq.id"
          @toggle="toggle"
        />
      </div>
    </div>
  </section>
</template>
