<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useContent } from '~/composables/useContent';

const { content } = useContent();

const heroRef = ref(null);
const skyRef = ref(null);
const houseRef = ref(null);
const houseInnerRef = ref(null);
const contentRef = ref(null);
const badgeRef = ref(null);
const headlineRef = ref(null);
const sublineRef = ref(null);
const ctaRef = ref(null);
const bankRef = ref(null);
const scrollHintRef = ref(null);

let ctx = null;

onMounted(() => {
  if (!heroRef.value) return;
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

  gsap.registerPlugin(ScrollTrigger);

  ctx = gsap.context(() => {
    const timeline = gsap.timeline({ defaults: { ease: 'power3.out' } });
    const entrance = [
      [houseInnerRef.value, { y: 48, opacity: 0, duration: 1.1 }, 0.08],
      [badgeRef.value, { y: 20, opacity: 0, duration: 0.7 }, 0.1],
      [headlineRef.value, { y: 36, opacity: 0, duration: 0.9 }, 0.16],
      [sublineRef.value, { y: 26, opacity: 0, duration: 0.8 }, 0.28],
      [ctaRef.value, { y: 22, opacity: 0, duration: 0.7 }, 0.38],
      [bankRef.value, { y: 20, opacity: 0, duration: 0.65 }, 0.46],
      [scrollHintRef.value, { y: 16, opacity: 0, duration: 0.6 }, 0.54],
    ];

    entrance.forEach(([target, vars, position]) => {
      if (!target) return;
      timeline.from(target, { ...vars, clearProps: 'transform,opacity' }, position);
    });

    const scrollTrigger = {
      trigger: heroRef.value,
      start: 'top top',
      end: 'bottom top',
      scrub: 0.5,
    };

    if (skyRef.value) {
      gsap.to(skyRef.value, { yPercent: 18, ease: 'none', scrollTrigger });
    }
    if (houseRef.value) {
      gsap.to(houseRef.value, {
        y: -52,
        scale: 1.02,
        transformOrigin: 'bottom center',
        ease: 'none',
        scrollTrigger,
      });
    }
    if (contentRef.value) {
      gsap.to(contentRef.value, { y: 56, ease: 'none', scrollTrigger });
    }
  }, heroRef.value);
});

onUnmounted(() => {
  ctx?.revert();
  ctx = null;
});
</script>

<template>
  <section
    id="beranda"
    ref="heroRef"
    class="relative flex h-[100dvh] min-h-[600px] sm:min-h-[640px] w-full items-center overflow-hidden bg-[linear-gradient(180deg,#5BA4E6_0%,#9CD2F7_58%,#E3F3FD_100%)] pt-[76px] pb-10 sm:pt-[84px] sm:pb-12"
  >
    <!-- Layer 0 — Langit biru cerah siang hari natural -->
    <div
      ref="skyRef"
      class="pointer-events-none absolute inset-x-0 -top-[16%] z-0 h-[132%] select-none will-change-transform"
      aria-hidden="true"
    >
      <div class="relative h-full w-full overflow-hidden">
        <img
          :src="content.hero.bgImage"
          alt=""
          loading="eager"
          fetchpriority="high"
          decoding="async"
          class="h-full w-full object-cover object-[56%_38%] opacity-75 [filter:grayscale(1)_brightness(1.45)_contrast(0.92)]"
        />
        <!-- Cerah & natural daytime sky blue tint tanpa gradien gelap di bawah -->
        <div class="absolute inset-0 bg-[#75BDF2] mix-blend-color opacity-85" />
        <div class="absolute inset-0 bg-[linear-gradient(180deg,rgba(91,164,230,0.22)_0%,rgba(156,210,247,0.14)_60%,rgba(227,243,253,0.12)_100%)]" />
      </div>
    </div>

    <!-- Glow matahari lembut kanan-atas -->
    <div
      class="pointer-events-none absolute inset-0 z-[2] bg-[radial-gradient(ellipse_54%_48%_at_76%_18%,rgba(255,252,240,0.42),rgba(255,252,240,0.12)_50%,transparent_75%)]"
      aria-hidden="true"
    />

    <!-- Layer 1 — Deretan rumah full kiri-ke-kanan layar tanpa fade -->
    <div
      ref="houseRef"
      class="pointer-events-none absolute inset-x-0 -bottom-14 z-[5] w-full select-none will-change-transform origin-bottom"
      aria-hidden="true"
    >
      <img
        ref="houseInnerRef"
        :src="content.hero.image"
        alt=""
        loading="eager"
        fetchpriority="high"
        decoding="async"
        class="h-[58dvh] min-h-[360px] lg:h-auto w-full object-cover object-bottom [filter:brightness(1.08)_saturate(1.08)]"
      />
    </div>

    <!-- Layer 2 — Overlay hitam transparan ringan agar hero tetap cerah & teks kiri kontras -->
    <div
      class="pointer-events-none absolute inset-0 z-10 bg-gradient-to-r from-black/55 via-black/28 to-transparent sm:from-black/50 sm:via-black/25 sm:to-transparent"
      aria-hidden="true"
    />
    <div
      class="pointer-events-none absolute inset-x-0 top-0 z-10 h-28 bg-gradient-to-b from-black/30 to-transparent"
      aria-hidden="true"
    />
    <div
      class="pointer-events-none absolute inset-x-0 bottom-0 z-10 h-32 bg-gradient-to-t from-black/35 to-transparent"
      aria-hidden="true"
    />

    <!-- Layer 3 — Kolom copy kiri -->
    <div class="relative z-20 w-full">
      <div class="container">
        <div ref="contentRef" class="max-w-[38rem]">
          <!-- Badge pill outline -->
          <div ref="badgeRef">
            <span
              class="inline-flex items-center rounded-full border border-white/55 bg-black/15 px-4 py-1.5 text-[10.5px] font-semibold uppercase tracking-[0.2em] text-white backdrop-blur-[2px]"
            >
              {{ content.meta.projectName }}
            </span>
          </div>

          <!-- Headline 3 baris uppercase putih tebal -->
          <h1
            ref="headlineRef"
            class="mt-4 font-sans font-bold uppercase leading-[0.98] tracking-[-0.02em] text-white text-[clamp(2rem,8vw,2.65rem)] sm:text-[clamp(2.65rem,4.2vw,3.85rem)]"
          >
            <span
              v-for="(line, idx) in content.hero.headlineLines"
              :key="idx"
              class="block"
            >
              {{ line }}
            </span>
          </h1>

          <!-- Subline -->
          <p
            v-if="content.hero.subline"
            ref="sublineRef"
            class="mt-3.5 sm:mt-4 max-w-[29rem] text-[13.5px] sm:text-[16px] leading-[1.55] text-white/90"
          >
            {{ content.hero.subline }}
          </p>

          <!-- CTA Utama putih bersih -->
          <div ref="ctaRef" class="mt-5 sm:mt-6">
            <a
              href="#tipe"
              class="group inline-flex items-center gap-3 rounded-xl bg-white text-[var(--ink)] hover:bg-[var(--sage-pale)] px-6 py-3.5 text-[13.5px] sm:text-[15px] font-semibold shadow-lg transition-all duration-300 hover:-translate-y-0.5 cursor-pointer"
            >
              <span>{{ content.hero.primaryCta }}</span>
              <svg
                class="h-4 w-4 shrink-0 text-[var(--ink)] transition-transform duration-300 group-hover:translate-x-1"
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

          <!-- Blok Mitra Pembiayaan KPR (Logo putih tanpa kotak background) -->
          <div
            v-if="content.hero.bankInfo && content.banks?.length"
            ref="bankRef"
            class="mt-6 sm:mt-8 max-w-[26rem] border-t border-white/25 pt-4"
          >
            <p class="text-[10px] sm:text-[11px] font-semibold uppercase tracking-[0.16em] text-white/80">
              {{ content.hero.bankInfo.label }}
            </p>

            <div class="mt-3 flex items-center gap-4 sm:gap-5">
              <template v-for="(bank, idx) in content.banks" :key="bank.name">
                <img
                  :src="bank.imageWhite || bank.image"
                  :alt="`Logo Bank ${bank.name}`"
                  loading="eager"
                  decoding="async"
                  :class="bank.name === 'BCA' ? 'h-6 sm:h-7 w-auto object-contain' : 'h-5 sm:h-6 w-auto object-contain'"
                />
                <span
                  v-if="idx < content.banks.length - 1"
                  class="w-px h-6 bg-white/30 shrink-0"
                  aria-hidden="true"
                />
              </template>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Scroll hint (desktop kanan bawah, tidak bertabrakan dengan logo KPR) -->
    <div
      v-if="content.hero.scrollLabel"
      class="pointer-events-none absolute inset-x-0 bottom-6 z-20 hidden md:block"
    >
      <div class="container flex justify-end">
        <a
          ref="scrollHintRef"
          href="#tentang"
          class="pointer-events-auto inline-flex items-center gap-2 text-xs font-medium text-white/80 transition-colors hover:text-white cursor-pointer"
        >
          <svg
            class="h-5 w-4 shrink-0 text-white/80"
            viewBox="0 0 18 26"
            fill="none"
            stroke="currentColor"
            stroke-width="1.6"
            aria-hidden="true"
          >
            <rect x="1.5" y="1.5" width="15" height="23" rx="7.5" />
            <circle cx="9" cy="8" r="1.5" fill="currentColor" stroke="none" />
          </svg>
          <span>{{ content.hero.scrollLabel }}</span>
        </a>
      </div>
    </div>
  </section>
</template>
