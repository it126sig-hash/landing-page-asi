<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useContent } from '~/composables/useContent';

const { content } = useContent();

// Tinggi logo bank disetel manual supaya bobot visual BCA & BTN terasa seimbang
// (rasio file kedua logo berbeda jauh: BCA 300×106, BTN 300×188 dengan padding).
const bankHeightClass = {
  BCA: 'h-[40px]',
  BTN: 'h-[46px]',
};

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

let ctx = null;

onMounted(() => {
  if (!heroRef.value) return;
  // Hormati preferensi sistem: tanpa animasi, konten tetap tampil apa adanya.
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

  gsap.registerPlugin(ScrollTrigger);

  ctx = gsap.context(() => {
    // --- Entrance bertahap (badge → headline → subline → CTA → blok KPR) ---
    const timeline = gsap.timeline({ defaults: { ease: 'power3.out' } });
    const entrance = [
      [houseInnerRef.value, { y: 40, opacity: 0, duration: 1.1 }, 0.1],
      [badgeRef.value, { y: 14, opacity: 0, duration: 0.6 }, 0.1],
      [headlineRef.value, { y: 34, opacity: 0, duration: 0.9 }, 0.2],
      [sublineRef.value, { y: 24, opacity: 0, duration: 0.8 }, 0.35],
      [ctaRef.value, { y: 20, opacity: 0, duration: 0.7 }, 0.45],
      [bankRef.value, { y: 20, opacity: 0, duration: 0.7 }, 0.55],
    ];

    entrance.forEach(([target, vars, position]) => {
      if (!target) return;
      timeline.from(target, { ...vars, clearProps: 'transform,opacity' }, position);
    });

    // --- Parallax saat hero keluar viewport ---
    const scrollTrigger = {
      trigger: heroRef.value,
      start: 'top top',
      end: 'bottom top',
      scrub: 0.6,
    };

    if (skyRef.value) {
      gsap.to(skyRef.value, { yPercent: 8, ease: 'none', scrollTrigger });
    }
    if (houseRef.value) {
      // Wrapper punya bleed bawah (-bottom-10) sehingga geser naik 26px tidak
      // memperlihatkan celah di tepi bawah section.
      gsap.to(houseRef.value, { y: -26, ease: 'none', scrollTrigger });
    }
    if (contentRef.value) {
      gsap.to(contentRef.value, { y: 30, ease: 'none', scrollTrigger });
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
    class="relative flex min-h-[100dvh] w-full items-center overflow-hidden bg-[#101619] pt-[88px] pb-10 sm:pt-[112px] sm:pb-16"
  >
    <!-- Layer 1 — Langit senja (diberi bleed atas/bawah untuk parallax) -->
    <div
      ref="skyRef"
      class="pointer-events-none absolute inset-x-0 -top-[10%] z-0 h-[120%] select-none will-change-transform"
      aria-hidden="true"
    >
      <img
        :src="content.hero.bgImage"
        alt=""
        loading="eager"
        fetchpriority="high"
        decoding="async"
        class="h-full w-full object-cover object-center [filter:saturate(1.08)_sepia(0.10)_contrast(1.05)_brightness(0.96)]"
      />
    </div>

    <!-- Layer 2 — Deretan rumah (cutout alfa, tanpa blend mode) -->
    <div
      ref="houseRef"
      class="pointer-events-none absolute -bottom-10 z-10 w-[max(92vh,140vw)] -left-[16%] select-none will-change-transform sm:left-[2%] sm:w-[max(88vh,112vw)] lg:left-[30%] lg:w-[152vh] lg:min-w-[70vw]"
      aria-hidden="true"
    >
      <img
        ref="houseInnerRef"
        :src="content.hero.image"
        alt=""
        loading="eager"
        fetchpriority="high"
        decoding="async"
        class="h-auto w-full [-webkit-mask-image:linear-gradient(to_right,transparent_0%,black_22%)] [filter:brightness(0.80)_sepia(0.20)_saturate(1.06)_contrast(1.03)] [mask-image:linear-gradient(to_right,transparent_0%,black_22%)]"
      />
    </div>

    <!-- Layer 3 — Scrim: atas (header), kiri (kolom copy), bawah (menyatukan ground) -->
    <div
      class="pointer-events-none absolute inset-x-0 top-0 z-20 h-36 bg-gradient-to-b from-black/55 via-black/25 to-transparent"
      aria-hidden="true"
    />
    <!-- Scrim kiri: base & lg seperti semula; pada sm–md gradien ditahan pekat
         sampai 70% lebar (kolom copy di tablet memanjang sampai ~66%) lalu turun
         cepat ke 10% supaya sisi kanan (rumah/langit) tetap terbaca. -->
    <div
      class="pointer-events-none absolute inset-y-0 left-0 z-20 w-full bg-gradient-to-r from-black/85 via-black/60 to-black/45 sm:via-black/50 sm:via-[70%] sm:to-black/10 lg:w-[74%] lg:via-black/55 lg:via-[50%] lg:to-transparent"
      aria-hidden="true"
    />
    <div
      class="pointer-events-none absolute inset-x-0 bottom-0 z-20 h-[48%] bg-gradient-to-t from-black/85 via-black/45 to-transparent lg:h-[42%] lg:from-black/70 lg:via-black/30"
      aria-hidden="true"
    />

    <!-- Layer 4 — Kolom copy kiri -->
    <div class="relative z-30 w-full">
      <div class="container">
        <div ref="contentRef" class="max-w-[40rem]">
          <span
            ref="badgeRef"
            class="inline-flex items-center rounded-full border border-white/45 px-5 py-2.5 text-[11px] font-semibold uppercase tracking-[0.22em] text-white/95"
          >
            {{ content.meta.projectName }}
          </span>

          <h1
            ref="headlineRef"
            class="mt-6 font-body font-semibold uppercase leading-[0.9] tracking-[-0.02em] text-white text-[clamp(2.1rem,10.2vw,2.5rem)] sm:mt-7 sm:text-[clamp(2.5rem,5.7vw,6rem)]"
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
            class="mt-5 max-w-[30rem] text-[clamp(1rem,1.55vw,1.4rem)] leading-[1.45] text-white/90 sm:mt-6"
          >
            {{ content.hero.subline }}
          </p>

          <div ref="ctaRef" class="mt-7 sm:mt-9">
            <a
              href="#tipe"
              class="group inline-flex items-center gap-3 rounded-xl bg-white px-7 py-[19px] text-[15px] font-semibold leading-6 text-[#141a1c] shadow-[0_18px_44px_rgba(0,0,0,0.32)] transition-colors duration-300 hover:bg-white/95 sm:text-base"
            >
              <span>{{ content.hero.primaryCta }}</span>
              <svg
                class="h-4 w-4 shrink-0 transition-transform duration-300 group-hover:translate-x-1"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
                aria-hidden="true"
              >
                <path d="M4 12h15" />
                <path d="M13 6l6 6-6 6" />
              </svg>
            </a>
          </div>

          <div
            v-if="content.hero.bankInfo && content.banks?.length"
            ref="bankRef"
            class="mt-8 max-w-[22rem] border-t border-white/25 pt-4 sm:mt-12 sm:pt-5"
          >
            <p class="text-[10px] font-semibold uppercase tracking-[0.2em] text-white/85 sm:text-[11px]">
              {{ content.hero.bankInfo.label }}
            </p>

            <div class="mt-4 flex items-center gap-5">
              <template v-for="(bank, idx) in content.banks" :key="bank.name">
                <span
                  v-if="idx > 0"
                  class="h-8 w-px bg-white/30"
                  aria-hidden="true"
                />
                <img
                  :src="bank.imageWhite"
                  :alt="`Logo Bank ${bank.name}`"
                  loading="eager"
                  decoding="async"
                  class="w-auto shrink-0 object-contain"
                  :class="bankHeightClass[bank.name] || 'h-10'"
                />
              </template>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
