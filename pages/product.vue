<script setup>
import img1 from '~/assets/images/image_1.png';
import img2 from '~/assets/images/image_2.jpg';
import img3 from '~/assets/images/image_3.jpg';
import mockData from '~/data/product.mock.json';

const productImages = [img1, img2, img3];
const productPrice = mockData.price;
</script>

<template>
  <main>
    <!-- 1. Image gallery with zoom, thumbnails, availability label -->
    <ProductImageGallery :images="productImages" />

    <div class="container">
      <!-- 2. Product title, subtitle, price -->
      <ProductInfo />

      <!-- 4. Quantity + Add to Cart button -->
      <ProductAddToCart />

      <!-- 5. Shipment info: free shipping eligibility + dynamic delivery window -->
      <ProductShipment :price="productPrice" />

      <!-- 6. USP strip: free shipping, delivery date, why Butopea, free returns -->
      <ProductTrustBadges />

      <!-- 6. Product specification accordion -->
      <ProductDetails />

      <!-- 7. Customer reviews -->
      <ProductReviewsSection />
    </div>

    <!-- 8. Sticky ATC bar (appears when main ATC scrolls out of view) --> 
        <ProductStickyCartBar
      :price="mockData.price"
      :currency="mockData.currency"
      :stock-count="mockData.stockCount"
    /> 

  </main>
</template>

<style scoped>
/*
  Clear the fixed-position StickyCartBar (≈72 px on mobile, ≈84 px on tablet).
  Using a permanent bottom padding is the simplest approach: the extra whitespace
  is invisible when the bar is hidden, and prevents the bar from ever covering
  the last element on the page when it is visible.
*/
main {
  padding-bottom: 72px;
}

@media (min-width: 768px) {
  main {
    padding-bottom: 84px;
  }
}
</style>
