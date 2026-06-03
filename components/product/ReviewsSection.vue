<script setup>
import { ref, computed } from 'vue'
import mockData from '~/data/product.mock.json'

// ── Data ─────────────────────────────────────────────────────────────────────
const rating      = mockData.rating      ?? 0
const reviewCount = mockData.reviewCount ?? 0
const reviews     = Array.isArray(mockData.reviews) ? mockData.reviews : []

// ── Drawer state ─────────────────────────────────────────────────────────────
const drawerOpen = ref(false)

function openDrawer()  {
  drawerOpen.value = true
  document.body.style.overflow = 'hidden'
}
function closeDrawer() {
  drawerOpen.value = false
  document.body.style.overflow = ''
}
// Close on Escape key
if (typeof window !== 'undefined') {
  window.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && drawerOpen.value) closeDrawer()
  })
}

// ── Star helpers ─────────────────────────────────────────────────────────────
function starSegments(value) {
  const clamped = Math.min(Math.max(Number(value) || 0, 0), 5)
  return Array.from({ length: 5 }, (_, i) => {
    const diff = clamped - i
    if (diff >= 0.75) return 'full'
    if (diff >= 0.25) return 'half'
    return 'empty'
  })
}

// ── Date formatter ───────────────────────────────────────────────────────────
function formatDate(dateStr) {
  if (!dateStr) return ''
  const d = new Date(dateStr)
  if (isNaN(d)) return dateStr
  return d.toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' })
}

// ── Computed ─────────────────────────────────────────────────────────────────
const summaryStars   = computed(() => starSegments(rating))
const firstReview    = computed(() => reviews[0] ?? null)
const firstStars     = computed(() => firstReview.value ? starSegments(firstReview.value.rating ?? 0) : [])
const drawerReviews  = computed(() =>
  reviews.map((r, i) => ({ ...r, stars: starSegments(r.rating ?? 0), _key: i }))
)
const hasReviews     = computed(() => reviews.length > 0)
</script>

<template>
  <section class="reviews-section" aria-labelledby="reviews-heading">
    <h2 id="reviews-heading" class="reviews-heading">Customer reviews</h2>

    <!-- ── Summary Card ──────────────────────────────────────────────────── -->
    <div class="card summary-card" role="region" aria-label="Rating summary">
      <div class="summary-left">
        <span class="summary-rating-number">{{ rating }}</span>
        <div class="stars-row" role="img" :aria-label="`${rating} out of 5 stars`">
          <svg
            v-for="(seg, idx) in summaryStars" :key="idx"
            class="star-icon" viewBox="0 0 24 24" aria-hidden="true"
          >
            <defs v-if="seg === 'half'">
              <clipPath :id="`hs-${idx}`"><rect x="0" y="0" width="12" height="24"/></clipPath>
            </defs>
            <path class="star-bg" d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
            <path v-if="seg !== 'empty'" class="star-fill"
              d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"
              :clip-path="seg === 'half' ? `url(#hs-${idx})` : undefined"
            />
          </svg>
        </div>
        <span class="summary-label">Average rating</span>
      </div>

      <div class="summary-divider" aria-hidden="true"></div>

      <div class="summary-right">
        <span class="summary-count-number">{{ reviewCount }}</span>
        <span class="summary-label">Reviews</span>
      </div>
    </div>

    <!-- ── Single preview review ─────────────────────────────────────────── -->
    <template v-if="hasReviews && firstReview">
      <div class="card review-card">
        <!-- Top row: pill badge + date -->
        <div class="review-top-row">
          <div class="rating-badge" role="img" :aria-label="`${firstReview.rating} out of 5 stars`">
            <svg
              v-for="(seg, sidx) in firstStars" :key="sidx"
              class="star-icon star-icon--sm" viewBox="0 0 24 24" aria-hidden="true"
            >
              <defs v-if="seg === 'half'">
                <clipPath :id="`hf-${sidx}`"><rect x="0" y="0" width="12" height="24"/></clipPath>
              </defs>
              <path class="star-bg" d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
              <path v-if="seg !== 'empty'" class="star-fill"
                d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"
                :clip-path="seg === 'half' ? `url(#hf-${sidx})` : undefined"
              />
            </svg>
          </div>
          <span v-if="firstReview.date" class="review-date">{{ formatDate(firstReview.date) }}</span>
        </div>

        <!-- Title -->
        <p v-if="firstReview.title" class="review-title">{{ firstReview.title }}</p>

        <!-- Body -->
        <p class="review-text">{{ firstReview.text }}</p>

        <!-- Author row -->
        <div class="review-author-row">
          <!-- verified checkmark -->
          <svg class="verified-icon" viewBox="0 0 24 24" aria-hidden="true">
            <circle cx="12" cy="12" r="10" fill="var(--color-secondary, #2ebd96)"/>
            <polyline points="7 13 10 16 17 9" stroke="#fff" stroke-width="2.2" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <span class="review-author">{{ firstReview.author || 'Anonymous' }}</span>
        </div>
      </div>

      <!-- ── Show all reviews button ──────────────────────────────────── -->
      <button
        id="btn-show-all-reviews"
        class="btn-show-all"
        type="button"
        aria-haspopup="dialog"
        @click="openDrawer"
      >
        Show all reviews
      </button>
    </template>

    <!-- ── Empty state ────────────────────────────────────────────────────── -->
    <div v-else class="no-reviews" role="status">
      <svg class="no-reviews-icon" viewBox="0 0 24 24" aria-hidden="true">
        <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
      </svg>
      <p class="no-reviews-text">No reviews yet.</p>
      <p class="no-reviews-sub">Be the first to share your experience!</p>
    </div>
  </section>

  <!-- ── Reviews Drawer (Teleport to body) ───────────────────────────────── -->
  <Teleport to="body">
    <Transition name="backdrop-fade">
      <div
        v-if="drawerOpen"
        class="drawer-backdrop"
        aria-hidden="true"
        @click="closeDrawer"
      ></div>
    </Transition>

    <Transition name="drawer-slide">
      <div
        v-if="drawerOpen"
        class="drawer"
        role="dialog"
        aria-modal="true"
        aria-labelledby="drawer-title"
      >
        <!-- Drag handle bar -->
        <div class="drawer-handle" aria-hidden="true"></div>

        <!-- Header -->
        <div class="drawer-header">
          <h3 id="drawer-title" class="drawer-title">All Reviews</h3>
          <button
            class="drawer-close-btn"
            type="button"
            aria-label="Close reviews drawer"
            @click="closeDrawer"
          >
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <line x1="18" y1="6" x2="6" y2="18" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"/>
              <line x1="6" y1="6" x2="18" y2="18" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"/>
            </svg>
          </button>
        </div>

        <!-- Scrollable review list -->
        <ul class="drawer-review-list">
          <li
            v-for="review in drawerReviews"
            :key="review._key"
            class="drawer-review-item"
          >
            <!-- Top row: pill badge + date -->
            <div class="review-top-row">
              <div class="rating-badge" role="img" :aria-label="`${review.rating} out of 5 stars`">
                <svg
                  v-for="(seg, sidx) in review.stars" :key="sidx"
                  class="star-icon star-icon--sm" viewBox="0 0 24 24" aria-hidden="true"
                >
                  <defs v-if="seg === 'half'">
                    <clipPath :id="`hd-${review._key}-${sidx}`"><rect x="0" y="0" width="12" height="24"/></clipPath>
                  </defs>
                  <path class="star-bg" d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
                  <path v-if="seg !== 'empty'" class="star-fill"
                    d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"
                    :clip-path="seg === 'half' ? `url(#hd-${review._key}-${sidx})` : undefined"
                  />
                </svg>
                <svg class="chevron-icon" viewBox="0 0 24 24" aria-hidden="true">
                  <polyline points="6 9 12 15 18 9" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
              <span v-if="review.date" class="review-date">{{ formatDate(review.date) }}</span>
            </div>

            <!-- Title -->
            <p v-if="review.title" class="review-title">{{ review.title }}</p>

            <!-- Body -->
            <p class="review-text">{{ review.text }}</p>

            <!-- Author row -->
            <div class="review-author-row">
              <svg class="verified-icon" viewBox="0 0 24 24" aria-hidden="true">
                <circle cx="12" cy="12" r="10" fill="var(--color-secondary, #2ebd96)"/>
                <polyline points="7 13 10 16 17 9" stroke="#fff" stroke-width="2.2" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              <span class="review-author">{{ review.author || 'Anonymous' }}</span>
            </div>
          </li>
        </ul>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
/* ── Section wrapper ──────────────────────────────────────────────────────── */
.reviews-section {
  margin-top: var(--spacing-6, 24px);
  background-color: var(--color-bg-light, #f5f5f5);
  padding: var(--spacing-6) var(--spacing-4);
  border-radius: var(--radius-md, 8px);
  display: flex;
  flex-direction: column;
  gap: var(--spacing-4);
}

.reviews-heading {
  font-family: var(--font-body);
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-bold);
  color: var(--color-dark);
  margin: 0;
}

/* ── Shared card shell ────────────────────────────────────────────────────── */
.card {
  background-color: var(--color-white, #ffffff);
  border: 1px solid var(--color-light-grey, #e6e6e6);
  border-radius: var(--radius-md, 8px);
  padding: var(--spacing-5);
}

/* ── Summary card ─────────────────────────────────────────────────────────── */
.summary-card {
  display: flex;
  align-items: center;
  gap: var(--spacing-6);
}

.summary-left {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-1);
}

.summary-divider {
  width: 1px;
  align-self: stretch;
  background-color: var(--color-light-grey, #e6e6e6);
  flex-shrink: 0;
}

.summary-right {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-1);
}

.summary-rating-number,
.summary-count-number {
  font-family: var(--font-body);
  font-size: var(--font-size-2xl, 32px);
  font-weight: var(--font-weight-bold);
  color: var(--color-dark);
  line-height: 1;
}

.summary-label {
  font-family: var(--font-body);
  font-size: var(--font-size-sm, 14px);
  color: var(--color-grey, #5c5c5c);
  margin-top: var(--spacing-1);
}

/* ── Stars ────────────────────────────────────────────────────────────────── */
.stars-row {
  display: flex;
  align-items: center;
  gap: 2px;
}

.star-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.star-icon--sm {
  width: 17px;
  height: 17px;
}

.star-bg   { fill: var(--color-light-grey, #e6e6e6); }
.star-fill { fill: var(--color-secondary, #2ebd96); }

/* ── Rating badge (pill with stars + chevron) ─────────────────────────────── */
.rating-badge {
  display: inline-flex;
  align-items: center;
  gap: 3px;
}

/* ── Preview review card ──────────────────────────────────────────────────── */
.review-card {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-3);
}

.review-top-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--spacing-3);
  flex-wrap: wrap;
}

.review-date {
  font-family: var(--font-body);
  font-size: var(--font-size-sm, 14px);
  color: var(--color-grey, #5c5c5c);
  white-space: nowrap;
}

.review-title {
  font-family: var(--font-body);
  font-size: var(--font-size-base, 16px);
  font-weight: var(--font-weight-bold);
  color: var(--color-dark);
  margin: 0;
  line-height: var(--line-height-tight, 1.2);
}

.review-text {
  font-family: var(--font-body);
  font-size: var(--font-size-base, 16px);
  color: var(--color-dark);
  line-height: var(--line-height-normal, 1.5);
  margin: 0;
}

.review-author-row {
  display: flex;
  align-items: center;
  gap: var(--spacing-2);
  margin-top: var(--spacing-1);
}

.verified-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

.review-author {
  font-family: var(--font-body);
  font-size: var(--font-size-sm, 14px);
  color: var(--color-grey, #5c5c5c);
}

/* ── "Show all reviews" button ────────────────────────────────────────────── */
.btn-show-all {
  display: block;
  width: 100%;
  padding: var(--spacing-4) var(--spacing-6);
  background-color: transparent;
  border: 2px solid var(--color-light-grey, #e6e6e6);
  border-radius: var(--radius-full, 9999px);
  font-family: var(--font-body);
  font-size: var(--font-size-base, 16px);
  font-weight: var(--font-weight-bold);
  color: var(--color-dark, #222222);
  text-align: center;
  cursor: pointer;
}

.btn-show-all:focus-visible {
  outline: 2px solid var(--color-primary, #d95fa7);
  outline-offset: 3px;
}

/* ── Empty state ──────────────────────────────────────────────────────────── */
.no-reviews {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-2);
  padding: var(--spacing-10) var(--spacing-6);
  text-align: center;
}

.no-reviews-icon {
  width: 48px;
  height: 48px;
  fill: var(--color-light-grey, #e6e6e6);
}

.no-reviews-text {
  font-family: var(--font-body);
  font-size: var(--font-size-base, 16px);
  font-weight: var(--font-weight-bold);
  color: var(--color-dark);
  margin: 0;
}

.no-reviews-sub {
  font-family: var(--font-body);
  font-size: var(--font-size-sm, 14px);
  color: var(--color-grey, #5c5c5c);
  margin: 0;
}

/* ── Responsive ───────────────────────────────────────────────────────────── */
@media (min-width: 480px) {
  .reviews-section {
    padding: var(--spacing-8) var(--spacing-6);
  }
}

/* ================================================================
   DRAWER STYLES  (not scoped to section, rendered via Teleport)
   These need :global() because they live outside the component DOM
   ================================================================ */

/* Backdrop */
:global(.drawer-backdrop) {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  z-index: 200;
  cursor: pointer;
}

/* Drawer panel */
:global(.drawer) {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 201;
  background: #ffffff;
  border-radius: 20px 20px 0 0;
  max-height: 85dvh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 -4px 32px rgba(0, 0, 0, 0.15);
  /* Make width responsive and take more space on desktop */
  width: 100%;
  max-width: min(900px, 92vw);
  margin-inline: auto;
}

/* Drag handle */
:global(.drawer-handle) {
  width: 44px;
  height: 4px;
  border-radius: 2px;
  background: #d1d1d1;
  margin: 14px auto 0;
  flex-shrink: 0;
}

/* Drawer header */
:global(.drawer-header) {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px 12px;
  border-bottom: 1px solid #e6e6e6;
  flex-shrink: 0;
}

:global(.drawer-title) {
  font-family: var(--font-body, 'Roboto', sans-serif);
  font-size: 18px;
  font-weight: 700;
  color: #222222;
  margin: 0;
}

:global(.drawer-close-btn) {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border: none;
  border-radius: 50%;
  background: #f4f4f4;
  color: #222222;
  cursor: pointer;
  transition: background 0.15s ease;
  flex-shrink: 0;
}

:global(.drawer-close-btn:hover) {
  background: #e6e6e6;
}

:global(.drawer-close-btn svg) {
  width: 18px;
  height: 18px;
}

/* Scrollable list */
:global(.drawer-review-list) {
  list-style: none;
  padding: 0;
  margin: 0;
  overflow-y: auto;
  flex: 1;
  -webkit-overflow-scrolling: touch;
}

/* Individual review item */
:global(.drawer-review-item) {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 20px;
  border-bottom: 1px solid #e6e6e6;
}

:global(.drawer-review-item:last-child) {
  border-bottom: none;
  padding-bottom: 28px;
}

/* ── Backdrop transition ─────────────────────────────────────────────────── */
:global(.backdrop-fade-enter-active),
:global(.backdrop-fade-leave-active) {
  transition: opacity 0.28s ease;
}

:global(.backdrop-fade-enter-from),
:global(.backdrop-fade-leave-to) {
  opacity: 0;
}

/* ── Drawer slide transition ─────────────────────────────────────────────── */
:global(.drawer-slide-enter-active) {
  transition: transform 0.35s cubic-bezier(0.32, 0.72, 0, 1);
}

:global(.drawer-slide-leave-active) {
  transition: transform 0.28s cubic-bezier(0.4, 0, 1, 1);
}

:global(.drawer-slide-enter-from),
:global(.drawer-slide-leave-to) {
  transform: translateY(100%);
}
</style>
