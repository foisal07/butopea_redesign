<script setup>
import { computed } from 'vue'
import mockData from '~/data/product.mock.json'

/**
 * StatusLabel
 * A reusable pill-shaped label for status indicators.
 *
 * Props:
 *   text    {string} — the label text, e.g. "Available", "In Stock"
 *   variant {string} — controls color scheme:
 *              "success"  (default) → teal  — for in-stock / available
 *              "warning"            → gold  — for low-stock / limited
 *              "error"              → pink  — for out-of-stock / unavailable
 *              "primary"            → brand pink — for promotional tags
 *              "neutral"            → grey  — for informational labels
 */
const props = defineProps({
  text: {
    type: String,
    required: false,
  },
  variant: {
    type: String,
    default: 'success',
    validator: (v) =>
      ['success', 'warning', 'error', 'primary', 'neutral'].includes(v),
  },
})

const displayData = computed(() => {
  if (props.text) {
    return { text: props.text, variant: props.variant };
  }
  
  const stockCount = mockData.stockCount || 0;
  
  if (stockCount === 0) {
    return { text: 'Out of stock', variant: 'error' };
  } else if (stockCount < 5) {
    return { text: 'Finishing', variant: 'warning' };
  } else {
    return { text: 'Available', variant: 'success' };
  }
})
</script>

<template>
  <span
    :class="['status-label', `status-label--${displayData.variant}`]"
    role="status"
    :aria-label="displayData.text"
  >
    <span class="status-label__dot" aria-hidden="true"></span>
    {{ displayData.text }}
  </span>
</template>

<style scoped>
/* ---- Base ---- */
.status-label {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-1);
  padding: 3px var(--spacing-3);
  border-radius: var(--radius-full);
  font-family: var(--font-body);
  font-size: var(--font-size-xs);
  font-weight: var(--font-weight-medium);
  letter-spacing: 0.04em;
  text-transform: uppercase;
  white-space: nowrap;
  transition: opacity var(--transition-fast);
  user-select: none;
}

/* ---- Leading dot ---- */
.status-label__dot {
  display: inline-block;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  flex-shrink: 0;
  background-color: currentColor;
  opacity: 0.75;
}

/* ---- Variants ---- */

/* success → teal (in-stock, available) */
.status-label--success {
  background-color: rgba(46, 189, 150, 0.12);
  color: var(--color-success);
  border: 1px solid rgba(46, 189, 150, 0.3);
}

/* warning → gold (low stock, limited) */
.status-label--warning {
  background-color: rgba(238, 189, 1, 0.12);
  color: var(--color-gold);
  border: 1px solid rgba(238, 189, 1, 0.3);
}

/* error → red (out of stock) */
.status-label--error {
  background-color: rgba(235, 87, 87, 0.12);
  color: var(--color-error);
  border: 1px solid rgba(235, 87, 87, 0.3);
}

/* primary → brand pink (promo, featured) */
.status-label--primary {
  background-color: rgba(217, 95, 167, 0.12);
  color: var(--color-primary);
  border: 1px solid rgba(217, 95, 167, 0.3);
}

/* neutral → grey (informational) */
.status-label--neutral {
  background-color: rgba(92, 92, 92, 0.08);
  color: var(--color-grey);
  border: 1px solid var(--color-light-grey);
}
</style>
