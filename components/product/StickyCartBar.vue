<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// ─── Props ────────────────────────────────────────────────────────────────────
const props = defineProps({
  price: {
    type: Number,
    default: null,
  },
  currency: {
    type: String,
    default: 'HUF',
  },
  stockCount: {
    type: Number,
    default: 0,
  },
})

// ─── Visibility state ─────────────────────────────────────────────────────────
// true  → bar slides into view (main ATC is out of viewport)
// false → bar is hidden (main ATC is visible)
const isVisible = ref(false)

// ─── Price formatting ─────────────────────────────────────────────────────────
const formattedPrice = computed(() => {
  const p = Number(props.price)
  if (isNaN(p) || props.price === null) return '—'
  return new Intl.NumberFormat('hu-HU', {
    style: 'currency',
    currency: props.currency || 'HUF',
    maximumFractionDigits: 0,
  }).format(p)
})

// ─── Shipping text ────────────────────────────────────────────────────────────
const shippingText = computed(() => {
  const p = Number(props.price)
  return !isNaN(p) && p >= 10000
    ? 'FREE shipment for this item'
    : 'FREE shipping for orders over 10,000 HUF'
})

// ─── Delivery date range ──────────────────────────────────────────────────────
const deliveryRange = computed(() => {
  const now = new Date()
  const start = new Date(now)
  start.setDate(start.getDate() + 14)
  const end = new Date(now)
  end.setDate(end.getDate() + 28)
  const fmt = (d) =>
    new Intl.DateTimeFormat('en-GB', { day: 'numeric', month: 'short' }).format(d)
  return `${fmt(start)} – ${fmt(end)}`
})

// ─── Add-to-cart handler ──────────────────────────────────────────────────────
const quantity = ref(1)
const showToast = ref(false)
let toastTimeout = null

const addToCart = () => {
  if (toastTimeout) clearTimeout(toastTimeout)
  showToast.value = true
  toastTimeout = setTimeout(() => {
    showToast.value = false
    toastTimeout = null
  }, 3000)
}

// ─── IntersectionObserver setup ───────────────────────────────────────────────
let observer = null
let mutationObserver = null
let pollTimer = null

/**
 * Create (or re-create) the IntersectionObserver for the given target element.
 * Re-creating is necessary after resize because we want a fresh threshold
 * evaluation against the updated viewport geometry.
 */
function attachObserver(target) {
  // Disconnect any previous instance first
  if (observer) {
    observer.disconnect()
    observer = null
  }

  observer = new IntersectionObserver(
    ([entry]) => {
      // Show the sticky bar only when the main ATC is NOT intersecting
      isVisible.value = !entry.isIntersecting
    },
    {
      // rootMargin: '0px' — evaluate against the full viewport
      threshold: 0,
    }
  )

  observer.observe(target)
}

/**
 * Attempt to find #main-add-to-cart and attach the observer.
 * Returns true if the element was found, false otherwise.
 */
function tryAttach() {
  const target = document.getElementById('main-add-to-cart')
  if (target) {
    attachObserver(target)
    return true
  }
  return false
}

/**
 * Edge case 1 — target not yet in DOM on mount:
 * We use a MutationObserver to watch for the element being added to the document
 * body, with a short polling fallback (5 × 200 ms) for environments where
 * MutationObserver callbacks fire too coarsely.
 */
function setupFallback() {
  // MutationObserver: efficient DOM-insertion listener
  mutationObserver = new MutationObserver(() => {
    if (tryAttach()) {
      // Found it — clean up both fallbacks
      mutationObserver.disconnect()
      mutationObserver = null
      if (pollTimer !== null) {
        clearInterval(pollTimer)
        pollTimer = null
      }
    }
  })

  mutationObserver.observe(document.body, {
    childList: true,
    subtree: true,
  })

  // Polling fallback: try every 200 ms for up to 1 s (5 attempts)
  let attempts = 0
  pollTimer = setInterval(() => {
    attempts++
    if (tryAttach() || attempts >= 5) {
      clearInterval(pollTimer)
      pollTimer = null
      if (mutationObserver) {
        mutationObserver.disconnect()
        mutationObserver = null
      }
    }
  }, 200)
}

/**
 * Edge case 2 — resize / orientation change:
 * Re-attach the observer so the IntersectionObserver root bounds are
 * recalculated against the new viewport geometry. Without this, the observer
 * can get stuck if the element moved in/out of view during the resize.
 */
function handleResize() {
  const target = document.getElementById('main-add-to-cart')
  if (target) {
    attachObserver(target)
  }
}

// ─── Lifecycle ────────────────────────────────────────────────────────────────
onMounted(() => {
  // Try immediately; if the element is not yet in the DOM, set up fallbacks.
  if (!tryAttach()) {
    setupFallback()
  }

  window.addEventListener('resize', handleResize, { passive: true })
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
    observer = null
  }
  if (mutationObserver) {
    mutationObserver.disconnect()
    mutationObserver = null
  }
  if (pollTimer !== null) {
    clearInterval(pollTimer)
    pollTimer = null
  }
  if (toastTimeout) {
    clearTimeout(toastTimeout)
    toastTimeout = null
  }
  window.removeEventListener('resize', handleResize)
})
</script>

<template>
  <Teleport to="body">
    <!-- Slide-up transition: translates from below the viewport to in-place -->
    <Transition name="slide-up">
      <div
        v-if="isVisible"
        class="sticky-cart-bar"
        role="complementary"
        aria-label="Quick add to cart"
      >
        <!-- ── Left column: price + shipping info ─────────────────────── -->
        <div class="bar-left">
          <span class="bar-price">{{ formattedPrice }}</span>
          <span class="bar-delivery">Delivery: {{ deliveryRange }}</span>
          <span class="bar-shipping">{{ shippingText }}</span>
        </div>

        <!-- ── Right column: primary CTA ─────────────────────────────── -->
        <div class="bar-right">
          <button
            id="sticky-add-to-cart"
            class="bar-atc-btn"
            @click="addToCart"
            aria-label="Add to cart"
          >
            Add to Cart
          </button>
        </div>
      </div>
    </Transition>

    <!-- Toast notification (teleported alongside the bar) -->
    <Transition name="toast-fade">
      <div v-if="showToast" class="bar-toast" role="alert">
        Successfully added to cart!
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
/* ─── Bar shell ───────────────────────────────────────────────────────────── */
.sticky-cart-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  z-index: 50;

  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--spacing-4, 16px);

  /* Use the page surface color so it matches the background */
  background-color: var(--color-white, #ffffff);
  box-shadow: 0 -4px 6px -1px rgba(0, 0, 0, 0.1);

  padding: var(--spacing-3, 12px) var(--spacing-4, 16px);
}

@media (min-width: 768px) {
  .sticky-cart-bar {
    padding: var(--spacing-4, 16px) var(--spacing-8, 32px);
  }
}

/* ─── Left column ─────────────────────────────────────────────────────────── */
.bar-left {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0; /* allow text to truncate rather than overflow */
}

.bar-price {
  font-family: var(--font-display, 'Comfortaa', cursive);
  font-size: var(--font-size-xl, 24px);
  font-weight: var(--font-weight-bold, 700);
  color: var(--color-dark, #222222);
  line-height: var(--line-height-tight, 1.2);
  white-space: nowrap;
}

.bar-delivery {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-xs, 12px);
  font-weight: var(--font-weight-bold, 700);
  color: var(--color-dark, #222222);
  line-height: var(--line-height-normal, 1.5);
}

.bar-shipping {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-xs, 12px);
  font-weight: var(--font-weight-medium, 500);
  color: var(--color-secondary, #2ebd96);
  line-height: var(--line-height-normal, 1.5);
}

/* ─── Right column ────────────────────────────────────────────────────────── */
.bar-right {
  flex-shrink: 0;
}

.bar-atc-btn {
  height: 48px;
  padding: 0 var(--spacing-6, 24px);
  background-color: var(--color-primary, #d95fa7);
  color: var(--color-white, #ffffff);
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-base, 16px);
  font-weight: var(--font-weight-bold, 700);
  border-radius: var(--radius-md, 8px);
  white-space: nowrap;
  display: flex;
  align-items: center;
  justify-content: center;
  transition:
    background-color var(--transition-base, 0.25s ease),
    transform 0.1s ease;
}

.bar-atc-btn:hover {
  background-color: var(--color-primary-hover, #ff40a7);
}

.bar-atc-btn:active {
  transform: scale(0.97);
}

.bar-atc-btn:focus-visible {
  outline: 2px solid var(--color-primary, #d95fa7);
  outline-offset: 3px;
}

/* ─── Slide-up transition ─────────────────────────────────────────────────── */
.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(100%);
}

.slide-up-enter-to,
.slide-up-leave-from {
  transform: translateY(0);
}

/* ─── Toast notification ──────────────────────────────────────────────────── */
.bar-toast {
  position: fixed;
  top: var(--spacing-6, 24px);
  left: 50%;
  transform: translateX(-50%);
  background-color: var(--color-secondary, #2ebd96);
  color: var(--color-white, #ffffff);
  padding: var(--spacing-3, 12px) var(--spacing-6, 24px);
  border-radius: var(--radius-full, 9999px);
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: var(--font-weight-bold, 700);
  box-shadow: var(--shadow-modal, 0 8px 32px rgba(0, 0, 0, 0.18));
  z-index: var(--z-toast, 400);
  white-space: nowrap;
}

.toast-fade-enter-active,
.toast-fade-leave-active {
  transition:
    opacity var(--transition-base, 0.25s ease),
    transform var(--transition-base, 0.25s ease);
}

.toast-fade-enter-from,
.toast-fade-leave-to {
  opacity: 0;
  transform: translate(-50%, -16px);
}
</style>
