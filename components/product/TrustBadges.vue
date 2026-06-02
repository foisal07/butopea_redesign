<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// ─── Modal state ──────────────────────────────────────────────────────────────
const isModalOpen = ref(false)

function openModal() {
  isModalOpen.value = true
}

function closeModal() {
  isModalOpen.value = false
}

// ─── Keyboard: close on Escape ────────────────────────────────────────────────
function handleKeydown(event) {
  if (event.key === 'Escape' && isModalOpen.value) {
    closeModal()
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <!-- ─── Main trust badge block ─────────────────────────────────────────── -->
  <div class="trust-badges">
    <p class="trust-heading">Why choose Butopea?</p>

    <!-- Row 1: price -->
    <div class="badge-row">
      <!-- Money / banknote icon -->
      <svg
        class="badge-icon"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="1.75"
        stroke-linecap="round"
        stroke-linejoin="round"
        aria-hidden="true"
        focusable="false"
      >
        <rect x="2" y="6" width="20" height="12" rx="2" />
        <circle cx="12" cy="12" r="2" />
        <path d="M6 12h.01M18 12h.01" />
      </svg>
      <p class="trust-line">
        25% less price compared to other brands&nbsp;
        <button class="how-link" @click="openModal" aria-haspopup="dialog">
          How?
        </button>
      </p>
    </div>

    <!-- Row 2: returns -->
    <div class="badge-row">
      <!-- Return / refresh arrow icon -->
      <svg
        class="badge-icon"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="1.75"
        stroke-linecap="round"
        stroke-linejoin="round"
        aria-hidden="true"
        focusable="false"
      >
        <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" />
        <path d="M3 3v5h5" />
      </svg>
      <p class="trust-line">FREE returns within 30 days</p>
    </div>
  </div>

  <!-- ─── Modal (teleported to <body>) ──────────────────────────────────── -->
  <Teleport to="body">
    <Transition name="modal">
      <div
        v-if="isModalOpen"
        class="modal-overlay"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-heading"
        @click.self="closeModal"
      >
        <div class="modal-card">
          <!-- Close button -->
          <button class="modal-close" @click="closeModal" aria-label="Close modal">
            &times;
          </button>

          <!-- Heading -->
          <h2 id="modal-heading" class="modal-heading">
            Why are our prices so good?
          </h2>

          <!-- Body text -->
          <p class="modal-body">
            How can we sell stylish furniture at a more affordable price? By
            saving not on quality, but on logistics costs, as our products are
            shipped directly from the factory, without having to wait at
            intermediate stations.
          </p>

          <!-- Factory-to-house logistics graphic -->
          <div class="image-placeholder">
            <img
              src="~/assets/images/image.png"
              alt="Products ship directly from our factory to your home, bypassing intermediate warehouses"
              class="logistics-image"
            />
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
/* ─── Trust badge block ───────────────────────────────────────────────────── */
.trust-badges {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: var(--spacing-3, 12px) var(--spacing-4, 16px);
  margin-top: var(--spacing-3, 12px);
  background: color-mix(in srgb, var(--color-secondary, #2ebd96) 8%, transparent);
  border: 1px solid color-mix(in srgb, var(--color-secondary, #2ebd96) 25%, transparent);
  border-radius: var(--radius-md, 8px);
}

.trust-heading {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: 500;
  color: var(--color-dark, #222222);
  line-height: var(--line-height-tight, 1.2);
  letter-spacing: 0.04em;
}

/* ─── Icon + text row ─────────────────────────────────────────────────────── */
.badge-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.badge-icon {
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  color: var(--color-dark, #222222)
}

.trust-line {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: 400;
  color: var(--color-grey, #5c5c5c);
  line-height: var(--line-height-normal, 1.5);
  margin: 0;
}

/* ─── "How?" inline link-button ──────────────────────────────────────────── */
.how-link {
  display: inline;
  background: none;
  border: none;
  padding: 0;
  margin: 0;
  font-family: inherit;
  font-size: inherit;
  font-weight: var(--font-weight-medium, 500);
  color: var(--color-secondary);
  cursor: pointer;
  text-decoration: underline;
  text-underline-offset: 2px;
  transition: color var(--transition-fast, 0.15s ease),
              opacity var(--transition-fast, 0.15s ease);
}

.how-link:hover {
  color: var(--color-secondary-bright, #32e7b6);
  text-decoration: underline;
}

.how-link:focus-visible {
  outline: 2px solid var(--color-secondary);
  outline-offset: 2px;
  border-radius: 2px;
}

/* ─── Overlay ─────────────────────────────────────────────────────────────── */
.modal-overlay {
  position: fixed;
  inset: 0;
  z-index: var(--z-modal, 300);
  background: rgba(0, 0, 0, 0.52);
  backdrop-filter: blur(3px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--spacing-4, 16px);
}

/* ─── Card ────────────────────────────────────────────────────────────────── */
.modal-card {
  position: relative;
  background: var(--color-white, #ffffff);
  border-radius: var(--radius-lg, 12px);
  box-shadow: var(--shadow-modal, 0 8px 32px rgba(0, 0, 0, 0.18));
  padding: var(--spacing-8, 32px) var(--spacing-8, 32px) var(--spacing-6, 24px);
  max-width: 480px;
  width: 100%;
}

/* ─── Close button ────────────────────────────────────────────────────────── */
.modal-close {
  position: absolute;
  top: var(--spacing-4, 16px);
  right: var(--spacing-4, 16px);
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--color-bg-light, #f4f4f4);
  border: none;
  border-radius: var(--radius-full, 9999px);
  font-size: 20px;
  line-height: 1;
  color: var(--color-grey, #5c5c5c);
  cursor: pointer;
  transition: background var(--transition-fast, 0.15s ease),
              color var(--transition-fast, 0.15s ease);
}

.modal-close:hover {
  background: var(--color-light-grey, #e6e6e6);
  color: var(--color-dark, #222222);
}

.modal-close:focus-visible {
  outline: 2px solid var(--color-secondary);
  outline-offset: 2px;
}

/* ─── Modal typography ────────────────────────────────────────────────────── */
.modal-heading {
  font-family: var(--font-display, 'Comfortaa', cursive);
  font-size: var(--font-size-xl, 24px);
  font-weight: var(--font-weight-bold, 700);
  color: var(--color-dark, #222222);
  line-height: var(--line-height-tight, 1.2);
  margin-bottom: var(--spacing-4, 16px);
  /* Leave room for the close button */
  padding-right: var(--spacing-8, 32px);
}

.modal-body {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-base, 16px);
  font-weight: var(--font-weight-regular, 400);
  color: var(--color-grey, #5c5c5c);
  line-height: var(--line-height-loose, 1.75);
  margin-bottom: var(--spacing-6, 24px);
}

/* ─── Logistics image ─────────────────────────────────────────────────────── */
.image-placeholder {
  width: 100%;
  border-radius: var(--radius-md, 8px);
  overflow: hidden;
  background: var(--color-bg-light, #f4f4f4);
}

.logistics-image {
  width: 100%;
  height: auto;
  display: block;
  object-fit: contain;
}

/* ─── Enter / Leave transition ────────────────────────────────────────────── */
.modal-enter-active,
.modal-leave-active {
  transition: opacity var(--transition-base, 0.25s ease);
}

.modal-enter-active .modal-card,
.modal-leave-active .modal-card {
  transition: opacity var(--transition-base, 0.25s ease),
              transform var(--transition-base, 0.25s ease);
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-card,
.modal-leave-to .modal-card {
  opacity: 0;
  transform: translateY(12px) scale(0.97);
}
</style>
