<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const props = defineProps({
  images: {
    type: Array,
    default: () => []
  }
});

const scrollContainer = ref(null);
const activeIndex = ref(0);

// Handle manual scroll to update activeIndex
const handleScroll = () => {
  if (!scrollContainer.value || !props.images || props.images.length === 0) return;
  const container = scrollContainer.value;
  const scrollLeft = container.scrollLeft;
  const itemWidth = container.clientWidth;
  
  if (itemWidth === 0) return;
  activeIndex.value = Math.round(scrollLeft / itemWidth);
};

// Scroll to specific index via click (Desktop navigation)
const scrollTo = (index) => {
  if (!props.images || index < 0 || index >= props.images.length || !scrollContainer.value) return;
  const container = scrollContainer.value;
  const itemWidth = container.clientWidth;
  
  container.scrollTo({
    left: itemWidth * index,
    behavior: 'smooth'
  });
};

// Handle resize edge case (syncing visual scroll with activeIndex)
let resizeTimeout = null;
const handleResize = () => {
  clearTimeout(resizeTimeout);
  resizeTimeout = setTimeout(() => {
    if (!scrollContainer.value || !props.images || props.images.length === 0) return;
    const container = scrollContainer.value;
    const itemWidth = container.clientWidth;
    container.scrollTo({
      left: itemWidth * activeIndex.value,
      behavior: 'instant'
    });
  }, 100);
};

onMounted(() => {
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
  clearTimeout(resizeTimeout);
});
</script>

<template>
  <div class="product-gallery">
    <!-- Edge Case 1: Fallback State -->
    <div v-if="!images || images.length === 0" class="gallery-fallback">
      <div class="fallback-icon">
        <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
          <circle cx="8.5" cy="8.5" r="1.5"></circle>
          <polyline points="21 15 16 10 5 21"></polyline>
        </svg>
      </div>
      <p>No images available</p>
    </div>

    <!-- Gallery Container -->
    <div v-else class="gallery-container">
      <!-- Status Label overlay -->
      <div class="gallery-status-overlay">
        <UtilsStatusLabel text="Available" />
      </div>

      <!-- Scroll snapping container for mobile swiping -->
      <div 
        class="gallery-slider" 
        ref="scrollContainer"
        @scroll="handleScroll"
      >
        <div 
          v-for="(image, index) in images" 
          :key="index"
          class="gallery-slide"
        >
          <img :src="image" :alt="`Product image ${index + 1}`" class="gallery-image" />
        </div>
      </div>

      <!-- Desktop Navigation Arrows -->
      <button 
        v-if="images.length > 1"
        class="nav-btn prev-btn" 
        @click="scrollTo(activeIndex - 1)"
        :disabled="activeIndex === 0"
        aria-label="Previous Image"
      >
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="15 18 9 12 15 6"></polyline>
        </svg>
      </button>

      <button 
        v-if="images.length > 1"
        class="nav-btn next-btn" 
        @click="scrollTo(activeIndex + 1)"
        :disabled="activeIndex === images.length - 1"
        aria-label="Next Image"
      >
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="9 18 15 12 9 6"></polyline>
        </svg>
      </button>

      <!-- Mobile Dots Navigation -->
      <div v-if="images.length > 1" class="gallery-dots">
        <button
          v-for="(_, index) in images"
          :key="`dot-${index}`"
          class="dot-btn"
          :class="{ 'is-active': index === activeIndex }"
          @click="scrollTo(index)"
          :aria-label="`Go to image ${index + 1}`"
        ></button>
      </div>

      <!-- Desktop Thumbnails Navigation -->
      <div v-if="images.length > 1" class="gallery-thumbnails">
        <button
          v-for="(image, index) in images"
          :key="`thumb-${index}`"
          class="thumbnail-btn"
          :class="{ 'is-active': index === activeIndex }"
          @click="scrollTo(index)"
          :aria-label="`Go to image ${index + 1}`"
        >
          <img :src="image" alt="" />
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.product-gallery {
  width: 100%;
  --nav-btn-size: 40px;
}

.gallery-fallback {
  width: 100%;
  aspect-ratio: 1 / 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: var(--color-surface, #f9f9f9);
  border: 2px dashed var(--color-border, #e5e5e5);
  border-radius: var(--radius-lg, 8px);
  color: var(--color-secondary, #6b7280);
}

.fallback-icon {
  margin-bottom: 12px;
  opacity: 0.6;
}

.gallery-container {
  position: relative;
  width: 100%;
}

.gallery-status-overlay {
  position: absolute;
  top: 16px;
  left: 16px;
  z-index: 10;
}

.gallery-slider {
  display: flex;
  width: 100%;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE and Edge */
  border-radius: var(--radius-lg, 8px);
  background-color: var(--color-surface, #f9f9f9);
}

.gallery-slider::-webkit-scrollbar {
  display: none; /* Chrome, Safari and Opera */
}

.gallery-slide {
  flex: 0 0 100%;
  width: 100%;
  scroll-snap-align: center;
  scroll-snap-stop: always;
  display: flex;
  align-items: center;
  justify-content: center;
}

.gallery-image {
  width: 100%;
  height: auto;
  object-fit: contain;
  display: block;
}

/* Navigation Buttons - Hidden on mobile, shown on desktop */
.nav-btn {
  display: none;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: var(--nav-btn-size);
  height: var(--nav-btn-size);
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.95);
  border: 1px solid var(--color-border, #e5e5e5);
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  cursor: pointer;
  z-index: 10;
  align-items: center;
  justify-content: center;
  color: var(--color-primary, #111827);
  transition: all 0.2s ease;
}

.nav-btn:hover:not(:disabled) {
  background-color: #ffffff;
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

.nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  box-shadow: none;
}

.prev-btn {
  left: 16px;
}

.next-btn {
  right: 16px;
}

/* Mobile Dots */
.gallery-dots {
  position: absolute;
  bottom: 16px;
  left: 0;
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 8px;
  margin: 0;
  z-index: 10;
}

.dot-btn {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: var(--color-border, #ccc);
  border: none;
  padding: 0;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.dot-btn.is-active {
  background-color: var(--color-primary, #111827);
}

/* Thumbnails */
.gallery-thumbnails {
  display: none; /* Hidden on mobile */
  justify-content: flex-start;
  gap: 8px;
  margin-top: 16px;
  overflow-x: auto;
  padding-bottom: 8px;
  scrollbar-width: none;
}

.gallery-thumbnails::-webkit-scrollbar {
  display: none;
}

.thumbnail-btn {
  width: 64px;
  height: 64px;
  flex-shrink: 0;
  padding: 0;
  border: 2px solid transparent;
  border-radius: var(--radius-sm, 6px);
  overflow: hidden;
  cursor: pointer;
  transition: all 0.2s ease;
  background: var(--color-surface, #f9f9f9);
}

.thumbnail-btn img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  opacity: 0.6;
  transition: opacity 0.2s ease;
}

.thumbnail-btn:hover img {
  opacity: 1;
}

.thumbnail-btn.is-active {
  border-color: var(--color-primary, #111827);
}

.thumbnail-btn.is-active img {
  opacity: 1;
}

/* Media Queries for Desktop */
@media (min-width: 768px) {
  .nav-btn {
    display: flex;
  }

  .gallery-dots {
    display: none;
  }

  .gallery-thumbnails {
    display: flex;
  }
}
</style>
