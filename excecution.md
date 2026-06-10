
# Shopify Grow Plan Execution README

## 1. Context
You already have:
- Shopify store created
- Shopify **Grow** plan active

You need:
- A practical, granular execution sequence to replace this app's custom commerce flow with Shopify.

This document assumes you are keeping the current Next.js website and moving commerce to Shopify headless + Shopify checkout.

## 2. What Grow Plan Means for This Migration
Based on Shopify Help/Docs (checked June 10, 2026):
- Grow includes stronger reporting and lower fees than Basic.
- Grow supports multiple staff accounts (Help Center currently states 5).
- You can run a headless storefront with Storefront API and hosted checkout flow (`Cart.checkoutUrl`).

Important practical note:
- If any feature is Shopify Plus-only, treat it as out of scope and use a workaround.  
  For this migration, you do **not** need Plus-only checkout replacement because you are redirecting to hosted checkout.

## 3. Recommended Approach (for your current repo)
- Keep your storefront pages and UI.
- Replace product/cart data source with Shopify Storefront API.
- Replace custom payment flow with redirect to Shopify `checkoutUrl`.
- Move post-purchase triggers to Shopify webhooks.

Do **not** attempt big-bang rewrite in one release. Use feature flags and phased cutover.

## 4. 4-Week Execution Plan

## Week 1: Foundation and Product Mapping
### Goal
Prepare Shopify data model + app integration foundation.

### Tasks
- [ ] Create/confirm custom app in Shopify admin.
- [ ] Enable Storefront API token and Admin API token.
- [ ] Finalize category mapping:
  - `comprehensive_lab_panel`
  - `genetic_testing_panel`
  - `supplements`
  - `supplement_stacks`
- [ ] Decide stack/bundle strategy:
  - Shopify Bundles app OR
  - Product + metafields + custom grouping
- [ ] Migrate all products, images, SKUs, inventory to Shopify.
- [ ] Create product metafields for any custom legacy fields needed by UI.
- [ ] Add feature flags in app:
  - `NEXT_PUBLIC_USE_SHOPIFY_COMMERCE`
  - `NEXT_PUBLIC_USE_SHOPIFY_CHECKOUT`
  - `SHOPIFY_WEBHOOKS_ENABLED`

### Repo Actions
- [ ] Add `src/lib/shopify/client.ts`
- [ ] Add `src/lib/shopify/queries.ts`
- [ ] Add `src/types/shopify.ts`

### Exit Criteria
- [ ] Products visible in Shopify and queryable via Storefront API.
- [ ] Feature flags merged with default `false`.

---

## Week 2: Product Pages and Cart Migration
### Goal
Make storefront product cards + cart run from Shopify (without cutover yet).

### Tasks
- [ ] Build Storefront API query for each required product section.
- [ ] Replace current product fetches on:
  - `src/components/Services/LabsPricingContent.tsx`
  - `src/components/Services/SupplementsTempBundles.tsx`
  - `src/components/Services/SupplementsTempIndividualProducts.tsx`
- [ ] Update cart model in `src/contexts/CartContext.tsx`:
  - persist Shopify `cartId`
  - store line IDs for update/remove
  - use `cartCreate`, `cartLinesAdd`, `cartLinesUpdate`, `cartLinesRemove`
- [ ] Keep existing cart slider UX; only data source changes.

### Test Cases
- [ ] Add/remove/update quantity across all categories.
- [ ] Cart persists across refresh and session.
- [ ] Cart totals from Shopify match UI.

### Exit Criteria
- [ ] End-to-end add-to-cart works against Shopify behind feature flag.

---

## Week 3: Checkout Switch and Webhooks
### Goal
Switch payment/checkout to Shopify and implement webhook bridge.

### Tasks
- [ ] Replace custom payment submit in `src/app/checkout/page.tsx`:
  - redirect to `cart.checkoutUrl`
- [ ] Disable/guard old payment UI path:
  - `src/components/Checkout/PaymentForm.tsx`
- [ ] Build webhook endpoints:
  - `src/app/api/webhooks/shopify/orders-paid/route.ts`
  - `src/app/api/webhooks/shopify/orders-fulfilled/route.ts`
  - `src/app/api/webhooks/shopify/orders-updated/route.ts`
- [ ] Implement webhook HMAC verification using raw request body and `X-Shopify-Hmac-Sha256`.
- [ ] Add webhook idempotency using `X-Shopify-Webhook-Id`.
- [ ] Map webhook payload to existing integration contracts (GHL/Zapier/Shipedge).

### Test Cases
- [ ] Place test order through Shopify checkout.
- [ ] Verify webhook delivery and signature pass.
- [ ] Ensure duplicate webhook delivery is ignored.

### Exit Criteria
- [ ] Paid order in Shopify triggers your downstream automation successfully.

---

## Week 4: Cutover, Reconciliation, and Cleanup
### Goal
Production cutover with low risk and rollback ready.

### Tasks
- [ ] Enable:
  - `NEXT_PUBLIC_USE_SHOPIFY_COMMERCE=true`
  - `NEXT_PUBLIC_USE_SHOPIFY_CHECKOUT=true`
  - `SHOPIFY_WEBHOOKS_ENABLED=true`
- [ ] Monitor 48-72 hours:
  - checkout success rate
  - webhook failure rate
  - order reconciliation count
- [ ] Validate analytics:
  - begin_checkout
  - purchase events still fired correctly
- [ ] Freeze/deprecate old routes:
  - `src/app/api/checkout/create-order/route.ts`
  - `src/app/api/checkout/process-payment/route.ts`
  - `src/app/api/checkout/confirm-order/route.ts`

### Exit Criteria
- [ ] Stable traffic on Shopify checkout.
- [ ] Zero critical mismatch in order/integration data.

## 5. Day-by-Day First 10 Days (Granular)

### Day 1
- [ ] Create `.env` entries for Shopify tokens and flags.
- [ ] Add Shopify client and run one test query locally.

### Day 2
- [ ] Create `products by category` query and type definitions.
- [ ] Render one section from Shopify data.

### Day 3
- [ ] Render all sections from Shopify data.
- [ ] Keep old source behind fallback flag.

### Day 4
- [ ] Implement `cartCreate`.
- [ ] Save `cartId` in localStorage.

### Day 5
- [ ] Implement `cartLinesAdd` and line rendering.
- [ ] Validate qty updates and line removal.

### Day 6
- [ ] Integrate cart context with slider and cart icon count.
- [ ] QA on mobile and desktop.

### Day 7
- [ ] Switch checkout CTA to use `checkoutUrl` under flag.
- [ ] Smoke-test Shopify checkout.

### Day 8
- [ ] Add webhook receiver + raw body HMAC verification.
- [ ] Add webhook event dedupe storage.

### Day 9
- [ ] Connect webhook handler to GHL/Zapier/Shipedge adapters.
- [ ] Run end-to-end paid-order simulation.

### Day 10
- [ ] Reconciliation report and launch readiness review.
- [ ] Prepare cutover checklist and rollback commands.

## 6. Technical Checklist (Repo-Specific)

### Replace/Refactor first
- [ ] `src/contexts/CartContext.tsx`
- [ ] `src/app/checkout/page.tsx`
- [ ] `src/components/Checkout/PaymentForm.tsx` (decommission path)
- [ ] Product listing components under `src/components/Services/*`

### Keep (initially)
- [ ] Existing cart slider UI (`src/components/Cart/CartSlider.tsx`)
- [ ] Existing analytics modules (`src/lib/gtmEcommerce.ts`, Convertmax helpers)

### Decommission later
- [ ] `supabase/functions/create-order`
- [ ] `supabase/functions/process-payment`
- [ ] `supabase/functions/confirm-order`
- [ ] Next `/api/checkout/*` routes

## 7. Integration Mapping You Must Define

For each Shopify order webhook, decide:
- [ ] Which payload fields map to GHL contact update.
- [ ] Which payload fields map to Zapier.
- [ ] Which payload fields map to Shipedge fulfillment creation.
- [ ] Whether to trigger at `orders/paid` or `orders/create` (recommend `orders/paid`).

## 8. UAT Pass/Fail Gates
- [ ] Product prices and titles match Shopify admin.
- [ ] Cart line updates are stable and idempotent.
- [ ] Checkout redirect always includes valid `checkoutUrl`.
- [ ] Order appears in Shopify on payment success.
- [ ] Webhooks verified and deduplicated.
- [ ] GHL/Zapier/Shipedge downstream success >= 99% in test window.

## 9. Rollback Plan (Operational)
If issues occur after cutover:
- [ ] Set `NEXT_PUBLIC_USE_SHOPIFY_CHECKOUT=false`.
- [ ] Set `NEXT_PUBLIC_USE_SHOPIFY_COMMERCE=false`.
- [ ] Set `SHOPIFY_WEBHOOKS_ENABLED=false`.
- [ ] Re-enable old flow temporarily.
- [ ] Reconcile impacted orders manually from Shopify order export.

## 10. Suggested Owner Split
- Frontend lead: Storefront + cart + checkout redirect
- Backend lead: Shopify webhooks + integration bridge
- Ops lead: product migration + shipping/tax + monitor/reconciliation

## 11. References (Official)
- Grow plan features: https://help.shopify.com/en/manual/intro-to-shopify/pricing-plans/plans-features/grow-plan
- Plan naming update context: https://help.shopify.com/en/manual/intro-to-shopify/pricing-plans/plans-features/shopify-plus
- Storefront Cart object (`checkoutUrl`): https://shopify.dev/docs/api/storefront/latest/objects/Cart
- Cart mutation (`cartLinesAdd`): https://shopify.dev/docs/api/storefront/latest/mutations/cartlinesadd
- Verify webhook deliveries: https://shopify.dev/docs/apps/build/webhooks/verify-deliveries
- Manage webhook subscriptions: https://shopify.dev/docs/apps/build/webhooks/subscribe

