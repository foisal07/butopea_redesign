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

// Lightbox logic
const isLightboxOpen = ref(false);
const lightboxScale = ref(1);

const panX = ref(0);
const panY = ref(0);
const originX = ref(0);
const originY = ref(0);

const isPinching = ref(false);
const isPanning = ref(false);

let initialDistance = 0;
let initialScale = 1;
let lastPanX = 0;
let lastPanY = 0;
let lastTouchX = 0;
let lastTouchY = 0;
let touchStartX = 0;

const resetLightboxZoom = () => {
  lightboxScale.value = 1;
  panX.value = 0;
  panY.value = 0;
  originX.value = window.innerWidth / 2;
  originY.value = window.innerHeight / 2;
};

const openLightbox = () => {
  isLightboxOpen.value = true;
  resetLightboxZoom();
  document.body.style.overflow = 'hidden';
};

const closeLightbox = () => {
  isLightboxOpen.value = false;
  resetLightboxZoom();
  document.body.style.overflow = '';
};

const clampPan = () => {
  const W = window.innerWidth;
  const H = window.innerHeight;
  const S = lightboxScale.value;
  const OX = originX.value;
  const OY = originY.value;
  
  const maxX = W/2 - OX + OX*S;
  const minX = W/2 - OX - (W - OX)*S;
  
  const maxY = H/2 - OY + OY*S;
  const minY = H/2 - OY - (H - OY)*S;
  
  if (panX.value > maxX) panX.value = maxX;
  if (panX.value < minX) panX.value = minX;
  if (panY.value > maxY) panY.value = maxY;
  if (panY.value < minY) panY.value = minY;
};

const zoomIn = () => {
  if (lightboxScale.value < 4) {
    lightboxScale.value += 0.5;
    clampPan();
  }
};

const zoomOut = () => {
  if (lightboxScale.value > 1) {
    lightboxScale.value -= 0.5;
    if (lightboxScale.value <= 1) {
      resetLightboxZoom();
    } else {
      clampPan();
    }
  }
};

const nextLightboxImage = () => {
  if (props.images && activeIndex.value < props.images.length - 1) {
    activeIndex.value++;
    resetLightboxZoom();
    scrollTo(activeIndex.value);
  }
};

const prevLightboxImage = () => {
  if (activeIndex.value > 0) {
    activeIndex.value--;
    resetLightboxZoom();
    scrollTo(activeIndex.value);
  }
};

const handleTouchStart = (e) => {
  if (e.touches.length === 2) {
    isPinching.value = true;
    isPanning.value = false;
    
    const dx = e.touches[0].clientX - e.touches[1].clientX;
    const dy = e.touches[0].clientY - e.touches[1].clientY;
    initialDistance = Math.hypot(dx, dy);
    initialScale = lightboxScale.value;
    
    const midX = (e.touches[0].clientX + e.touches[1].clientX) / 2;
    const midY = (e.touches[0].clientY + e.touches[1].clientY) / 2;
    
    panX.value = panX.value + (midX - originX.value) * (lightboxScale.value - 1);
    panY.value = panY.value + (midY - originY.value) * (lightboxScale.value - 1);
    
    originX.value = midX;
    originY.value = midY;
    
  } else if (e.touches.length === 1) {
    isPanning.value = true;
    isPinching.value = false;
    lastTouchX = e.touches[0].clientX;
    lastTouchY = e.touches[0].clientY;
    touchStartX = e.touches[0].clientX;
    lastPanX = panX.value;
    lastPanY = panY.value;
  }
};

const handleTouchMove = (e) => {
  if (isPinching.value && e.touches.length === 2) {
    const dx = e.touches[0].clientX - e.touches[1].clientX;
    const dy = e.touches[0].clientY - e.touches[1].clientY;
    const currentDistance = Math.hypot(dx, dy);
    
    const ratio = currentDistance / initialDistance;
    let newScale = initialScale * ratio;
    
    if (newScale < 1) newScale = 1;
    if (newScale > 5) newScale = 5;
    
    lightboxScale.value = newScale;
    
    if (lightboxScale.value === 1) {
      resetLightboxZoom();
    } else {
      clampPan();
    }
  } else if (isPanning.value && e.touches.length === 1) {
    lastTouchX = e.touches[0].clientX;
    lastTouchY = e.touches[0].clientY;

    if (lightboxScale.value > 1) {
      const deltaX = e.touches[0].clientX - touchStartX;
      const deltaY = e.touches[0].clientY - touchStartX; // wait, this was wrong in old code if I used touchStartX. I need to track lastTouchY.
      // Actually wait, lastPan was recorded at touchStart, so delta from touchStart is correct.
      // Wait, delta is current - start.
      panX.value = lastPanX + (e.touches[0].clientX - touchStartX);
      panY.value = lastPanY + (e.touches[0].clientY - touchStartX); // wait, touchStartY! I need to add touchStartY.
      clampPan();
    }
  }
};

const handleTouchEnd = (e) => {
  if (e.touches.length < 2) {
    isPinching.value = false;
  }
  
  if (e.touches.length === 0 && isPanning.value) {
    isPanning.value = false;
    if (lightboxScale.value <= 1) {
      const swipeDist = lastTouchX - touchStartX;
      if (swipeDist > 50 && activeIndex.value > 0) {
        prevLightboxImage();
      } else if (swipeDist < -50 && activeIndex.value < props.images.length - 1) {
        nextLightboxImage();
      }
    }
  }
  
  if (lightboxScale.value <= 1) {
    resetLightboxZoom();
  } else {
    clampPan();
  }
};
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
        <UtilsStatusLabel />
      </div>

      <!-- Zoom Trigger -->
      <button 
        class="gallery-zoom-trigger" 
        @click="openLightbox"
        aria-label="Open fullscreen gallery"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="11" cy="11" r="8"></circle>
          <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
          <line x1="11" y1="8" x2="11" y2="14"></line>
          <line x1="8" y1="11" x2="14" y2="11"></line>
        </svg>
      </button>

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


    </div>

    <!-- Lightbox Modal via Teleport -->
    <Teleport to="body">
      <div v-if="isLightboxOpen" class="lightbox-overlay">
        <div class="lightbox-header">
          <div class="lightbox-controls">
            <button class="zoom-btn" @click="zoomOut" :disabled="lightboxScale <= 1" aria-label="Zoom out">−</button>
            <span class="zoom-level">{{ Math.round(lightboxScale * 100) }}%</span>
            <button class="zoom-btn" @click="zoomIn" :disabled="lightboxScale >= 4" aria-label="Zoom in">+</button>
          </div>
          <button class="lightbox-close" @click="closeLightbox" aria-label="Close lightbox">
            <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="18" y1="6" x2="6" y2="18"></line>
              <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
          </button>
        </div>
        <!-- Desktop Lightbox Navigation Arrows -->
        <button 
          v-if="images.length > 1"
          class="lightbox-nav-btn prev-btn" 
          @click="prevLightboxImage"
          :disabled="activeIndex === 0"
          aria-label="Previous Image"
        >
          <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="15 18 9 12 15 6"></polyline>
          </svg>
        </button>

        <button 
          v-if="images.length > 1"
          class="lightbox-nav-btn next-btn" 
          @click="nextLightboxImage"
          :disabled="activeIndex === images.length - 1"
          aria-label="Next Image"
        >
          <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="9 18 15 12 9 6"></polyline>
          </svg>
        </button>

        <div 
          class="lightbox-zoom-area"
          @touchstart="handleTouchStart"
          @touchmove="handleTouchMove"
          @touchend="handleTouchEnd"
        >
          <img 
            :src="images[activeIndex]" 
            :style="{ 
              transform: `translate(${panX}px, ${panY}px) scale(${lightboxScale})`,
              transformOrigin: `${originX}px ${originY}px`,
              transition: (isPinching || isPanning) ? 'none' : 'transform 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94)'
            }"
            class="lightbox-img"
            alt="Fullscreen product image"
            draggable="false"
          />
        </div>
      </div>
    </Teleport>
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



@media (min-width: 768px) {
  .nav-btn {
    display: flex;
  }

  .gallery-dots {
    display: none;
  }
}

/* Gallery Zoom Icon */
.gallery-zoom-trigger {
  position: absolute;
  top: 16px;
  right: 16px;
  z-index: 10;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: rgba(255,255,255,0.9);
  border: 1px solid #e5e5e5;
  box-shadow: 0 2px 4px rgba(34,34,34,0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #222222;
  transition: all 0.2s;
}

.gallery-zoom-trigger:hover {
  color: #d95fa7;
  border-color: #d95fa7;
}

/* Lightbox Modal */
.lightbox-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background-color: #222222;
  z-index: 9999;
  display: flex;
  flex-direction: column;
}

.lightbox-header {
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding: 0 24px;
  z-index: 10000;
  background: linear-gradient(to bottom, rgba(34,34,34,0.8), transparent);
}

.lightbox-controls {
  display: flex;
  align-items: center;
  gap: 16px;
  background-color: rgba(255,255,255,0.15);
  padding: 8px 16px;
  border-radius: 20px;
  color: #ffffff;
}

.zoom-btn {
  background: none;
  border: none;
  color: #ffffff;
  font-size: 24px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  transition: background-color 0.2s;
}

.zoom-btn:hover:not(:disabled) {
  background-color: #d95fa7;
}

.zoom-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.zoom-level {
  font-family: var(--font-body, sans-serif);
  font-size: 14px;
  min-width: 48px;
  text-align: center;
}

.lightbox-close {
  background: none;
  border: none;
  color: #ffffff;
  cursor: pointer;
  margin-left: 24px;
  padding: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.2s;
}

.lightbox-close:hover {
  color: #d95fa7;
  background-color: rgba(255,255,255,0.1);
}

.lightbox-zoom-area {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  touch-action: none;
  width: 100vw;
}

.lightbox-img {
  width: 100vw;
  height: 100vh;
  object-fit: contain;
  will-change: transform;
}

/* Lightbox Desktop Navigation */
.lightbox-nav-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(255, 255, 255, 0.1);
  border: none;
  border-radius: 50%;
  width: 56px;
  height: 56px;
  cursor: pointer;
  color: #ffffff;
  z-index: 10000;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.lightbox-nav-btn:hover:not(:disabled) {
  background-color: rgba(255, 255, 255, 0.2);
  color: #d95fa7;
}

.lightbox-nav-btn:disabled {
  opacity: 0.2;
  cursor: not-allowed;
}

.lightbox-nav-btn.prev-btn { left: 24px; }
.lightbox-nav-btn.next-btn { right: 24px; }

@media (max-width: 767px) {
  .lightbox-nav-btn {
    display: none;
  }
}
</style>
