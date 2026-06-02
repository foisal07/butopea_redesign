<script setup>
</script>

<template>
  <main>
    <!-- 1. Image gallery with zoom, thumbnails, availability label -->
    <ProductImageGallery />

    <!-- 2. Product title, subtitle, price -->
    <ProductInfo />

    <!-- 3. Colour / variant selector -->
    <ProductVariantSelector />

    <!-- 4. Quantity + Add to Cart button -->
    <ProductAddToCart />

    <!-- 5. USP strip: free shipping, delivery date, why Butopea, free returns -->
    <ProductTrustBadges />

    <!-- 6. Product specification accordion -->
    <ProductDetails />

    <!-- 7. Customer reviews -->
    <ProductReviewsSection />

    <!-- 8. Sticky ATC bar (appears when main ATC scrolls out of view) -->
    <ProductStickyCartBar />
  </main>
</template>
