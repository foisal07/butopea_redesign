# Butopea Product Page Redesign — Submission Notes

## Time Breakdown
UX audit, research & redesign (wireframing): 1h 30m 
AI assisted development: 5h 30m
Documentation: 30m 
**Total** : **~7h 30m**

---

## 1. UX Reasoning — Why I Made These Changes

Every change was driven by a single question: **what stops a user from clicking Add to Cart?**

I identified friction points through auditing the live page, a quick round of user observation, and competitor benchmarking (IKEA, JYSK, TEMU). Each redesign decision directly removes one of those friction points.

---

### 1.1 Sticky Add to Cart — Mobile Bottom Bar & Desktop Sticky Column

**Problem:** Once the user scrolls past the gallery, the Add to Cart button is gone. They have to scroll all the way back up to buy. In my quick user observation, users were literally scrolling up and down to get back to the button.

**Change:** A sticky bottom bar on mobile that appears when the main button scrolls out of view. On desktop, the right column (price, CTA, delivery) stays fixed as the user reads reviews and specs on the left.

**Why it improves conversion:** The moment a user decides to buy, they need zero friction. Any extra action — even a scroll — is a drop-off risk. Removing that barrier directly reduces abandonment at the most critical step.

> A sticky bar that only says "Add to Cart" isn't enough. Users who haven't decided yet need the two things that close decisions: **Price + Delivery**. So the sticky bar includes both.

---

### 1.2 Delivery Date Format & FREE Shipping

**Problem:** The current format is dense — it shows two dates with years that the user must mentally parse. It also never says the word "delivery", so a scanning user can miss it entirely. The free shipping information sits in a separate block with no visual connection to the delivery info. Currently user needs to qualify themselves if it can be FREE shipment

**Change:** Reformatted to `Delivery: 16 Jun – 30 Jun` with a truck icon. Free shipping text moved directly under the Add to Cart button as a single bolded line: *"FREE shipping for this item."*

**Why it improves conversion:** Delivery date and shipping cost are the two biggest sources of purchase anxiety after price. Making them scannable and immediately visible reduces hesitation. Putting "FREE shipping for this item" directly under the CTA button is a micro-nudge — they don't need to qualify and it rewards the decision the user is about to make. 

---
### 1.3 "Why Choose Butopea?" — Price Transparency

**Problem:** The "Why are prices so good?" section competes visually with reviews despite being less important to a first-time buyer. Price skepticism is a real barrier for a brand the user may not know yet and the price point and direct to customer is an USP for Butopea that should get user attention.

**Change:** Compact section showing USPs (25% less than other brands, free returns within 30 days). "How?" opens a modal explaining the direct-from-factory logistics model — keeping the user on the page.

**Why it improves conversion:** Price skepticism kills conversions silently. Proactively addressing it — and doing it in a modal rather than a new page — neutralises the doubt without breaking the purchase flow.

---

### 1.4 Size in the Product Title and Product gallery

**Problem:** Product size is buried inside the product specifications accordion. Users have to actively scroll and hunt for it — but for furniture, size is one of the top reasons people don't buy.

**Change:** Appended size to the title above the fold: *"FIACRE — 3 személyes".*

**Why it improves conversion:** A user who can't easily find size information will leave to measure or think about it — and likely not come back. IKEA and JYSK both surface size in the title for this reason. Removing that question above the fold keeps the user in the decision-making flow.

---

### 1.5 Price & Title Hierarchy

**Problem:** The title is slightly more prominent than the price. But users who land on a product page already know what the product is — they came from a listing, an ad, or a search result. What they need next is the **price**.

**Change:** Price made larger and bolder. Title kept smaller above it.

**Why it improves conversion:** Give users the information they need in the order they need it. Price → size → delivery → add to cart. Matching visual hierarchy to the user's mental model reduces friction.

---

### 1.6 Gallery — Swipe & Mobile Zoom

**Problem:** The gallery is click-to-slide on mobile, which is unintuitive. Pinch-to-zoom doesn't work — it zooms the entire browser page instead of the product image. From user research, people zoom to check fabric and material texture, which is a critical trust signal for furniture.

**Change:** Native CSS scroll-snapping for mobile swipe. Dedicated lightbox with pinch-to-zoom, pan, and button zoom controls. Fullscreen lightbox via Vue Teleport so it sits above all layout layers.

**Why it improves conversion:** Seeing material quality up close is a purchase prerequisite for furniture. If the user can't verify it, they don't buy.

---


### 1.7 Reviews — Remove Auto-Sliding

**Problem:** Reviews auto-slide, which is a bad pattern for reading. Users trying to read a review get interrupted, creating cognitive overload and a sense that the content is being controlled rather than explored.

**Change:** Static reviews. One visible by default, "Show all" reveals the full list with a slide-up animation.

**Why it improves conversion:** Reviews are trust signals. Trust signals only work if users can actually read them. Letting users read at their own pace makes the review section do its actual job.

---

### 1.8 Product Details — Accordion

**Problem:** All specs displayed as a flat block. Users don't need all the information — showing everything creates cognitive burden and makes it harder to find the specific thing they're looking for.

**Change:** Three-section accordion (Features & Dimensions / Wrapping / Product Details). Smooth CSS grid animation.

**Why it improves conversion:** Users self-select the information they need. Less noise → less cognitive load → more focus on the decision.

---

### 1.9 Button Size — 39px → 44px+

**Problem:** The current Add to Cart button is 39px tall. Apple's HIG and Google's Material Design both specify a **minimum 44px tap target** for touch interfaces.

**Change:** Increased button height to meet the minimum standard.

**Why it improves conversion:** A sub-standard tap target causes misses, frustration, and distrust — all at the exact moment the user is trying to convert.

---

## 2. How I Worked With AI

## Implementation Strategy & Setup: Spec-Driven, Component-by-Component

Before writing any component code, I created a `spec.md` — a source-of-truth document with goals, constraints, folder structure, mock JSON schema, and brand tokens. I gave the AI a rough draft and asked it to improve it for agent consumption while keeping my original intent intact.

From there I followed a consistent prompt pattern:

> *Read spec.md and acknowledge constraints → Implementation plan → Wait for approval → Code*
> 

Every component prompt followed the same structure:

Read spec.md and acknowledge the constraints. Then:

Goal:        One sentence.
Rules:       3–7 bullets (constraints, no libraries, design tokens to use).
Examples:    2 input → output examples.
Edge cases:  2 cases that could break the component.

This kept the AI focused and made it easier to spot when it hallucinated or overreached.

---

## 3. Accept / Reject Log

| Component | AI Output | Decision | Reason |
|---|---|---|---|
| `spec.md` | Added "read this file before every task" rule | **Rejected** | Unnecessary token cost per prompt; I reference specific rules on demand instead |
| `ImageGallery` | Used generic Tailwind-style colour variables | **Rejected** | Violated brand tokens; prompted to replace with `var(--color-*)` from spec |
| `ImageGallery` | No scroll event debouncing | **Fixed** | Rapid `@scroll` events caused redundant state updates; added debouncing |
| `ImageGallery` | Named status badge `<StatusLabel />` | **Fixed** | Wrong convention; correct is `<UtilsStatusLabel text="Available" />` |
| Component scaffold | Generated `VariantSelector` not in wireframe | **Rejected** | Out of scope at that stage; removed to stay focused |
| `ProductActions` | Added Pinia store for cart state | **Rejected** | Spec explicitly excluded real cart management; stripped to local state + `setTimeout` |
| `StickyCartBar` | Proposed JS scroll detection for bottom padding | **Rejected** | Simple static CSS padding achieves the same result with zero JS overhead |
| `pages/product.vue` | Two-column layout applied at all breakpoints | **Fixed** | Should be desktop-only; prompted for CSS flat grid using `grid-column`/`grid-row` |
| `ReviewsSection` | All reviews visible on initial load | **Revised** | Wireframe calls for one visible review + slide-up panel on "Show all" click |

---
## 4. Detailed Reports
UX audit of Butopea product page: 
https://app.notion.com/p/UX-Audit-and-Redesign-Decisions-Why-37324ec6117f802f8ce9f8e040a1c6b1?source=copy_link

The Full AI assisted developement workflow: 
https://app.notion.com/p/Documentation-AI-assisted-workflow-for-Butopea-product-page-redesign-37224ec6117f8078b2efc6f04bd0859a?source=copy_link


## 5. What I Read & Referenced

- Live audit of the [Butopea product page](https://butopea.com/3-szemelyes-kanape-kinyithato-szovet-szurke-fem-labakkal-fiacre-dehal-199649-d)
- Quick user observation (2–3 users, unmoderated)
- Competitor pages: IKEA, JYSK, TEMU — for title format, accordion pattern, shipping display
- Apple HIG & Material Design — tap target minimums (44px)
- Vue 3 docs — `IntersectionObserver`, `Teleport`, `scroll-snap`
- Spec-driven AI development references:
  - [Why Write Specs Before Coding](https://youtu.be/RhaF4LVAVng)
  - [AI-Assisted Component Workflow](https://youtu.be/mViFYTwWvcM)

## 6. What I Would Do With More Time

- **A/B test the sticky bar** — validate whether showing price + delivery in the bar increases conversion vs. a CTA-only version
- **AR furniture preview** — let users visualise the piece in their own space before buying (highest-impact trust signal for furniture)
- **Real user testing** — validate delivery date format and size-in-title assumptions with actual Butopea customers
- **Price comparison widget** — surface a live "25% cheaper than [competitor]" data point next to the price to close price-skeptical users
