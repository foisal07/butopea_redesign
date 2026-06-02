<script setup>
import { computed } from 'vue'

// ─── Props ────────────────────────────────────────────────────────────────────
const props = defineProps({
  price: {
    type: Number,
    default: null,
  },
})

// ─── Shipping text ────────────────────────────────────────────────────────────
// Edge case: price is undefined / null / NaN → treat as sub-threshold (no crash).
const shippingText = computed(() => {
  const p = Number(props.price)
  return !isNaN(p) && p >= 10000
    ? 'FREE shipping for this item'
    : 'FREE shipping for orders over 10,000 HUF'
})

// ─── Delivery date range ──────────────────────────────────────────────────────
// Pure native JS — strictly no external libraries.
// Window starts 14 days from today; ends 14 days after that (28 days out).
// JS Date arithmetic handles month / year roll-overs automatically.
const deliveryRange = computed(() => {
  const now = new Date()

  const start = new Date(now)
  start.setDate(start.getDate() + 14)

  const end = new Date(now)
  end.setDate(end.getDate() + 28)

  // Intl.DateTimeFormat gives locale-aware, human-readable output.
  const fmt = (date) =>
    new Intl.DateTimeFormat('en-GB', { day: 'numeric', month: 'short' }).format(date)

  return `${fmt(start)} – ${fmt(end)}`
})
</script>

<template>
  <div class="shipment">
    <div class="shipment-row">
      <!-- Inline SVG: delivery truck -->
      <svg
        class="truck-icon"
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
        <!-- Cargo box body -->
        <rect x="1" y="3" width="15" height="13" rx="1" />
        <!-- Truck cab -->
        <path d="M16 8h4l3 5v4h-7V8Z" />
        <!-- Front wheel -->
        <circle cx="5.5" cy="18.5" r="2.5" />
        <!-- Rear wheel -->
        <circle cx="18.5" cy="18.5" r="2.5" />
      </svg>

      <!-- Text block -->
      <div class="shipment-text">
        <span class="shipping-text">{{ shippingText }}</span>
        <span class="delivery-text">Delivery: {{ deliveryRange }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* ─── Wrapper ─────────────────────────────────────────────────────────────── */
.shipment {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding: var(--spacing-3, 12px) var(--spacing-4, 16px);
  margin-top: var(--spacing-4, 16px);
  background: color-mix(in srgb, var(--color-secondary, #2ebd96) 8%, transparent);
  border: 1px solid color-mix(in srgb, var(--color-secondary, #2ebd96) 25%, transparent);
  border-radius: var(--radius-md, 8px);
}

/* ─── Row (icon + text) ───────────────────────────────────────────────────── */
.shipment-row {
  display: flex;
  align-items: flex-start;
  gap: 10px;
}

/* ─── Truck icon ──────────────────────────────────────────────────────────── */
.truck-icon {
  flex-shrink: 0;
  width: 26px;
  height: 26px;
  margin-top: 1px;
  color: var(--color-secondary, #2ebd96); /* Teal via currentColor */
  transition: transform var(--transition-fast, 0.15s ease);
}

.shipment-row:hover .truck-icon {
  transform: translateX(4px);
}

/* ─── Text block ──────────────────────────────────────────────────────────── */
.shipment-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

/* Shipping message — bold + brand teal */
.shipping-text {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: 700;                /* spec: bold */
  color: var(--color-secondary, #2ebd96); /* spec: Teal brand token */
  line-height: var(--line-height-tight, 1.2);
}

/* Delivery date range — subtler secondary text */
.delivery-text {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-xs, 12px);
  font-weight: var(--font-weight-bold, 700);
  color: var(--color-dark, #222222);
  line-height: var(--line-height-normal, 1.5);
}
</style>
