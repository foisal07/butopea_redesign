<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// ─── Data Fetching ────────────────────────────────────────────────────────────
// Import the mock JSON statically so it is bundled, then simulate a network
// round-trip with a small setTimeout so the v-if loading guard is exercised.
// Swap out the setTimeout block for a real fetch() / useFetch() call in prod.
import rawMock from '~/data/product.mock.json'
import img1 from '~/assets/images/image_1.png'
import img2 from '~/assets/images/image_2.jpg'
import img3 from '~/assets/images/image_3.jpg'

/** @type {import('vue').Ref<typeof rawMock | null>} */
const productData = ref(null)
const galleryImages = ref([])

onMounted(() => {
  // Simulates ~300 ms network latency so the skeleton is visible on load.
  setTimeout(() => {
    productData.value = rawMock
    galleryImages.value = [img1, img2, img3]
  }, 300)
})

// ─── StickyCartBar mount guard ────────────────────────────────────────────────
// StickyCartBar uses <Teleport to="body">, so CSS display:none on a wrapper
// div does NOT prevent the teleported content from rendering. We must use v-if
// so the component is never mounted (and its IntersectionObserver never starts)
// on viewports ≥ 768 px where the right column's AddToCart is always visible.
const isMobile = ref(true) // safe default: assume mobile until DOM is available

function updateIsMobile() {
  isMobile.value = window.innerWidth < 768
}

onMounted(() => {
  updateIsMobile()
  window.addEventListener('resize', updateIsMobile, { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('resize', updateIsMobile)
})
</script>

<template>
  <main class="product-page">

    <!-- ── LOADING SKELETON ─────────────────────────────────────────────────── -->
    <div v-if="!productData" class="product-grid skeleton-grid" aria-busy="true" aria-label="Loading product">
      <!-- Mobile DOM Order -->
      <div class="skeleton skeleton--gallery grid-item-gallery" />
      
      <!-- Right column components interleave here on mobile -->
      <div class="grid-item-info">
         <div class="skeleton skeleton--title" />
         <div class="skeleton skeleton--subtitle" />
         <div class="skeleton skeleton--price" />
         <div class="skeleton skeleton--swatch-row" />
      </div>
      <div class="skeleton skeleton--button grid-item-addtocart" />
      <div class="skeleton skeleton--shipment grid-item-shipment" />
      <div class="skeleton skeleton--trust grid-item-trust" />

      <!-- Left column resumes -->
      <div class="skeleton-accordion grid-item-details">
        <div class="skeleton skeleton--accordion-row" />
        <div class="skeleton skeleton--accordion-row" />
        <div class="skeleton skeleton--accordion-row" />
      </div>
      <div class="skeleton skeleton--reviews grid-item-reviews" />
    </div>

    <!-- ── REAL CONTENT ──────────────────────────────────────────────────────── -->
    <div v-else class="product-grid" role="main">
      <!-- 1. Image gallery: zoom, thumbnails, availability label -->
      <div class="grid-item-gallery">
        <ProductImageGallery :images="galleryImages" />
      </div>

      <!-- Right column group (DOM order: immediately after gallery for mobile) -->
      <div class="grid-item-right-sticky">
        <div class="right-sticky-inner">
            <div class="grid-item-info">
              <!-- 4. Product title, subtitle, price (+ inline VariantSelector) -->
              <ProductInfo />
            </div>

            <div class="grid-item-addtocart">
              <!-- 5. Quantity + Add to Cart button (anchor for IntersectionObserver) -->
              <ProductAddToCart />
            </div>

            <div class="grid-item-shipment">
              <!-- 6. Shipment info: free shipping + dynamic delivery window -->
              <ProductShipment :price="productData.price" />
            </div>

            <div class="grid-item-trust">
              <!-- 7. USP strip: why Butopea, free returns -->
              <ProductTrustBadges />
            </div>
        </div>
      </div>

      <!-- 2. Product specification accordion -->
      <div class="grid-item-details">
        <ProductDetails />
      </div>

      <!-- 3. Customer reviews -->
      <div class="grid-item-reviews">
        <ProductReviewsSection />
      </div>
    </div>

    <!-- ── STICKY CART BAR ───────────────────────────────────────────────────── -->
    <ProductStickyCartBar
      v-if="isMobile"
      :price="productData ? productData.price : null"
      :currency="productData ? productData.currency : 'HUF'"
      :stock-count="productData ? productData.stockCount : 0"
    />

  </main>
</template>

<style scoped>
/* ─── Page shell ──────────────────────────────────────────────────────────── */
.product-page {
  padding-bottom: 80px;
}

/* ─── Flat Grid Layout ────────────────────────────────────────────────────── */
.product-grid {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-2); /* Reduced gap to bring components closer */
  width: 100%;
}

@media (max-width: 767px) {
  .grid-item-right-sticky,
  .grid-item-details,
  .grid-item-reviews,
  /* Skeletons */
  .grid-item-info,
  .grid-item-addtocart,
  .grid-item-shipment,
  .grid-item-trust,
  .skeleton-accordion,
  .skeleton--reviews {
    padding-inline: var(--spacing-2); /* sm padding for left/right */
  }
}

/* Mobile ordering is handled by DOM order:
   Gallery -> Right-Sticky-Group -> Details -> Reviews */

@media (min-width: 768px) {
  .product-page {
    padding-bottom: 0;
  }

  .product-grid {
    display: grid;
    grid-template-columns: 60fr 40fr;
    gap: 2.5rem;
    max-width: var(--max-width-container);
    margin-inline: auto;
    padding-inline: var(--page-padding-tablet);
    /*
      Do NOT set align-items: start here.
      The grid items must stretch to allow position: sticky to work within them.
    */
  }

  /* Place left column items explicitly in column 1 */
  .grid-item-gallery { 
    grid-column: 1; 
    grid-row: 1; 
    padding-top: var(--spacing-10); /* larger padding-top for desktop */
  }
  .grid-item-details { grid-column: 1; grid-row: 2; }
  .grid-item-reviews { grid-column: 1; grid-row: 3; }

  /* Place the entire right column group in column 2, spanning all rows */
  .grid-item-right-sticky {
    grid-column: 2;
    grid-row: 1 / span 3; 
    /* Spanning ensures the right column container is as tall as the whole left side, giving sticky room to travel */
  }
}

/* ─── Sticky right panel ──────────────────────────────────────────────────── */
@media (min-width: 768px) {
  .right-sticky-inner {
    position: sticky;
    top: 2rem;
    align-self: start;
    display: flex;
    flex-direction: column;
    gap: 0; /* Components provide their own spacing usually, or add here if needed */
  }
}

/* ─── Loading skeletons ───────────────────────────────────────────────────── */
.skeleton {
  border-radius: 6px;
  background: linear-gradient(
    90deg,
    var(--color-light-grey, #e6e6e6) 25%,
    #f0f0f0 50%,
    var(--color-light-grey, #e6e6e6) 75%
  );
  background-size: 200% 100%;
  animation: skeleton-shimmer 1.6s ease-in-out infinite;
}

@keyframes skeleton-shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.skeleton--gallery { width: 100%; aspect-ratio: 1 / 1; }
.skeleton-accordion { margin-top: 1.5rem; display: flex; flex-direction: column; gap: 1px; }
.skeleton--accordion-row { height: 60px; }
.skeleton--reviews { margin-top: 1.5rem; height: 220px; }

/* Right column skeletons */
.grid-item-info { display: flex; flex-direction: column; gap: 0.75rem; padding-top: 0; }
.skeleton--title    { height: 32px; width: 55%; }
.skeleton--subtitle { height: 20px; width: 85%; }
.skeleton--price    { height: 52px; width: 45%; margin-top: 0.25rem; }
.skeleton--swatch-row { height: 28px; width: 60%; }
.skeleton--button   { height: 48px; margin-top: 0.25rem; }
.skeleton--shipment { height: 68px; margin-top: 0.25rem; }
.skeleton--trust    { height: 88px; }

@media (min-width: 768px) {
  .skeleton-grid {
    display: grid;
    grid-template-columns: 60fr 40fr;
    gap: 2.5rem;
  }
}

@media (min-width: 1024px) {
  .product-grid {
    padding-inline: var(--page-padding-desktop);
  }
}
</style>
