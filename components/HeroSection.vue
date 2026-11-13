<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useContent } from '~/composables/useContent';

const { content } = useContent();

const bankHeightClass = {
  BCA: 'h-5 sm:h-6',
  BTN: 'h-4 sm:h-5',
};

const heroRef = ref(null);
const bgParallaxRef = ref(null);
const titleParallaxRef = ref(null);
const titleInnerRef = ref(null);
const houseParallaxRef = ref(null);
const houseInnerRef = ref(null);
const uiCopyRef = ref(null);
const uiBankRef = ref(null);

let ctx = null;

onMounted(() => {
  if (!heroRef.value) return;
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

  gsap.registerPlugin(ScrollTrigger);

  ctx = gsap.context(() => {
    if (titleInnerRef.value) {
      gsap.from(titleInnerRef.value, {
        y: 44,
        opacity: 0,
        duration: 1.0,
        delay: 0.2,
        ease: 'power3.out',
        clearProps: 'transform',
      });
    }

    if (houseInnerRef.value) {
      gsap.from(houseInnerRef.value, {
        y: 80,
        opacity: 0,
        scale: 1.04,
        duration: 1.15,
        delay: 0.35,
        ease: 'power3.out',
        clearProps: 'transform',
      });
    }

    const uiTargets = [uiCopyRef.value, uiBankRef.value].filter(Boolean);
    if (uiTargets.length) {
      gsap.from(uiTargets, {
        y: 28,
        opacity: 0,
        duration: 0.85,
        stagger: 0.12,
        delay: 0.55,
        ease: 'power2.out',
        clearProps: 'transform',
      });
    }

    const scrollTrigger = {
      trigger: heroRef.value,
      start: 'top top',
      end: 'bottom top',
      scrub: 0.6,
    };

    if (bgParallaxRef.value) {
      gsap.to(bgParallaxRef.value, {
        yPercent: 12,
        ease: 'none',
        scrollTrigger,
      });
    }

    if (titleParallaxRef.value) {
      gsap.to(titleParallaxRef.value, {
        y: 165,
        ease: 'none',
        scrollTrigger,
      });
    }

    if (houseParallaxRef.value) {
      gsap.to(houseParallaxRef.value, {
        y: -48,
        ease: 'none',
        scrollTrigger,
      });
    }
  }, heroRef.value);
});

onUnmounted(() => {
  ctx?.revert();
});
</script>

<template>
  <section
    id="beranda"
    ref="heroRef"
    class="relative min-h-[100dvh] w-full overflow-hidden flex flex-col justify-end bg-[#1d281f]"
  >
    <!-- Layer 1: Background Photo (with top/bottom bleed for parallax) + Soft Top/Bottom Scrim -->
    <div
      ref="bgParallaxRef"
      data-parallax="bg"
      class="pointer-events-none select-none absolute inset-x-0 -top-[16%] h-[118%] z-0 will-change-transform"
      aria-hidden="true"
    >
      <img
        :src="content.hero.bgImage"
        alt=""
        loading="eager"
        decoding="async"
        fetchpriority="high"
        class="w-full h-full object-cover object-center"
      />
    </div>
    <div
      class="pointer-events-none absolute inset-x-0 top-0 z-[5] h-40 bg-gradient-to-b from-black/50 via-black/20 to-transparent"
      aria-hidden="true"
    />
    <div
      class="pointer-events-none absolute inset-0 z-[5] bg-gradient-to-t from-black/40 via-transparent to-transparent"
      aria-hidden="true"
    />

    <!-- Layer 2: Giant Sky Typography (outer parallax wrapper + inner entrance element) -->
    <div
      ref="titleParallaxRef"
      data-parallax="title"
      class="pointer-events-none select-none absolute inset-x-0 top-[14dvh] sm:top-[18dvh] lg:top-[20dvh] z-10 flex justify-center px-3 sm:px-6 will-change-transform"
      aria-hidden="true"
    >
      <p
        ref="titleInnerRef"
        class="font-display font-bold uppercase text-center text-white/95 leading-[0.9] md:leading-[0.94] tracking-[-0.025em] drop-shadow-[0_4px_24px_rgba(0,0,0,0.35)] text-[clamp(3rem,13vw,5.25rem)] md:text-[clamp(3.2rem,9.2vw,9.25rem)] md:whitespace-nowrap max-w-full"
      >
        {{ content.hero.giantTitle }}
      </p>
    </div>

    <!-- Layer 3: Foreground Houses (outer parallax wrapper with bottom bleed + inner entrance image) -->
    <div
      ref="houseParallaxRef"
      data-parallax="house"
      class="pointer-events-none select-none absolute inset-x-0 -bottom-16 z-20 h-[calc(68dvh+4rem)] sm:h-[calc(78dvh+4rem)] lg:h-[calc(90dvh+4rem)] will-change-transform"
    >
      <img
        ref="houseInnerRef"
        :src="content.hero.image"
        :alt="content.meta.projectName"
        loading="eager"
        decoding="async"
        class="w-full h-full object-cover object-[28%_bottom] lg:object-bottom"
      />
    </div>

    <!-- Bottom Scrim above foreground ground for crisp UI legibility -->
    <div
      class="pointer-events-none absolute inset-x-0 bottom-0 z-[25] h-[58%] sm:h-[52%] bg-gradient-to-t from-black/80 via-black/40 to-transparent"
      aria-hidden="true"
    />

    <!-- Layer 4: Overlay UI (Bottom-Left Headline + CTA, Bottom-Right Bank Info; Top-Left stays empty) -->
    <div class="relative z-30 w-full pt-24 pb-24 sm:pb-10 lg:pb-12">
      <div class="container flex flex-col gap-5 lg:flex-row lg:items-end lg:justify-between lg:gap-8">
        <!-- Bottom-Left: Stacked Headline + Architectural Split CTA -->
        <div ref="uiCopyRef" class="max-w-xl">
          <h1
            class="font-display font-bold uppercase text-white text-[clamp(1.75rem,3.6vw,3.15rem)] leading-[1.03] tracking-[-0.015em] drop-shadow-[0_2px_16px_rgba(0,0,0,0.45)]"
          >
            <span
              v-for="(line, idx) in content.hero.headlineLines"
              :key="idx"
              class="block"
            >
              {{ line }}
            </span>
          </h1>

          <div class="mt-4 sm:mt-5">
            <a
              href="#tipe"
              class="group inline-flex items-stretch border border-white/40 bg-white text-[#1a1a1a] hover:bg-white/90 shadow-[0_12px_32px_rgba(0,0,0,0.3)] transition-all duration-300 rounded-sm overflow-hidden"
            >
              <span class="px-5 py-3 sm:py-3.5 text-[11px] sm:text-xs font-semibold tracking-[0.08em] uppercase flex items-center">
                {{ content.hero.primaryCta }}
              </span>
              <span
                class="px-4 py-3 sm:py-3.5 border-l border-black/15 bg-black/5 flex items-center justify-center text-base font-semibold transition-transform duration-300 group-hover:translate-x-0.5"
                aria-hidden="true"
              >
                &raquo;
              </span>
            </a>
          </div>
        </div>

        <!-- Bottom-Right: KPR Partner Bank Info Panel -->
        <div
          v-if="content.hero.bankInfo && content.banks?.length"
          ref="uiBankRef"
          class="w-full sm:max-w-md lg:max-w-[360px] bg-black/45 backdrop-blur-md border border-white/15 rounded-xl p-4 sm:p-5 text-white shadow-[0_16px_40px_rgba(0,0,0,0.28)]"
        >
          <p class="text-[10px] sm:text-[11px] font-semibold uppercase tracking-[0.14em] text-white/80">
            {{ content.hero.bankInfo.label }}
          </p>

          <div class="mt-3 flex flex-wrap items-center gap-5">
            <img
              v-for="bank in content.banks"
              :key="bank.name"
              :src="bank.imageWhite"
              :alt="`Bank ${bank.name}`"
              loading="eager"
              decoding="async"
              class="w-auto object-contain"
              :class="bankHeightClass[bank.name] || 'h-5'"
            />
          </div>

          <p class="mt-3 text-[11px] sm:text-xs text-white/90 leading-relaxed">
            {{ content.hero.bankInfo.desc }}
          </p>
        </div>
      </div>
    </div>
  </section>
</template>
