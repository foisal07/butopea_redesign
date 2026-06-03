<template>
  <div class="product-info">
    <div v-if="!productData" class="skeleton-container">
      <div class="skeleton-title"></div>
      <div class="skeleton-subtitle"></div>
      <div class="skeleton-price"></div>
    </div>
    
    <div v-else class="info-content">
      <h1 class="title">
        {{ productData.name }}
      </h1>
      <p class="subtitle-container">
        <span class="subtitle">{{ productData.subtitle }}</span><template v-if="formattedSize"><span class="title-size">{{ formattedSize }}</span></template>
      </p>
      
      <div class="price-container">
        <span class="price-value">{{ formattedPriceValue }}</span><sup class="price-currency">{{ formattedCurrency }}</sup>
        <span class="price-note">including VAT</span>
      </div>
            <ProductVariantSelector :colors="productColors" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import mockData from '~/data/product.mock.json'

// ASSUMPTION: Product data is loaded locally from the JSON file.
const productData = ref(mockData)
const productColors = mockData.variants?.colors ?? [];

const formattedSize = computed(() => {
  if (!productData.value) return ''
  const attributes = productData.value.attributes || []
  const sizeGroup = attributes.find(g => g.group === 'Product size')
  if (!sizeGroup) return ''
  
  const items = sizeGroup.items || []
  const getVal = (label) => {
    const item = items.find(i => i.label === label)
    return item ? Math.round(parseFloat(item.value)) : ''
  }
  
  const length = getVal('Length')
  const width = getVal('Width')
  const height = getVal('Height')
  
  const parts = []
  if (length) parts.push(`${length}cm`)
  if (width) parts.push(`${width}cm`)
  if (height) parts.push(`${height}cm`)
  
  if (parts.length > 0) {
    return ` ${parts.join(' x ')}`
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
  padding: var(--spacing-6) 0 0;
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
  font-size: var(--font-size-base);
  line-height: var(--line-height-tight);
  color: var(--color-dark);
  font-weight: var(--font-weight-bold);
  text-transform: uppercase;
}

.title-size {
  text-transform: lowercase;
  text-decoration: underline;
}

.subtitle-container {
  margin: 0 0 var(--spacing-4) 0;
  font-family: var(--font-body);
  font-size: var(--font-size-md);
  line-height: var(--line-height-normal);
  color: var(--color-grey);
  font-weight: var(--font-weight-regular);
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
  align-self: flex-end;
  margin: 0 0 4px var(--spacing-2);
  font-family: var(--font-body);
  font-size: var(--font-size-xs);
  color: var(--color-grey);
  font-weight: var(--font-weight-regular);
  white-space: nowrap;
}

/* Tablet and Desktop scaling */
@media (min-width: 768px) {
  .product-info {
    padding: var(--spacing-6) 0 0;
  }
  
  .title {
    font-size: var(--font-size-lg);
  }
}
</style>
