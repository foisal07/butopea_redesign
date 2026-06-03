<script setup>
import { ref, watch } from 'vue'

// ── Props & Emits ─────────────────────────────────────────────────────────────
const props = defineProps({
  colors: {
    type: Array,
    default: null
  }
})

const emit = defineEmits(['update:color'])

// ── State ─────────────────────────────────────────────────────────────────────
// Initialise selectedColor from the first color that has `active: true`,
// falling back to the first item in the array.
const initialColor = () => {
  if (!props.colors || props.colors.length === 0) return null
  return props.colors.find(c => c.active) ?? props.colors[0]
}

const selectedColor = ref(initialColor())

// Keep selection in sync if the parent swaps out the entire colors array.
watch(
  () => props.colors,
  () => { selectedColor.value = initialColor() },
  { flush: 'sync' }
)

// ── Interaction ───────────────────────────────────────────────────────────────
const selectColor = (color) => {
  if (selectedColor.value?.id === color.id) return
  selectedColor.value = color
  emit('update:color', color)
}
</script>

<template>
  <div v-if="colors && colors.length > 0" class="variant-selector">
    <!-- Label row -->
    <div class="swatch-row">
      <span class="color-label">
        Color:
      </span>

      <!-- Swatches -->
      <div class="swatches" role="group" aria-label="Color variants">
        <button
          v-for="color in colors"
          :key="color.id"
          type="button"
          class="swatch"
          :class="{ 'swatch--active': selectedColor?.id === color.id }"
          :style="{ backgroundColor: color.hex || '#e5e5e5' }"
          :aria-label="`Select color: ${color.label}`"
          :aria-pressed="selectedColor?.id === color.id"
          :title="color.label"
          @click="selectColor(color)"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.variant-selector {
  margin: var(--spacing-4, 16px) 0;
}

/* ── Label + Swatches inline row ───────────────────────────────────────────── */
.swatch-row {
  display: flex;
  align-items: center;
  gap: var(--spacing-4, 16px);
  flex-wrap: wrap;
}

.color-label {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  color: var(--color-dark, #222);
  font-weight: var(--font-weight-regular, 400);
  white-space: nowrap;
  line-height: 1;
}

.color-label strong {
  color: var(--color-dark, #222);
  font-weight: var(--font-weight-medium, 500);
}

/* ── Swatch container ─────────────────────────────────────────────────────── */
.swatches {
  display: flex;
  align-items: center;
  gap: var(--spacing-2, 8px);
  flex-wrap: wrap;
}

/* ── Individual swatch button ─────────────────────────────────────────────── */
.swatch {
  /* Base size */
  width: 20px;
  height: 20px;
  border-radius: 50%;

  /* Reset button defaults */
  border: none;
  padding: 0;
  cursor: pointer;

  /* Smooth transitions for all interactive states */
  transition:
    transform 0.18s ease,
    box-shadow 0.18s ease;

  /* Fallback for missing hex */
  background-color: #e5e5e5;
}

.swatch:hover:not(.swatch--active) {
  transform: scale(1.12);
  box-shadow: 0 0 0 2px var(--color-grey, #757575);
}

/* ── Active state: outer ring + white gap ─────────────────────────────────── */
.swatch--active {
  /* Slightly larger */
  transform: scale(1.15);
  /* White gap via outline + box-shadow ring */
  outline: 2px solid transparent;
  box-shadow:
    0 0 0 1px var(--color-white, #ffffff),
    0 0 0 2px var(--color-grey, #9e9e9e);
  cursor: default;
}
</style>
