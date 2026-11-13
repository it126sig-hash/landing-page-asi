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

let heroObserver = null;
let scrollFallbackActive = false;

const updateScrollFallback = () => {
  const heroEl = document.getElementById('beranda');
  const threshold = heroEl ? Math.max(heroEl.offsetHeight - 80, 120) : 400;
  isScrolled.value = window.scrollY > threshold;
};

onMounted(() => {
  window.addEventListener('keydown', onKeydown);

  const heroEl = document.getElementById('beranda');
  if ('IntersectionObserver' in window && heroEl) {
    heroObserver = new IntersectionObserver(
      (entries) => {
        const entry = entries[0];
        if (entry) {
          isScrolled.value = !entry.isIntersecting;
        }
      },
      {
        rootMargin: '-80px 0px 0px 0px',
        threshold: 0,
      }
    );
    heroObserver.observe(heroEl);
  } else {
    scrollFallbackActive = true;
    updateScrollFallback();
    window.addEventListener('scroll', updateScrollFallback, { passive: true });
  }
});

onUnmounted(() => {
  window.removeEventListener('keydown', onKeydown);
  if (heroObserver) {
    heroObserver.disconnect();
    heroObserver = null;
  }
  if (scrollFallbackActive) {
    window.removeEventListener('scroll', updateScrollFallback);
  }
});
</script>

<template>
  <header
    class="fixed top-0 left-0 right-0 z-50 transition-all duration-300"
    :class="
      isScrolled
        ? 'bg-white text-[#111111] border-b border-black/10 shadow-sm'
        : 'bg-black/15 backdrop-blur-[4px] text-white border-b border-white/25'
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
          class="font-display font-bold text-sm sm:text-base lg:text-lg tracking-[-0.015em] whitespace-nowrap transition-colors duration-300"
          :class="isScrolled ? 'text-[#111111]' : 'text-white'"
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
          class="nav-link relative py-2 text-xs font-semibold tracking-[0.02em] whitespace-nowrap transition-colors duration-200"
          :class="
            isScrolled
              ? 'text-black/80 hover:text-black nav-link--dark'
              : 'text-white/90 hover:text-white nav-link--light'
          "
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
          class="hidden lg:inline-flex items-center justify-center gap-2 rounded-full px-5 py-2.5 text-xs font-semibold tracking-[0.02em] whitespace-nowrap transition-all duration-300"
          :class="
            isScrolled
              ? 'bg-[var(--green)] text-white hover:bg-[var(--green-deep)] shadow-sm'
              : 'bg-white text-[#111111] hover:bg-white/90 shadow-sm'
          "
        >
          {{ ctaItem.label }}
        </a>

        <button
          type="button"
          class="lg:hidden grid place-items-center w-10 h-10 rounded-full border transition-colors duration-300 cursor-pointer"
          :class="
            isScrolled
              ? 'border-black/15 text-[#111111] hover:bg-black/5'
              : 'border-white/30 text-white hover:bg-white/10'
          "
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
      class="lg:hidden bg-white text-[#111111] border-t border-black/10 shadow-lg"
      aria-label="Navigasi mobile"
    >
      <div class="container py-4 flex flex-col gap-1">
        <a
          v-for="item in navItems"
          :key="item.id"
          :href="`#${item.id}`"
          class="py-3 px-2 text-sm font-semibold text-[#111111] hover:text-[var(--green)] border-b border-black/5 last:border-b-0 transition-colors"
          @click="closeMenu"
        >
          {{ item.label }}
        </a>

        <a
          v-if="ctaItem"
          :href="wa('konsultasi')"
          target="_blank"
          rel="noopener noreferrer"
          class="mt-3 rounded-full px-5 py-3 text-center text-sm font-semibold bg-[var(--green)] text-white hover:bg-[var(--green-deep)] transition-colors"
          @click="closeMenu"
        >
          {{ ctaItem.label }}
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
  height: 1.5px;
  width: 0;
  transition: width 0.25s ease, background-color 0.25s ease;
}

.nav-link--light::after {
  background-color: #ffffff;
}

.nav-link--dark::after {
  background-color: #111111;
}

.nav-link:hover::after {
  width: 100%;
}
</style>
