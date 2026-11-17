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
      gsap.to(houseRef.value, { y: -52, scale: 1.02, ease: 'none', scrollTrigger });
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
    class="relative flex h-[100dvh] min-h-[580px] sm:min-h-[640px] w-full items-start sm:items-center overflow-hidden bg-[linear-gradient(to_bottom,#CBE3F8_0%,#E6F2FC_62%,#FFFFFF_100%)] pt-[82px] pb-[180px] sm:pt-[96px] sm:pb-[140px] lg:pt-[88px] lg:pb-[96px]"
  >
    <!-- Layer 1 — Langit biru cerah berawan tanpa scrim gelap -->
    <div
      ref="skyRef"
      class="pointer-events-none absolute inset-x-0 -top-[16%] z-0 h-[132%] select-none will-change-transform"
      aria-hidden="true"
    >
      <div
        class="absolute inset-0 isolate overflow-hidden [mask-image:linear-gradient(to_bottom,#000_48%,transparent_88%)] [-webkit-mask-image:linear-gradient(to_bottom,#000_48%,transparent_88%)]"
      >
        <img
          :src="content.hero.bgImage"
          alt=""
          loading="eager"
          fetchpriority="high"
          decoding="async"
          class="h-full w-full object-cover object-[58%_45%] opacity-65 [filter:grayscale(1)_brightness(1.5)_contrast(0.78)]"
        />
        <div class="absolute inset-0 bg-[#9FD0F2] mix-blend-color" />
        <div class="absolute inset-0 bg-white/40" />
      </div>
    </div>

    <!-- Glow matahari lembut kanan-atas -->
    <div
      class="pointer-events-none absolute inset-0 z-[5] bg-[radial-gradient(ellipse_56%_50%_at_76%_18%,rgba(255,244,218,0.55),rgba(255,244,218,0.18)_48%,rgba(255,244,218,0)_74%)]"
      aria-hidden="true"
    />

    <!-- Haze lembut kiri agar teks gelap selalu kontras & jelas -->
    <div
      class="pointer-events-none absolute inset-y-0 left-0 z-10 w-full bg-gradient-to-r from-[#F4F8FC]/85 via-[#E6F2FC]/45 to-transparent lg:w-[68%] lg:from-[#F4F8FC]/80 lg:via-[#E6F2FC]/40 lg:to-transparent"
      aria-hidden="true"
    />

    <!-- Layer 2 — Visual deretan rumah kanan-bawah -->
    <div
      ref="houseRef"
      class="pointer-events-none absolute -bottom-6 -left-[8%] z-10 w-[126vw] select-none will-change-transform sm:-bottom-12 sm:left-[12%] sm:w-[104vw] lg:-bottom-14 lg:left-[35%] lg:w-[min(140vh,72vw)] xl:left-[36%] xl:w-[min(138vh,70vw)]"
      aria-hidden="true"
    >
      <img
        ref="houseInnerRef"
        :src="content.hero.image"
        alt=""
        loading="eager"
        fetchpriority="high"
        decoding="async"
        class="h-auto w-full sm:[-webkit-mask-image:linear-gradient(to_right,transparent_0%,black_24%)] sm:[mask-image:linear-gradient(to_right,transparent_0%,black_24%)] [filter:brightness(1.03)_saturate(1.03)]"
      />
    </div>

    <!-- Kurva gelombang putih halus di bagian bawah Hero -->
    <div
      class="pointer-events-none absolute inset-x-0 bottom-0 z-20 overflow-hidden leading-none"
      aria-hidden="true"
    >
      <svg
        class="block h-[38px] w-full sm:h-[58px] lg:h-[78px]"
        viewBox="0 0 1440 96"
        fill="none"
        preserveAspectRatio="none"
      >
        <path
          d="M0,44 C360,94 980,78 1440,18 L1440,96 L0,96 Z"
          fill="#ffffff"
        />
      </svg>
    </div>

    <!-- Layer 3 — Kolom copy kiri -->
    <div class="relative z-30 w-full">
      <div class="container">
        <div ref="contentRef" class="max-w-[36rem]">
          <h1
            ref="headlineRef"
            class="font-sans font-bold leading-[1.06] tracking-[-0.025em] text-[var(--ink)] text-[clamp(1.95rem,7.8vw,2.5rem)] sm:text-[clamp(2.55rem,4.1vw,3.65rem)]"
          >
            <span
              v-for="(line, idx) in content.hero.headlineLines"
              :key="idx"
              class="block"
            >
              {{ line }}
            </span>
          </h1>

          <p
            v-if="content.hero.subline"
            ref="sublineRef"
            class="mt-3 sm:mt-4 max-w-[29rem] text-[13.5px] sm:text-[16px] leading-[1.55] text-[var(--muted)]"
          >
            {{ content.hero.subline }}
          </p>

          <div ref="ctaRef" class="mt-4 sm:mt-6">
            <a
              href="#tipe"
              class="group inline-flex items-center gap-3 rounded-full bg-[var(--green)] px-6 py-3 sm:px-7 sm:py-3.5 text-[13.5px] sm:text-[15px] font-semibold text-white shadow-[0_12px_28px_rgba(43,108,196,0.28)] transition-all duration-300 hover:-translate-y-0.5 hover:bg-[var(--green-deep)] hover:shadow-[0_16px_32px_rgba(23,60,114,0.32)]"
            >
              <span>{{ content.hero.primaryCta }}</span>
              <svg
                class="h-4 w-4 shrink-0 text-white transition-transform duration-300 group-hover:translate-x-1"
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

          <!-- Blok Mitra Pembiayaan KPR Resmi (Logo + Nama Bank) -->
          <div
            v-if="content.hero.bankInfo && content.banks?.length"
            ref="bankRef"
            class="mt-4 sm:mt-6 max-w-[28rem] border-t border-[var(--ink)]/12 pt-3 sm:pt-4"
          >
            <p class="text-[10px] sm:text-[11px] font-semibold uppercase tracking-[0.16em] text-[var(--muted)]">
              {{ content.hero.bankInfo.label }}
            </p>

            <div class="mt-2 sm:mt-2.5 flex flex-wrap items-center gap-2 sm:gap-3">
              <div
                v-for="bank in content.banks"
                :key="bank.name"
                class="inline-flex items-center gap-2 rounded-xl bg-white/85 backdrop-blur-sm border border-[var(--line)] px-3 py-1.5 shadow-sm"
              >
                <img
                  :src="bank.image"
                  :alt="`Logo Bank ${bank.name}`"
                  loading="eager"
                  decoding="async"
                  class="h-4 sm:h-5 w-auto shrink-0 object-contain"
                />
                <span class="text-[11px] sm:text-xs font-bold text-[var(--ink)] whitespace-nowrap">
                  Bank {{ bank.name }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Scroll hint di kiri bawah (desktop/tablet) -->
    <div
      v-if="content.hero.scrollLabel"
      class="pointer-events-none absolute inset-x-0 bottom-8 sm:bottom-10 lg:bottom-11 z-30"
    >
      <div class="container">
        <a
          ref="scrollHintRef"
          href="#tentang"
          class="pointer-events-auto hidden sm:inline-flex items-center gap-2.5 text-xs font-medium text-[var(--muted)] transition-colors hover:text-[var(--ink)]"
        >
          <svg
            class="h-5 w-4 shrink-0 text-[var(--muted)]"
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
