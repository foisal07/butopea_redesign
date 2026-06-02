<template>
  <div class="product-info">
    <div v-if="!productData" class="skeleton-container">
      <div class="skeleton-title"></div>
      <div class="skeleton-subtitle"></div>
      <div class="skeleton-price"></div>
    </div>
    
    <div v-else class="info-content">
      <h1 class="title">{{ productData.name }}</h1>
      <p class="subtitle-container">
        <span class="subtitle">{{ productData.subtitle }}</span><template v-if="formattedSize">,<span class="size">{{ formattedSize }}</span></template>
      </p>
      
      <div class="price-container">
        <span class="price-value">{{ formattedPriceValue }}</span><sup class="price-currency">{{ formattedCurrency }}</sup>
      </div>
      <p class="price-note">Price, including VAT</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import mockData from '~/data/product.mock.json'

// ASSUMPTION: Product data is loaded locally from the JSON file.
const productData = ref(mockData)

const formattedSize = computed(() => {
  if (!productData.value) return ''
  const attrs = productData.value.attributes || []
  
  const width = attrs.find(a => a.label === 'Szélesség')?.value.replace(/[^0-9]/g, '')
  const depth = attrs.find(a => a.label === 'Mélység')?.value.replace(/[^0-9]/g, '')
  const height = attrs.find(a => a.label === 'Magasság')?.value.replace(/[^0-9]/g, '')
  
  if (width || depth || height) {
    return ` ${width}x${depth}x${height}cm`
  }
  return ''
})

const formattedPriceValue = computed(() => {
  if (!productData.value || typeof productData.value.price !== 'number') return ''
  return new Intl.NumberFormat('hu-HU').format(productData.value.price)
})

const formattedCurrency = computed(() => {
  if (!productData.value) return ''
  return productData.value.currency === 'HUF' ? 'Ft' : productData.value.currency
})
</script>

<style scoped>
.product-info {
  padding: var(--spacing-6) 0; 
}

/* Skeletons */
.skeleton-container {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-2);
}

.skeleton-title {
  height: 28px;
  width: 60%;
  background-color: var(--color-light-grey);
  border-radius: var(--radius-sm);
  animation: pulse 1.5s infinite;
}

.skeleton-subtitle {
  height: 20px;
  width: 90%;
  background-color: var(--color-light-grey);
  border-radius: var(--radius-sm);
  animation: pulse 1.5s infinite;
}

.skeleton-price {
  height: 48px;
  width: 40%;
  background-color: var(--color-light-grey);
  border-radius: var(--radius-sm);
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 0.3; }
  100% { opacity: 0.6; }
}

/* Real Content */
.info-content {
  display: flex;
  flex-direction: column;
}

.title {
  margin: 0 0 var(--spacing-1) 0;
  font-family: var(--font-display);
  font-size: var(--font-size-xl);
  line-height: var(--line-height-tight);
  color: var(--color-dark);
  font-weight: var(--font-weight-bold);
  text-transform: uppercase;
}

.subtitle-container {
  margin: 0 0 var(--spacing-4) 0;
  font-family: var(--font-body);
  font-size: var(--font-size-lg);
  line-height: var(--line-height-normal);
  color: var(--color-dark);
  font-weight: var(--font-weight-regular);
}

.size {
  text-decoration: underline;
}

.price-container {
  margin: 0;
  display: flex;
  align-items: flex-start;
}

.price-value {
  font-family: var(--font-body);
  font-size: var(--font-size-3xl);
  font-weight: var(--font-weight-bold);
  color: var(--color-dark);
  line-height: 1;
}

.price-currency {
  font-family: var(--font-body);
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-bold);
  color: var(--color-dark);
  margin-left: 2px;
  line-height: 1;
  vertical-align: super;
}

.price-note {
  margin: var(--spacing-2) 0 0 0;
  font-family: var(--font-body);
  font-size: var(--font-size-sm);
  color: var(--color-grey);
  font-weight: var(--font-weight-regular);
}

/* Tablet and Desktop scaling */
@media (min-width: 768px) {
  .product-info {
    padding: var(--spacing-6) 0;
  }
  
  .title {
    font-size: var(--font-size-2xl);
  }
}
</style>
