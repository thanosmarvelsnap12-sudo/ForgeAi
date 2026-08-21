# GameKit AI Generation Backend

## Goal
Turn each customer's saved customization profile into an original digital asset brief, then generate assets only after a successful purchase.

## Customer flow
1. Customer selects Gamer Pack ($2.99), Gamer Plus Pack ($3.99), or Ultimate Creator Pack ($4.99).
2. Customer chooses style, colors, display name, purpose, and desired assets.
3. The site shows a preview of the brief.
4. Customer checks out through Stripe Checkout.
5. Stripe webhook verifies payment server-side.
6. Backend creates a generation job from the saved customization profile.
7. AI generation service creates the requested original assets.
8. Backend stores generated files in private object storage.
9. Customer receives a time-limited download link.
10. Support can locate the order by Stripe payment/order ID.

## Important safety/product rules
- Never expose an AI provider API key in browser JavaScript.
- Never generate or distribute third-party game logos, characters, screenshots, or other copyrighted/trademarked assets as if GameKit owns them.
- Customer-provided text should be treated as untrusted input.
- The final generator should use a controlled prompt template plus the customer's choices rather than directly executing arbitrary prompts.
- Generation must be tied to a verified Stripe payment on the server.
- Downloads should use short-lived signed URLs rather than public permanent file URLs.
- Store only the customer information required to deliver and support the order.

## Controlled generation brief
Use a server-side template such as:

"Create original digital gaming/creator artwork. Style: {style}. Color direction: {colors}. Display name: {sanitized_name}. Purpose: {purpose}. Requested assets: {allowed_assets}. Notes: {sanitized_notes}. Do not use or imitate third-party characters, logos, screenshots, trademarks, or copyrighted game artwork. Do not claim affiliation with any game or publisher. Create commercially usable original artwork suitable for the purchased GameKit license."

## Pack entitlements
### Gamer Pack — $2.99
- 5 personalized wallpapers
- Personalized quest tracker
- Style/color/name customization

### Gamer Plus Pack — $3.99
- Everything in Gamer Pack
- Personalized stream screens
- 10 personalized challenge cards
- Expanded creator options

### Ultimate Creator Pack — $4.99
- Everything in Gamer Plus Pack
- YouTube creator planner
- Discord starter resources
- Extra creator assets
- Eligible future pack updates

## Backend endpoints to implement
- `POST /api/checkout-session` — creates Stripe Checkout Session from an allowed product ID.
- `POST /api/stripe-webhook` — verifies Stripe signature and records successful payment.
- `POST /api/generation-jobs` — creates a job only for a paid order.
- `GET /api/orders/:id/download` — returns a short-lived signed download URL.
- `POST /api/support` — creates a support/refund request linked to an order.

## Data model
`orders`: id, stripe_checkout_session_id, stripe_payment_intent_id, customer_email, product_id, status, created_at

`customizations`: order_id, tier, style, colors, display_name, purpose, assets, notes, created_at

`generation_jobs`: id, order_id, status, provider_job_id, error_code, created_at, completed_at

`downloads`: id, order_id, storage_key, expires_at, download_count

`support_requests`: id, order_id, email, type, message, attachment_key, status, created_at

## Current status
The public GameKit page already contains the customer customization wizard and preview. The remaining production work is the secure server-side Stripe + generation + private delivery connection described above.
