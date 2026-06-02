<script setup>
import { ref } from 'vue'
import productData from '~/data/product.mock.json'

// ─── Data ─────────────────────────────────────────────────────────────────────
const product = ref(productData)

// ─── Accordion state — index 0 open by default ────────────────────────────────
const openIndex = ref(null)

function toggle(index) {
  openIndex.value = openIndex.value === index ? null : index
}

// ─── Section definitions ──────────────────────────────────────────────────────
// Each section exposes a `hasContent` guard so the template can gracefully
// handle missing/empty data without throwing.
const sections = [
  {
    id: 'features',
    label: 'Features & Dimensions',
    // Grouped array: [{ group, items[] }]
    hasContent: () =>
      Array.isArray(product.value.attributes) &&
      product.value.attributes.length > 0 &&
      product.value.attributes.some(g => g.items?.length > 0),
  },
  {
    id: 'wrapping',
    label: 'Wrapping',
    hasContent: () => Boolean(product.value.packaging),
  },
  {
    id: 'product-details',
    label: 'Product Details',
    hasContent: () =>
      (Array.isArray(product.value.productDetails) && product.value.productDetails.length > 0) ||
      Boolean(product.value.description),
  },
]
</script>

<template>
  <div class="accordion" role="list">
    <!-- ── Section 0 : Features & Dimensions ─────────────────────────────── -->
    <div
      class="accordion-item"
      :class="{ 'is-open': openIndex === 0 }"
      role="listitem"
    >
      <button
        class="accordion-header"
        :aria-expanded="openIndex === 0"
        :aria-controls="`panel-${sections[0].id}`"
        :id="`header-${sections[0].id}`"
        @click="toggle(0)"
      >
        <span class="header-label">{{ sections[0].label }}</span>
        <svg
          class="chevron"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
          focusable="false"
        >
          <polyline points="6 9 12 15 18 9" />
        </svg>
      </button>

      <div
        class="accordion-body"
        :id="`panel-${sections[0].id}`"
        role="region"
        :aria-labelledby="`header-${sections[0].id}`"
      >
        <div class="accordion-inner">
          <div class="accordion-content">
            <!-- Has data: iterate groups, render a heading + dl per group -->
            <template v-if="sections[0].hasContent()">
              <div
                v-for="group in product.attributes"
                :key="group.group"
                class="attr-group"
              >
                <p class="attr-group-heading">{{ group.group }}</p>
                <dl class="attributes-grid">
                  <template
                    v-for="item in group.items"
                    :key="item.label"
                  >
                    <dt class="attr-label">{{ item.label }}</dt>
                    <dd class="attr-value">{{ item.value }}</dd>
                  </template>
                </dl>
              </div>
            </template>
            <!-- Fallback -->
            <p v-else class="fallback-text">Information unavailable.</p>
          </div>
        </div>
      </div>
    </div>



    <!-- ── Section 1 : Wrapping ───────────────────────────────────────────── -->
    <div
      class="accordion-item"
      :class="{ 'is-open': openIndex === 1 }"
      role="listitem"
    >
      <button
        class="accordion-header"
        :aria-expanded="openIndex === 1"
        :aria-controls="`panel-${sections[1].id}`"
        :id="`header-${sections[1].id}`"
        @click="toggle(1)"
      >
        <span class="header-label">{{ sections[1].label }}</span>
        <svg
          class="chevron"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
          focusable="false"
        >
          <polyline points="6 9 12 15 18 9" />
        </svg>
      </button>

      <div
        class="accordion-body"
        :id="`panel-${sections[1].id}`"
        role="region"
        :aria-labelledby="`header-${sections[1].id}`"
      >
        <div class="accordion-inner">
          <div class="accordion-content">
            <template v-if="sections[1].hasContent()">
              <!-- Render packaging as a clean definition list, matching the
                   IKEA-style two-column grid used in Features & Dimensions -->
              <p class="attr-group-heading">Package size</p>
              <dl class="attributes-grid">
                <template v-if="product.packaging?.size">
                  <dt class="attr-label">Dimensions</dt>
                  <dd class="attr-value">{{ product.packaging.size }}</dd>
                </template>
                <template v-if="product.packaging?.packages != null">
                  <dt class="attr-label">Package number</dt>
                  <dd class="attr-value">{{ product.packaging.packages }}</dd>
                </template>
                <template v-if="product.packaging?.weight">
                  <dt class="attr-label">Package weight</dt>
                  <dd class="attr-value">{{ product.packaging.weight }}</dd>
                </template>
              </dl>
            </template>
            <p v-else class="fallback-text">Information unavailable.</p>
          </div>
        </div>
      </div>
    </div>



    <!-- ── Section 2 : Product Details ───────────────────────────────────── -->
    <div
      class="accordion-item"
      :class="{ 'is-open': openIndex === 2 }"
      role="listitem"
    >
      <button
        class="accordion-header"
        :aria-expanded="openIndex === 2"
        :aria-controls="`panel-${sections[2].id}`"
        :id="`header-${sections[2].id}`"
        @click="toggle(2)"
      >
        <span class="header-label">{{ sections[2].label }}</span>
        <svg
          class="chevron"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
          focusable="false"
        >
          <polyline points="6 9 12 15 18 9" />
        </svg>
      </button>

      <div
        class="accordion-body"
        :id="`panel-${sections[2].id}`"
        role="region"
        :aria-labelledby="`header-${sections[2].id}`"
      >
        <div class="accordion-inner">
          <div class="accordion-content">
            <template v-if="sections[2].hasContent()">
              <!-- Structured key/value pairs (Color, Article number, …) -->
              <dl
                v-if="product.productDetails?.length"
                class="attributes-grid"
              >
                <template
                  v-for="detail in product.productDetails"
                  :key="detail.label"
                >
                  <dt class="attr-label">{{ detail.label }}</dt>
                  <dd class="attr-value">{{ detail.value }}</dd>
                </template>
              </dl>

              <!-- Prose description below the key/value pairs -->
              <p
                v-if="product.description"
                class="description-text"
                :class="{ 'description-text--spaced': product.productDetails?.length }"
              >{{ product.description }}</p>
            </template>
            <p v-else class="fallback-text">Information unavailable.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/*
 * Design pattern: flat row list, no outer card.
 * Structure is defined entirely by hairline borders on each item.
 * Matches the IKEA-style reference image:
 *   – top border on the accordion wrapper
 *   – border-bottom on every item (last item gives the bottom edge)
 *   – bold labels, generous vertical padding, dark-only chevron
 */

/* ─── Accordion container ─────────────────────────────────────────────────── */
.accordion {
  margin-top: var(--spacing-6, 24px);
  border-top: 1px solid var(--color-light-grey, #e6e6e6);
  background: var(--color-white, #ffffff);
}

/* ─── Each accordion item — provides its own bottom border ───────────────── */
.accordion-item {
  border-bottom: 1px solid var(--color-light-grey, #e6e6e6);
}

/* ─── Header button ───────────────────────────────────────────────────────── */
.accordion-header {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--spacing-4, 16px);
  /* Tall rows match the reference: ~20px top + bottom */
  padding: var(--spacing-5, 20px) 0;
  background: transparent;
  border: none;
  cursor: pointer;
  text-align: left;
  font-family: var(--font-body, 'Roboto', sans-serif);
  -webkit-tap-highlight-color: transparent;
}

.accordion-header:focus-visible {
  outline: 2px solid var(--color-primary, #d95fa7);
  outline-offset: 2px;
  border-radius: var(--radius-sm, 4px);
}

/* ─── Header label — bold, same weight as reference ──────────────────────── */
.header-label {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-base, 16px);
  font-weight: var(--font-weight-bold, 700);
  color: var(--color-dark, #222222);
  line-height: var(--line-height-tight, 1.2);
}

/* ─── Chevron SVG — dark, rotates only, no color change ──────────────────── */
.chevron {
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  color: var(--color-dark, #222222);
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  will-change: transform;
}

.is-open .chevron {
  transform: rotate(180deg);
}

/* ─── Accordion body — CSS grid height trick ──────────────────────────────── */
/*
 * grid-template-rows: 0fr → 1fr animates height without JS.
 * .accordion-inner overflow:hidden clips the content at 0fr.
 * Rapid clicks simply reverse the in-flight transition — no jumpiness.
 */
.accordion-body {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  will-change: grid-template-rows;
}

.is-open .accordion-body {
  grid-template-rows: 1fr;
}

.accordion-inner {
  overflow: hidden;
  min-height: 0;
  /*
   * NO padding here — padding on a grid child is not clipped by the
   * 0fr row collapse. Padding lives in .accordion-content (nested)
   * so it is real inner content that overflow:hidden can fully clip.
   */
}

/* Inner wrapper: this is where spacing lives, fully inside the clip zone */
.accordion-content {
  padding-bottom: var(--spacing-5, 20px);
}

/* ─── Attribute groups (Features & Wrapping) ──────────────────────────────── */
.attr-group {
  /* Space between consecutive groups */
}

.attr-group + .attr-group {
  margin-top: var(--spacing-5, 20px);
}

/* Sub-heading above each dl: "Product size", "Weight", "Package size" */
.attr-group-heading {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: var(--font-weight-bold, 700);
  color: var(--color-dark, #222222);
  margin-bottom: var(--spacing-3, 12px);
}

/* ─── Two-column definition list shared by all three sections ─────────────── */
.attributes-grid {
  display: grid;
  grid-template-columns: minmax(130px, auto) 1fr;
  gap: var(--spacing-3, 12px) var(--spacing-8, 32px);
  list-style: none;
}

.attr-label {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: var(--font-weight-medium, 500);
  color: var(--color-grey, #5c5c5c);
  white-space: nowrap;
  line-height: var(--line-height-normal, 1.5);
}

.attr-value {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: var(--font-weight-regular, 400);
  color: var(--color-dark, #222222);
  line-height: var(--line-height-normal, 1.5);
}

/* ─── Wrapping — icon + text rows ────────────────────────────────────────── */
.wrapping-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: var(--spacing-4, 16px);
}

.wrapping-item {
  display: flex;
  align-items: flex-start;
  gap: var(--spacing-3, 12px);
}

.wrap-icon {
  flex-shrink: 0;
  width: 20px;
  height: 20px;
  margin-top: 2px;
  color: var(--color-dark, #222222);
}

.wrapping-item div {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.wrap-label {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-xs, 12px);
  font-weight: var(--font-weight-medium, 500);
  color: var(--color-grey, #5c5c5c);
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

.wrap-value {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: var(--font-weight-regular, 400);
  color: var(--color-dark, #222222);
  line-height: var(--line-height-normal, 1.5);
}

/* ─── Product Details — prose description ─────────────────────────────────── */
.description-text {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: var(--font-weight-regular, 400);
  color: var(--color-grey, #5c5c5c);
  line-height: var(--line-height-loose, 1.75);
}

/* Extra top gap when the description follows the productDetails dl */
.description-text--spaced {
  margin-top: var(--spacing-5, 20px);
}

/* ─── Fallback message ────────────────────────────────────────────────────── */
.fallback-text {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-style: italic;
  color: var(--color-grey, #5c5c5c);
}
</style>
