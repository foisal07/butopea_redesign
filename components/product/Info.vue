<script setup>
</script>

<template>
  <div id="product-info-verify" class="product-info-verify">
    <p class="label">ProductInfo</p>
    <p class="sub">Variables verified ✓</p>
  </div>
</template>

<style scoped>
/* VERIFICATION BLOCK — proves CSS variables from main.css are globally available */
.product-info-verify {
  background-color: var(--color-white);
  border: var(--border-default);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-card);
  padding: var(--spacing-4);
  margin: var(--spacing-2);
}

.label {
  font-family: var(--font-display);
  font-size: var(--font-size-lg);
  color: var(--color-primary);
  font-weight: var(--font-weight-bold);
}

.sub {
  font-family: var(--font-body);
  font-size: var(--font-size-sm);
  color: var(--color-secondary);
  margin-top: var(--spacing-1);
}
</style>
