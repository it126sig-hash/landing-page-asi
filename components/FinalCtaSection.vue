<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from 'vue';
import { useContent } from '~/composables/useContent';
import { useConsultationForm } from '~/composables/useConsultationForm';

const { content } = useContent();

const {
  isSubmitting,
  isSuccess,
  isError,
  isCaptchaStep,
  isNotConfigured,
  captchaLoading,
  captchaErrorMsg,
  submit: submitToEngine,
  reset: resetEngine,
  renderRecaptchaWidget,
  retryCaptcha,
} = useConsultationForm();

const isModalOpen = ref(false);
const form = ref({
  name: '',
  phone: '',
  interest: content.contactCta?.options?.[0]?.value || 'Belum menentukan',
});

const validationError = ref('');
const recaptchaContainer = ref(null);

const directWaUrl = computed(() => {
  const number = content.meta.whatsapp.number;
  const project = content.meta.projectName;
  const nameStr = form.value.name ? `Nama: ${form.value.name}. ` : '';
  const interestStr = form.value.interest ? `Kebutuhan: ${form.value.interest}. ` : '';
  const msg = `Halo tim marketing ${project}, saya ingin konsultasi. ${nameStr}${interestStr}Mohon informasi lebih lanjut.`;
  return `https://wa.me/${number}?text=${encodeURIComponent(msg)}`;
});

function openModal() {
  isModalOpen.value = true;
}

function closeModal() {
  isModalOpen.value = false;
}

function onKeydown(e) {
  if (e.key === 'Escape' && isModalOpen.value) {
    closeModal();
  }
}

watch(isModalOpen, (val) => {
  if (typeof document !== 'undefined') {
    document.body.style.overflow = val ? 'hidden' : '';
  }
});

watch(isCaptchaStep, (newVal) => {
  if (newVal && isModalOpen.value) {
    nextTick(() => {
      if (recaptchaContainer.value) {
        renderRecaptchaWidget(recaptchaContainer.value);
      }
    });
  }
});

function handleSubmit() {
  validationError.value = '';
  const trimmedName = form.value.name.trim();
  const trimmedPhone = form.value.phone.trim();

  if (!trimmedName || trimmedName.length < 2) {
    validationError.value = content.consultationForm?.validation?.nameMin || 'Silakan masukkan nama lengkap Anda.';
    return;
  }

  const cleanPhone = trimmedPhone.replace(/[\s\-().]/g, '');
  if (!cleanPhone || cleanPhone.length < 9) {
    validationError.value = content.consultationForm?.validation?.phoneInvalid || 'Silakan masukkan nomor WhatsApp yang valid.';
    return;
  }

  submitToEngine({
    name: trimmedName,
    phone: cleanPhone,
    interest: form.value.interest,
    source: 'landing-page-asi',
  });
}

function handleRetryCaptcha() {
  if (recaptchaContainer.value) {
    retryCaptcha(recaptchaContainer.value);
  }
}

function resetForm() {
  form.value = {
    name: '',
    phone: '',
    interest: content.contactCta?.options?.[0]?.value || 'Belum menentukan',
  };
  validationError.value = '';
  resetEngine();
}

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
  <section id="kontak" class="bg-white py-14 sm:py-20 scroll-mt-20">
    <div class="container">
      <div class="relative overflow-hidden rounded-3xl bg-gradient-to-r from-[var(--green-deep)] via-[var(--green)] to-[#255CA8] text-white p-8 sm:p-12 lg:px-14 lg:py-12 shadow-lg reveal">
        <!-- Siluet Ilustrasi Deretan Rumah SVG Halus (Kanan Bawah) -->
        <svg
          class="pointer-events-none absolute -bottom-2 right-0 w-[340px] sm:w-[440px] lg:w-[520px] h-auto text-[#173c72] opacity-15"
          viewBox="0 0 520 140"
          fill="currentColor"
          aria-hidden="true"
        >
          <path d="M12 140V84l44-32 44 32v56H12Zm96 0V68l52-38 52 38v72h-104Zm114 0V52l58-42 58 42v88H222Zm124 0V74l48-34 48 34v66h-96Zm104 0V88l35-26 35 26v52h-70Z" />
          <path d="M40 140v-30h24v30H40Zm108 0v-36h26v36h-26Zm118 0v-42h30v42h-30Zm118 0v-32h24v32h-24Z" opacity="0.5" />
        </svg>

        <div class="relative z-10 flex flex-col lg:flex-row lg:items-center lg:justify-between gap-6 lg:gap-10">
          <!-- Copy Kiri -->
          <div class="max-w-[560px]">
            <h2 class="font-sans text-2xl sm:text-3xl lg:text-[36px] font-bold tracking-[-0.02em] leading-tight text-white">
              {{ content.contactCta.bannerTitle || content.contactCta.title }}
            </h2>
            <p class="mt-3 text-sm sm:text-[15px] leading-relaxed text-white/90">
              {{ content.contactCta.bannerDesc || content.contactCta.desc }}
            </p>
          </div>

          <!-- Tombol Pill Putih Kanan -->
          <div class="shrink-0">
            <button
              type="button"
              class="group bg-white text-[var(--ink)] hover:bg-[var(--sage-pale)] rounded-full px-7 py-3.5 text-sm font-semibold shadow-md inline-flex items-center gap-2.5 cursor-pointer transition-all duration-200 hover:-translate-y-0.5"
              @click="openModal"
            >
              <span>{{ content.contactCta.bannerCtaLabel || 'Hubungi Kami' }}</span>
              <svg
                class="w-4 h-4 transition-transform duration-200 group-hover:translate-x-0.5"
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
      </div>
    </div>

    <!-- Modal Form Konsultasi -->
    <Teleport to="body">
      <div
        v-if="isModalOpen"
        class="fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6 bg-[#17253b]/60 backdrop-blur-sm"
        role="dialog"
        aria-modal="true"
        :aria-label="content.consultationForm.title"
        @click.self="closeModal"
      >
        <div class="relative w-full max-w-md max-h-[90vh] overflow-y-auto rounded-3xl bg-white p-6 sm:p-8 shadow-2xl border border-[var(--line)] text-[var(--ink)]">
          <button
            type="button"
            class="absolute top-4 right-4 w-9 h-9 rounded-full bg-[var(--sage-pale)] text-[var(--ink)] hover:bg-[var(--green-soft)] grid place-items-center transition-colors cursor-pointer"
            aria-label="Tutup form konsultasi"
            @click="closeModal"
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
              <path d="M18 6 6 18" />
              <path d="m6 6 12 12" />
            </svg>
          </button>

          <div class="pr-8 mb-5">
            <span class="inline-block text-[11px] font-bold uppercase tracking-wider text-[var(--green)]">
              {{ content.contactCta.eyebrow }}
            </span>
            <h3 class="mt-1 font-sans text-xl sm:text-2xl font-bold text-[var(--ink)]">
              {{ content.consultationForm.title }}
            </h3>
            <p class="mt-1.5 text-xs sm:text-[13px] leading-relaxed text-[var(--muted)]">
              {{ content.consultationForm.microcopy }}
            </p>
          </div>

          <!-- Sukses State -->
          <div v-if="isSuccess" class="p-5 rounded-2xl bg-[var(--sage-pale)] border border-[var(--green-soft)] text-center space-y-4">
            <div class="w-12 h-12 mx-auto rounded-full bg-emerald-100 text-emerald-700 flex items-center justify-center text-xl font-bold">
              ✓
            </div>
            <p class="text-xs sm:text-sm text-[var(--ink)] font-medium leading-relaxed">
              {{ content.consultationForm.messages.success }}
            </p>
            <a
              :href="directWaUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="btn btn-primary w-full"
            >
              <span>{{ content.consultationForm.successCtaWa }}</span>
              <span aria-hidden="true">↗</span>
            </a>
            <button
              type="button"
              class="text-xs font-semibold text-[var(--muted)] hover:text-[var(--ink)] underline cursor-pointer"
              @click="resetForm"
            >
              Kirim formulir lain
            </button>
          </div>

          <!-- Captcha Verification Step -->
          <div v-else-if="isCaptchaStep" class="p-4 rounded-2xl bg-[var(--paper)] border border-[var(--line)] space-y-4">
            <p class="text-xs font-medium text-center text-[var(--ink)]">
              Selesaikan verifikasi keamanan singkat di bawah untuk mengirim data:
            </p>
            <div ref="recaptchaContainer" class="flex justify-center min-h-[78px] py-1" />
            <div v-if="captchaLoading" class="text-xs text-[var(--muted)] text-center">
              Memuat verifikasi...
            </div>
            <div v-if="captchaErrorMsg" class="text-xs text-rose-700 text-center bg-rose-50 border border-rose-200 p-2.5 rounded-xl space-y-2">
              <p>{{ captchaErrorMsg }}</p>
              <button
                type="button"
                class="font-semibold underline text-[var(--green)] hover:text-[var(--green-deep)] cursor-pointer"
                @click="handleRetryCaptcha"
              >
                {{ content.consultationForm.captcha.retry }}
              </button>
            </div>
            <a
              :href="directWaUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="btn btn-outline w-full text-xs"
            >
              <span>{{ content.consultationForm.captcha.fallbackCta }}</span>
              <span aria-hidden="true">↗</span>
            </a>
          </div>

          <!-- Error State / Not Configured -->
          <div v-else-if="isError || isNotConfigured" class="p-5 rounded-2xl bg-[var(--paper)] border border-[var(--line)] text-center space-y-3">
            <p class="text-xs text-[var(--muted)] leading-relaxed">
              {{ isNotConfigured ? content.consultationForm.messages.notConfigured : content.consultationForm.messages.error }}
            </p>
            <a
              :href="directWaUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="btn btn-primary w-full"
            >
              <span>{{ content.consultationForm.errorCtaWa }}</span>
              <span aria-hidden="true">↗</span>
            </a>
            <button
              type="button"
              class="text-xs font-semibold text-[var(--muted)] hover:text-[var(--ink)] underline block mx-auto cursor-pointer"
              @click="resetForm"
            >
              {{ content.consultationForm.errorCtaRetry }}
            </button>
          </div>

          <!-- Input Form Normal -->
          <form v-else class="space-y-4" @submit.prevent="handleSubmit">
            <div v-if="validationError" class="p-2.5 rounded-xl bg-rose-50 border border-rose-200 text-rose-700 text-xs text-center">
              {{ validationError }}
            </div>

            <div>
              <label for="c-name" class="block text-xs font-semibold text-[var(--ink)] mb-1.5">
                {{ content.consultationForm.labels.name }}
              </label>
              <input
                id="c-name"
                v-model="form.name"
                type="text"
                required
                :placeholder="content.consultationForm.placeholders.name"
                class="w-full h-11 px-3.5 rounded-xl bg-[var(--paper)] text-[var(--ink)] text-sm border border-[var(--line)] focus:bg-white focus:outline-none focus:border-[var(--green)] focus:ring-2 focus:ring-[var(--green)]/20"
              >
            </div>

            <div>
              <label for="c-phone" class="block text-xs font-semibold text-[var(--ink)] mb-1.5">
                {{ content.consultationForm.labels.phone }}
              </label>
              <input
                id="c-phone"
                v-model="form.phone"
                type="tel"
                required
                :placeholder="content.consultationForm.placeholders.phone"
                class="w-full h-11 px-3.5 rounded-xl bg-[var(--paper)] text-[var(--ink)] text-sm border border-[var(--line)] focus:bg-white focus:outline-none focus:border-[var(--green)] focus:ring-2 focus:ring-[var(--green)]/20"
              >
            </div>

            <div>
              <label for="c-interest" class="block text-xs font-semibold text-[var(--ink)] mb-1.5">
                {{ content.consultationForm.labels.need }}
              </label>
              <select
                id="c-interest"
                v-model="form.interest"
                class="w-full h-11 px-3.5 rounded-xl bg-[var(--paper)] text-[var(--ink)] text-sm border border-[var(--line)] focus:bg-white focus:outline-none focus:border-[var(--green)] focus:ring-2 focus:ring-[var(--green)]/20"
              >
                <option
                  v-for="opt in content.contactCta.options"
                  :key="opt.value"
                  :value="opt.value"
                >
                  {{ opt.label }}
                </option>
              </select>
            </div>

            <button
              type="submit"
              :disabled="isSubmitting"
              class="btn btn-primary w-full disabled:opacity-60"
            >
              <span>{{ isSubmitting ? content.consultationForm.messages.submitting : content.contactCta.ctaLabel }}</span>
              <span aria-hidden="true">→</span>
            </button>

            <div class="pt-1 flex items-center justify-between gap-2 text-[11px] text-[var(--muted)]">
              <span>{{ content.consultationForm.disclaimer }}</span>
              <a
                :href="directWaUrl"
                target="_blank"
                rel="noopener noreferrer"
                class="shrink-0 font-semibold text-[var(--green)] hover:underline"
              >
                WhatsApp →
              </a>
            </div>
          </form>
        </div>
      </div>
    </Teleport>
  </section>
</template>
