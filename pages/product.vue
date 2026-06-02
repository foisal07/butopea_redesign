<script setup>
import img1 from '~/assets/images/image_1.png';
import img2 from '~/assets/images/image_2.jpg';
import img3 from '~/assets/images/image_3.jpg';

const productImages = [img1, img2, img3];
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

      <!-- 5. USP strip: free shipping, delivery date, why Butopea, free returns -->
      <ProductTrustBadges />

      <!-- 6. Product specification accordion -->
      <ProductDetails />

      <!-- 7. Customer reviews -->
      <ProductReviewsSection />
    </div>

    <!-- 8. Sticky ATC bar (appears when main ATC scrolls out of view) -->
    <ProductStickyCartBar />
  </main>
</template>
