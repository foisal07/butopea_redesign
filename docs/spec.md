# Project Specification — Butopea Product Page Redesign

> **This document is the single source of truth for this project.**
>

---

## 1. Project Overview

**What we are building:**
A functional redesigned product detail page (PDP) mockup for a Hungarian furniture e-commerce brand, Butopêa. The redesign targets improved UX and conversion rate based on analysis of the original page.

**Reference product:**
- URL: https://butopea.com/3-szemelyes-kanape-kinyithato-szovet-szurke-fem-labakkal-fiacre-dehal-199649-d
- Product: FIACRE 3-seater sofa, foldable, grey fabric, metal legs
- Brand: FIACRE / Butopêa
- Price: 108,990 HUF
- Status: In stock

**Tech stack:**
- Framework: **Nuxt 3** with **Vue 3** (`<script setup>` Composition API)
- Styling: **CSS** (scoped component styles, no Tailwind)
- Language: English for code and comments; product copy may remain in Hungarian
- No TypeScript required unless the user specifies otherwise

---

## 2. What We Are NOT Building

Agents must not generate or scaffold any of the following:

| Out of scope | Reason |
|---|---|
| Header / Navbar | Not part of this redesign |
| Footer | Not part of this redesign |
| Any other common/shared layout components | Not part of this redesign |
| Real backend / API calls | Mocked only |
| Cart management | Mocked only (success toast/message on click) |
| User authentication or accounts | Out of scope |
| Search, category pages, filters | Out of scope |

If a wireframe section is not explicitly mentioned in the task for that component, **do not build it**.

---

## 3. Build Strategy

### 3.1 Platform Priority
- **Mobile-first.** All components are designed and coded for mobile (≤ 390px viewport width) first.
- Desktop responsive expansion comes later, after all mobile components are approved.
- Default breakpoints: `mobile < 768px`, `tablet 768px–1024px`, `desktop > 1024px`

### 3.2 Component-by-Component Workflow
- **One component at a time.** The agent codes only the component the user has approved.
- **Do not start the next component** until the user explicitly says "go ahead" or similar.
- After completing each component, the agent must report:
  1. Assumptions made during implementation
  2. Known edge cases
  3. What could go wrong or break

### 3.3 Implementation Plan First
- Before writing any code for a component, produce an **implementation plan**.
- The plan must describe: structure, props, state, interactions, mock behavior, and styling approach.
- **Wait for user approval** before writing code.

---

## 4. Interaction & Mock Behavior Rules

All user-facing actions must feel real but use only local mock data and state.

| Action | Expected Mock Behavior |
|---|---|
| Add to Cart | Shows a success toast/notification. No real cart state persists. |
| Image zoom / pinch-to-zoom | Works visually (CSS transform or lightbox). No external library unless approved. |
| Image gallery swipe | Swipeable carousel with dot indicators (touch events on mobile). |
| Variant / color selection | Updates displayed image and selection state locally. |
| Quantity selector | Increments/decrements a local reactive number, min 1. |
| Accordion / FAQ expand | Toggles open/close with animation. |
| Sticky CTA bar | Appears when the main "Add to Cart" button scrolls out of view. |

**No external API calls.** All product data comes from a local mock JSON file described in Section 6.

---

## 5. File & Folder Structure

The project lives at:
```
butopea-product-redesign/
├── spec.md                          ← this file
├── nuxt.config.ts
├── package.json
├── app.vue
├── pages/
│   └── product.vue                  ← entry page that assembles all components
├── components/
│   └── product/
│       ├── ImageGallery.vue
│       ├── ProductInfo.vue
│       ├── VariantSelector.vue
│       ├── AddToCart.vue
│       ├── ProductDetails.vue
│       ├── ReviewsSection.vue
│       └── [additional components as approved]
├── data/
│   └── product.mock.json            ← single source of mock data
└── assets/
    └── images/
        └── [product images]
```

Agents must follow this structure. Do not create files outside it unless the user approves.

---

## 6. Mock Product Data (`product.mock.json`)

The mock JSON must cover the following fields (populated with real values from the reference product):

```json
{
  "id": "dehal-199649-d",
  "sku": "DEHAL-199649-D",
  "name": "FIACRE",
  "subtitle": "3 személyes kanapé, kinyitható, szövet, szürke, fém lábakkal",
  "brand": "FIACRE",
  "price": 108990,
  "currency": "HUF",
  "originalPrice": null,
  "discount": null,
  "rating": 4.7,
  "reviewCount": 124,
  "inStock": true,
  "stockCount": 8,
  "images": [
    "https://butopea.com/img/1024/1024/resize/catalog/Product_pictures/DEHAL/dehal-199649[d]_1.jpg",
    "https://butopea.com/img/1024/1024/resize/catalog/Product_pictures/DEHAL/dehal-199649[d]_2.jpg",
    "https://butopea.com/img/1024/1024/resize/catalog/Product_pictures/DEHAL/dehal-199649[d]_3.jpg"
  ],
  "variants": {
    "colors": [
      { "id": "grey", "label": "Szürke", "hex": "#9E9E9E", "active": true },
      { "id": "beige", "label": "Bézs", "hex": "#D2B48C", "active": false }
    ]
  },
  "attributes": [
    { "label": "Anyag", "value": "Szövet" },
    { "label": "Lábak", "value": "Fém" },
    { "label": "Férőhelyek", "value": "3 személyes" },
    { "label": "Kinyitható", "value": "Igen" },
    { "label": "Szélesség", "value": "220 cm" },
    { "label": "Mélység", "value": "90 cm" },
    { "label": "Magasság", "value": "88 cm" }
  ],
  "description": "Egy szép és stílusos kanapé, amely szobánk gyöngyszeme lesz! Kialakíthatod kedvenc kuckózós sarkodat hálószobádban, egy hangulatos olvasós bázist a nappalidban, vagy épp a legkényelmesebb helyet a TV nézéshez!",
  "deliveryInfo": "3–5 munkanap",
  "returnPolicy": "30 napos visszaküldési jog",
  "reviews": [
    {
      "author": "Kovács Anna",
      "rating": 5,
      "date": "2024-11-10",
      "text": "Nagyon szép és kényelmes! Pontosan olyan, mint a képeken."
    },
    {
      "author": "Szabó Péter",
      "rating": 4,
      "date": "2024-10-22",
      "text": "Jó minőség, gyors szállítás. Ajánlom mindenkinek!"
    }
  ]
}
```

Agents should import this file into components using `useFetch` or direct `import` (since it's local). Do not hardcode product data inline inside component templates.

---

## 7. Design Reference

### 7.1 Original Brand Colors (extracted from live site)
| Token | Value | Usage |
|---|---|---|
| Primary (pink) | `#d95fa7` | CTAs, highlights |
| Primary bright | `#ff40a7` | Hover states |
| Secondary (teal) | `#2ebd96` | Trust badges, success |
| Dark | `#222222` | Body text |
| Grey | `#5c5c5c` | Secondary text |
| Light grey | `#e6e6e6` | Borders, dividers |
| Gold | `#eebd01` | Star ratings |

### 7.2 Typography (from live site)
- **Body / UI**: Roboto, sans-serif
- **Headings / Display**: Comfortaa, cursive
- Use Google Fonts CDN or self-host via Nuxt assets

### 7.3 Wireframe
- The wireframe is the visual authority for layout and component ordering.
- If a detail is ambiguous in the wireframe, the agent must note it as an assumption.
- Do not add sections or elements not visible in the wireframe.

---

## 8. Code Quality Rules

- Use **Vue 3 `<script setup>`** syntax in all components.
- Use **scoped CSS** (`<style scoped>`) for all component styles.
- All interactive elements must have **unique, descriptive `id` attributes** (for browser testing).
- Do not use any third-party UI component library (e.g. Vuetify, Quasar, Element Plus) unless explicitly approved.
- External libraries (e.g. Swiper.js for carousel) require user approval before use.
- Emit events from child components using `defineEmits`; do not mutate props directly.
- Use `defineProps` with validation for all component props.
- Components must be self-contained and reusable where possible.

---

## 9. Agent Behavior Rules (Critical)

These rules govern how any AI agent must operate on this project:

2. **Do not add anything not in the wireframe or this spec.** If unsure, ask.
3. **One component at a time.** Never start a new component without explicit user go-ahead.
4. **Plan before code.** Always present an implementation plan and wait for approval.
5. **Surface assumptions.** Before every component delivery, list your assumptions, edge cases, and risks.
6. **Conflict resolution.** If the user says something that contradicts this spec, ask for clarification — do not silently pick a side.
7. **No creative additions.** Do not add copy, badges, icons, animations, or sections that are not specified.
8. **Keep the mock data file as the source of truth.** Never hardcode product values inside component templates.
9. **Browser-testable.** Every delivered component must be startable with `npm run dev` and viewable in a browser without errors.
10. **Comment your assumptions** inline in the code with `// ASSUMPTION:` tags so the user can find and review them easily.

---

## 10. Delivery Checklist (per component)

After building each component, confirm all of the following before handing off to the user:

- [ ] Component file created at the correct path
- [ ] Props defined with `defineProps` and validated
- [ ] Events emitted with `defineEmits` where needed
- [ ] Mock data imported from `product.mock.json` (not hardcoded)
- [ ] Scoped CSS applied
- [ ] All interactive elements have unique `id` attributes
- [ ] Mobile layout correct at 390px viewport
- [ ] No console errors when loaded in browser
- [ ] Assumptions, edge cases, and risks documented in handoff notes

---

*Last updated: 2026-06-02*
