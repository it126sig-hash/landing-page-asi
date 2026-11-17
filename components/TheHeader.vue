<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';
import { useContent } from '~/composables/useContent';

const { content, wa } = useContent();

const isMenuOpen = ref(false);
const isScrolled = ref(false);

const navItems = computed(() =>
  (content.meta?.nav || []).filter((item) => !item.isCta)
);
const ctaItem = computed(() =>
  (content.meta?.nav || []).find((item) => item.isCta)
);

const closeMenu = () => {
  isMenuOpen.value = false;
};

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

const onKeydown = (e) => {
  if (e.key === 'Escape' && isMenuOpen.value) {
    closeMenu();
  }
};

const updateScrollState = () => {
  isScrolled.value = window.scrollY > 24;
};

onMounted(() => {
  window.addEventListener('keydown', onKeydown);
  updateScrollState();
  window.addEventListener('scroll', updateScrollState, { passive: true });
});

onUnmounted(() => {
  window.removeEventListener('keydown', onKeydown);
  window.removeEventListener('scroll', updateScrollState);
});
</script>

<template>
  <header
    class="fixed top-0 left-0 right-0 z-50 transition-all duration-300"
    :class="
      isScrolled
        ? 'bg-white/95 backdrop-blur-md text-[var(--ink)] border-b border-[var(--line)] shadow-sm'
        : 'bg-transparent text-[var(--ink)] border-b border-transparent'
    "
  >
    <div
      class="container h-[68px] lg:h-[76px] flex items-center justify-between lg:grid lg:grid-cols-[1fr_auto_1fr] gap-4"
    >
      <!-- Left: Logo + Project Name -->
      <a
        href="#beranda"
        class="inline-flex items-center gap-2.5 sm:gap-3 min-w-0 justify-self-start"
        :aria-label="`${content.meta.projectName} — Beranda`"
        @click="closeMenu"
      >
        <img
          :src="content.meta.logo"
          :alt="`Logo ${content.meta.projectName}`"
          class="h-9 sm:h-10 w-auto object-contain shrink-0 rounded"
        />
        <span
          class="font-display font-bold text-sm sm:text-base lg:text-lg tracking-[-0.015em] text-[var(--ink)] whitespace-nowrap"
        >
          {{ content.meta.projectName }}
        </span>
      </a>

      <!-- Center: Desktop Navigation -->
      <nav
        class="hidden lg:flex items-center justify-center gap-7 justify-self-center"
        aria-label="Navigasi utama"
      >
        <a
          v-for="item in navItems"
          :key="item.id"
          :href="`#${item.id}`"
          class="nav-link relative py-2 text-[13px] font-semibold tracking-[0.01em] text-[var(--ink)]/85 hover:text-[var(--green)] whitespace-nowrap transition-colors duration-200"
        >
          {{ item.label }}
        </a>
      </nav>

      <!-- Right: Desktop CTA + Mobile/Tablet Hamburger -->
      <div class="flex items-center justify-end gap-2 justify-self-end">
        <a
          v-if="ctaItem"
          :href="wa('konsultasi')"
          target="_blank"
          rel="noopener noreferrer"
          class="group hidden lg:inline-flex items-center justify-center gap-2 rounded-full bg-[var(--green)] px-5 py-2.5 text-xs font-semibold tracking-[0.01em] text-white shadow-[0_6px_18px_rgba(43,108,196,0.22)] whitespace-nowrap transition-all duration-300 hover:bg-[var(--green-deep)] hover:shadow-[0_8px_22px_rgba(23,60,114,0.28)]"
        >
          <span>{{ ctaItem.label }}</span>
          <svg
            class="h-3.5 w-3.5 shrink-0 transition-transform duration-300 group-hover:translate-x-0.5"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2.25"
            stroke-linecap="round"
            stroke-linejoin="round"
            aria-hidden="true"
          >
            <path d="M5 12h14" />
            <path d="m13 6 6 6-6 6" />
          </svg>
        </a>

        <button
          type="button"
          class="lg:hidden grid place-items-center w-10 h-10 rounded-full border border-[var(--ink)]/15 bg-white/60 backdrop-blur-sm text-[var(--ink)] hover:bg-white transition-colors duration-200 cursor-pointer"
          :aria-expanded="isMenuOpen"
          aria-controls="mobile-nav-menu"
          aria-label="Toggle navigasi menu"
          @click="toggleMenu"
        >
          <span class="text-lg leading-none" aria-hidden="true">{{ isMenuOpen ? '✕' : '☰' }}</span>
        </button>
      </div>
    </div>

    <!-- Mobile / Tablet Dropdown Menu (always white background for guaranteed legibility) -->
    <nav
      v-show="isMenuOpen"
      id="mobile-nav-menu"
      class="lg:hidden bg-white text-[var(--ink)] border-t border-[var(--line)] shadow-lg"
      aria-label="Navigasi mobile"
    >
      <div class="container py-4 flex flex-col gap-1">
        <a
          v-for="item in navItems"
          :key="item.id"
          :href="`#${item.id}`"
          class="py-3 px-2 text-sm font-semibold text-[var(--ink)] hover:text-[var(--green)] border-b border-[var(--line)] last:border-b-0 transition-colors"
          @click="closeMenu"
        >
          {{ item.label }}
        </a>

        <a
          v-if="ctaItem"
          :href="wa('konsultasi')"
          target="_blank"
          rel="noopener noreferrer"
          class="mt-3 inline-flex items-center justify-center gap-2 rounded-full px-5 py-3 text-center text-sm font-semibold bg-[var(--green)] text-white hover:bg-[var(--green-deep)] transition-colors"
          @click="closeMenu"
        >
          <span>{{ ctaItem.label }}</span>
          <span aria-hidden="true">→</span>
        </a>
      </div>
    </nav>
  </header>
</template>

<style scoped>
.nav-link::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: 4px;
  height: 2px;
  width: 0;
  border-radius: 999px;
  background-color: var(--green);
  transition: width 0.25s ease;
}

.nav-link:hover::after {
  width: 100%;
}
</style>
