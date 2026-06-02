<template>
  <div id="main-add-to-cart" class="add-to-cart-container">
    <!-- Quantity Selector -->
    <div class="quantity-selector">
      <button 
        class="qty-btn" 
        @click="decrement" 
        :disabled="quantity <= 1"
        aria-label="Decrease quantity"
      >
        -
      </button>
      <span class="qty-display">{{ quantity }}</span>
      <button 
        class="qty-btn" 
        @click="increment" 
        :disabled="quantity >= stockCount"
        aria-label="Increase quantity"
      >
        +
      </button>
    </div>

    <!-- Add to Cart Button -->
    <button class="add-to-cart-btn" @click="addToCart">
      Add to Cart
    </button>

    <!-- Toast Notification -->
    <Teleport to="body">
      <Transition name="toast-fade">
        <div v-if="showToast" class="toast-notification" role="alert">
          Successfully added {{ quantity }} item(s) to cart!
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  stockCount: {
    type: Number,
    required: true,
    default: 0
  }
})

const quantity = ref(1)
const showToast = ref(false)
let toastTimeout = null

const increment = () => {
  if (quantity.value < props.stockCount) {
    quantity.value++
  }
}

const decrement = () => {
  if (quantity.value > 1) {
    quantity.value--
  }
}

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
  gap: var(--spacing-4, 16px);
  margin-top: var(--spacing-6, 24px);
  width: 100%;
}

.quantity-selector {
  display: flex;
  align-items: center;
  border: 1px solid var(--border-color, #e6e6e6);
  border-radius: var(--radius-md, 8px);
  overflow: hidden;
  height: 48px;
  background: var(--color-white, #fff);
}

.qty-btn {
  background: transparent;
  color: var(--color-dark, #222);
  width: 40px;
  height: 100%;
  font-size: var(--font-size-lg, 18px);
  font-weight: var(--font-weight-regular, 400);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color var(--transition-fast, 0.15s ease);
}

.qty-btn:hover:not(:disabled) {
  background: var(--color-bg-light, #f4f4f4);
}

.qty-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.qty-display {
  width: 40px;
  text-align: center;
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-weight: var(--font-weight-medium, 500);
  color: var(--color-dark, #222);
  user-select: none;
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
