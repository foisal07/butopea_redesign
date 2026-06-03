<template>
  <div id="main-add-to-cart" class="add-to-cart-container">
    <!-- Add to Cart Button -->
    <button class="add-to-cart-btn" @click="addToCart">
      Add to Cart
    </button>

    <!-- Toast Notification -->
    <Teleport to="body">
      <Transition name="toast-fade">
        <div v-if="showToast" class="toast-notification" role="alert">
          Successfully added to cart!
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showToast = ref(false)
let toastTimeout = null

const addToCart = () => {
  // Edge Case 1: Spam-clicking resets the timer and prevents overlap
  if (toastTimeout) {
    clearTimeout(toastTimeout)
  }
  
  showToast.value = true
  
  toastTimeout = setTimeout(() => {
    showToast.value = false
    toastTimeout = null
  }, 3000)
}
</script>

<style scoped>
.add-to-cart-container {
  display: flex;
  align-items: center;
  margin-top: var(--spacing-2, 8px);
  width: 100%;
}

.add-to-cart-btn {
  flex: 1;
  height: 48px;
  background-color: var(--color-primary, #d95fa7);
  color: var(--color-white, #ffffff);
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-base, 16px);
  font-weight: var(--font-weight-bold, 700);
  border-radius: var(--radius-md, 8px);
  transition: background-color var(--transition-base, 0.25s ease), transform 0.1s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.add-to-cart-btn:hover {
  background-color: var(--color-primary-hover, #ff40a7);
}

.add-to-cart-btn:active {
  transform: scale(0.98);
}

.toast-notification {
  position: fixed;
  top: var(--spacing-6, 24px);
  left: 50%;
  transform: translateX(-50%);
  background-color: var(--color-secondary, #2ebd96);
  color: var(--color-white, #ffffff);
  padding: var(--spacing-3, 12px) var(--spacing-6, 24px);
  border-radius: var(--radius-full, 9999px);
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: var(--font-size-sm, 14px);
  font-weight: var(--font-weight-bold, 700);
  box-shadow: var(--shadow-modal, 0 8px 32px rgba(0,0,0,0.18));
  z-index: var(--z-toast, 400);
  display: flex;
  align-items: center;
  gap: var(--spacing-2, 8px);
}

.toast-fade-enter-active,
.toast-fade-leave-active {
  transition: opacity var(--transition-base, 0.25s ease), transform var(--transition-base, 0.25s ease);
}

.toast-fade-enter-from,
.toast-fade-leave-to {
  opacity: 0;
  transform: translate(-50%, -20px);
}
</style>
