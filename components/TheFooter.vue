<script setup>
import { ref } from 'vue';
import { useContent } from '~/composables/useContent';

const { content } = useContent();

const currentYear = new Date().getFullYear();
const contactInput = ref('');
const newsletterError = ref('');
const newsletterSent = ref(false);

function handleNewsletterSubmit() {
  const val = contactInput.value.trim();
  newsletterError.value = '';
  newsletterSent.value = false;

  if (!val || val.length < 4) {
    newsletterError.value = 'Mohon masukkan nomor WhatsApp atau email Anda.';
    return;
  }

  const number = content.meta.whatsapp.number;
  const project = content.meta.projectName;
  const msg = `Halo tim marketing ${project}, saya ingin mendapatkan informasi terbaru & promo hunian. Kontak saya: ${val}`;
  const url = `https://wa.me/${number}?text=${encodeURIComponent(msg)}`;
  window.open(url, '_blank', 'noopener,noreferrer');

  contactInput.value = '';
  newsletterSent.value = true;
}

function scrollToTop() {
  const el = document.getElementById('beranda');
  if (el) {
    el.scrollIntoView({ behavior: 'smooth' });
  } else {
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }
}
</script>

<template>
  <footer class="bg-[var(--green-deep)] text-white pt-14 pb-24">
    <div class="container">
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-[1.35fr_0.75fr_0.85fr_1.15fr] gap-10 pb-12">
        <!-- Kolom 1: Brand -->
        <div>
          <a
            href="#beranda"
            class="inline-flex items-center gap-3"
            :aria-label="`${content.meta.projectName} — Beranda`"
          >
            <img
              :src="content.meta.logo"
              :alt="`Logo ${content.meta.projectName}`"
              loading="lazy"
              class="h-10 w-auto rounded bg-white p-1 object-contain shrink-0"
            />
            <span class="flex flex-col">
              <strong class="font-display text-base sm:text-lg font-bold tracking-tight text-white leading-tight">
                {{ content.meta.projectName }}
              </strong>
              <small class="text-[11px] text-white/65 tracking-wide">
                {{ content.meta.developer }}
              </small>
            </span>
          </a>

          <p class="mt-4 text-xs sm:text-[13px] leading-relaxed text-white/75 max-w-xs">
            {{ content.footer.desc }}
          </p>
        </div>

        <!-- Kolom 2: Menu -->
        <div>
          <h3 class="text-xs font-bold uppercase tracking-wider text-white mb-4">
            Menu
          </h3>
          <nav class="flex flex-col space-y-2.5" aria-label="Menu footer">
            <a
              v-for="link in content.footer.quickLinks"
              :key="link.id"
              :href="`#${link.id}`"
              class="text-xs sm:text-[13px] text-white/75 hover:text-white transition-colors w-fit"
            >
              {{ link.label }}
            </a>
          </nav>
        </div>

        <!-- Kolom 3: Ikuti Kami -->
        <div>
          <h3 class="text-xs font-bold uppercase tracking-wider text-white mb-4">
            Ikuti Kami
          </h3>
          <div class="flex flex-wrap items-center gap-2.5">
            <a
              v-for="soc in content.footer.socials"
              :key="soc.id"
              :href="soc.href"
              target="_blank"
              rel="noopener noreferrer"
              :aria-label="soc.label"
              class="w-9 h-9 rounded-full bg-white/10 border border-white/15 text-white hover:bg-white hover:text-[var(--green-deep)] grid place-items-center transition-colors"
            >
              <!-- YouTube -->
              <svg v-if="soc.iconKey === 'youtube'" class="w-4 h-4 fill-current" viewBox="0 0 24 24" aria-hidden="true">
                <path d="M23.5 6.2a3 3 0 0 0-2.1-2.1C19.5 3.5 12 3.5 12 3.5s-7.5 0-9.4.6A3 3 0 0 0 .5 6.2 31.4 31.4 0 0 0 0 12a31.4 31.4 0 0 0 .5 5.8 3 3 0 0 0 2.1 2.1c1.9.6 9.4.6 9.4.6s7.5 0 9.4-.6a3 3 0 0 0 2.1-2.1A31.4 31.4 0 0 0 24 12a31.4 31.4 0 0 0-.5-5.8ZM9.6 15.6V8.4l6.3 3.6-6.3 3.6Z" />
              </svg>
              <!-- Instagram -->
              <svg v-else-if="soc.iconKey === 'instagram'" class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <rect x="2" y="2" width="20" height="20" rx="5" />
                <circle cx="12" cy="12" r="4" />
                <circle cx="17.5" cy="6.5" r="1" fill="currentColor" stroke="none" />
              </svg>
              <!-- Facebook -->
              <svg v-else-if="soc.iconKey === 'facebook'" class="w-4 h-4 fill-current" viewBox="0 0 24 24" aria-hidden="true">
                <path d="M13.5 22v-8.2h2.8l.4-3.2h-3.2V8.6c0-.9.3-1.6 1.6-1.6h1.7V4.1c-.3 0-1.3-.1-2.5-.1-2.5 0-4.2 1.5-4.2 4.3v2.3H7.3v3.2h2.8V22h3.4Z" />
              </svg>
              <!-- WhatsApp -->
              <svg v-else class="w-4 h-4 fill-current" viewBox="0 0 24 24" aria-hidden="true">
                <path d="M.057 24l1.687-6.163c-1.041-1.804-1.588-3.849-1.587-5.946.003-6.556 5.338-11.891 11.893-11.891 3.181.001 6.167 1.24 8.413 3.488 2.245 2.248 3.481 5.236 3.48 8.414-.003 6.557-5.338 11.892-11.893 11.892-1.99-.001-3.951-.5-5.688-1.448l-6.305 1.654zm6.597-3.807c1.676.995 3.276 1.591 5.392 1.592 5.448 0 9.886-4.434 9.889-9.885.002-5.462-4.415-9.89-9.881-9.892-5.452 0-9.887 4.434-9.889 9.884-.001 2.225.651 3.891 1.746 5.634l-.999 3.648 3.742-.981zm11.387-5.464c-.074-.124-.272-.198-.57-.347-.297-.149-1.758-.868-2.031-.967-.272-.099-.47-.149-.669.149-.198.297-.768.967-.941 1.165-.173.198-.347.223-.644.074-.297-.149-1.255-.462-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.297-.347.446-.521.151-.172.2-.296.3-.495.099-.198.05-.372-.025-.521-.075-.148-.669-1.611-.916-2.206-.242-.579-.487-.501-.669-.51l-.57-.01c-.198 0-.52.074-.792.372s-1.04 1.016-1.04 2.479 1.065 2.876 1.213 3.074c.149.198 2.095 3.2 5.076 4.487.709.306 1.263.489 1.694.626.712.226 1.36.194 1.872.118.571-.085 1.758-.719 2.006-1.413.248-.695.248-1.29.173-1.414z" />
              </svg>
            </a>
          </div>
        </div>

        <!-- Kolom 4: Dapatkan Informasi Terbaru -->
        <div v-if="content.footer.newsletter">
          <h3 class="text-xs font-bold uppercase tracking-wider text-white mb-4">
            {{ content.footer.newsletter.title }}
          </h3>
          <form
            class="bg-white rounded-full pl-4 pr-1.5 py-1.5 flex items-center gap-2 shadow-sm"
            @submit.prevent="handleNewsletterSubmit"
          >
            <label for="footer-newsletter-input" class="sr-only">
              {{ content.footer.newsletter.placeholder }}
            </label>
            <input
              id="footer-newsletter-input"
              v-model="contactInput"
              type="text"
              :placeholder="content.footer.newsletter.placeholder"
              class="w-full min-w-0 bg-transparent text-xs text-[var(--ink)] placeholder-[var(--muted)] focus:outline-none"
            >
            <button
              type="submit"
              :aria-label="content.footer.newsletter.buttonLabel"
              class="w-9 h-9 shrink-0 rounded-full bg-[var(--green)] hover:bg-[#1e4e98] text-white grid place-items-center transition-colors cursor-pointer"
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
          </form>

          <p v-if="newsletterError" class="mt-2 text-[11px] text-amber-200">
            {{ newsletterError }}
          </p>
          <p v-else-if="newsletterSent" class="mt-2 text-[11px] text-emerald-200">
            Menghubungkan ke WhatsApp marketing...
          </p>
        </div>
      </div>

      <!-- Bar Bawah -->
      <div class="border-t border-white/15 pt-6 flex flex-col sm:flex-row justify-between items-center gap-4 text-xs text-white/65">
        <span>© {{ currentYear }} {{ content.meta.projectName }}. All rights reserved.</span>

        <div class="flex items-center gap-4">
          <span class="text-center sm:text-right">
            {{ content.meta.developer }} · {{ content.footer.mainAddress.lines[0] }}
          </span>
          <button
            type="button"
            aria-label="Kembali ke atas"
            class="w-9 h-9 shrink-0 rounded-full border border-white/25 bg-white/10 text-white hover:bg-white hover:text-[var(--green-deep)] grid place-items-center transition-colors cursor-pointer"
            @click="scrollToTop"
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
              <path d="M12 19V5" />
              <path d="m6 11 6-6 6 6" />
            </svg>
          </button>
        </div>
      </div>
    </div>
  </footer>
</template>
