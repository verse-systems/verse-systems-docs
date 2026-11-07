# Verse Marketplace: Product Document

**Status:** Draft v1
**Owner:** Abimbola Jolayemi
**Scope:** Product rules only. Technical design lives in `architecture/`; decisions live in `adr/`.
**Last updated:** 2026-11-08

**Status tags used below:** `[Decided]` agreed with the project owner · `[Proposed]` recommended, awaiting confirmation · `[Open]` not yet decided.

---

## 1. Problem Statement

Traditional ecommerce platforms make it easy to buy and sell online but do not reflect local commerce, especially where trade follows time-based market cycles.

In many communities, commerce is organised around market days, when buyers and sellers gather, demand peaks, and pricing and availability shift within a specific window. Most ecommerce systems ignore this rhythm and offer static, always-on listings. Sellers miss peak-visibility opportunities, and buyers lose the timely discovery and haggling that define traditional markets.

Verse introduces a time-based commerce model that mirrors real market behaviour.

## 2. Product Vision

Verse is an ecommerce platform designed to work the way real-world commerce naturally operates. Activity is organised around structured **Market Days**: scheduled windows in which selected products receive special visibility and pricing, and buyers can negotiate. This brings urgency, structure and rhythm to digital commerce while keeping the scale and convenience of online systems.

## 3. Mission

Build a time-driven ecommerce platform that aligns buying and selling with real market cycles, so that sellers and buyers engage meaningfully during defined trading periods.

## 4. Glossary

| Term | Meaning |
|---|---|
| **Market Day** | A platform-scheduled trading window with special rules. |
| **Session** | One occurrence of a Market Day. |
| **Base price** | The seller's normal listing price. |
| **Display price** | The price shown to buyers during a Market Day for a selected product. |
| **Minimum price** | The seller's hidden lowest acceptable price for a product in one session. Never shown to anyone but the seller. |
| **Selection** | A seller choosing a product to take part in a session. |
| **Assistant** | The named, clearly labelled negotiation assistant that speaks on the seller's behalf. |
| **Price commitment** | A short-lived, single-use record that a specific buyer may buy a product at an agreed price. |

## 5. Roles

- **Buyer**, **Seller** and **Admin**. A user may hold more than one role (for example Buyer and Seller). `[Decided]`
- Only verified sellers may take part in Market Day. `[Proposed]`

## 6. Market Day Definition

- Market Days are set by the platform on a regular cadence, monthly or twice monthly, like recurring local markets. `[Decided]`
- In the MVP an Admin creates each session. Automatic generation from a recurrence rule is a later addition. `[Proposed]`
- Verse is a single marketplace, not a platform hosting several markets. There is one Market Day at a time for the whole platform, so sessions do not overlap. `[Decided]`
- All times are stored in UTC and displayed in the user's local time (WAT for Nigeria). `[Proposed]`
- A session lasts **24 hours** by default. The Admin sets the exact start and end for each session, so a longer session (for example 48 hours) remains possible. `[Decided]`

## 7. Market Day Session Lifecycle

States move forward only and are driven by timestamps on the session. `CANCELLED` is reachable only before `ACTIVE`. `[Proposed]`

| State | What happens | Who can do what |
|---|---|---|
| **SCHEDULED** | Session exists with fixed dates. | Admin may edit or cancel. Buyers see a countdown. |
| **SELECTION_OPEN** | Sellers choose products and set display and minimum prices. | Sellers add, edit and withdraw selections. Eligibility is validated. |
| **LOCKED** | Selection is frozen. Reference prices are snapshotted and the activation set is prepared. | No participation changes. Admin may cancel or remove a suspicious listing. |
| **ACTIVE** | Trading. Display prices are live and selected products get visibility. | Sellers may adjust prices within guardrails. Buyers shop, negotiate and check out. |
| **CLOSING** | Trading has ended. Short grace window, then cleanup. | No new Market Day cart lines. Only payments already started may complete. |
| **CLOSED** | Cooldown. Boosts are removed and participation is archived. | Sellers decide whether to keep or revert prices. |
| **CANCELLED** | Session called off before it started. | Affected sellers and buyers are notified. |

**Rules**
- "Market mode" for the platform is derived: market mode applies only while a session is `ACTIVE`; otherwise normal commerce applies.
- Deadlines are enforced by timestamp at the point of use (for example, checkout compares the current time to the session end). A late scheduler must never leave a market open.
- The grace window is **15 minutes** after the session ends. Only payments already started before the end may complete within it. Nothing new can start. `[Decided]`

## 8. Before, During and After Market Day

### Before (SCHEDULED to LOCKED): normal commerce
- Sellers create and publish products. Products follow normal ranking. Prices are the seller's base prices.
- Buyers browse, save products and add items to the cart.
- Sellers select products during `SELECTION_OPEN` only.

### During (ACTIVE): active trading
- Only selected products receive Market Day benefits: display pricing, visibility and negotiation.
- Unselected products stay under normal rules, without boosts.
- Buyers must complete checkout before the session ends.

### After (CLOSING and CLOSED): cooldown
- Visibility boosts and Market Day prices stop applying. Products return to normal ranking.
- Sellers choose to **revert** to the base price or **keep** the Market Day price as the new base price. If a seller does nothing, the price reverts. `[Proposed]`
- The seller has **48 hours** after close to decide. If the seller does nothing by then, the price stays reverted. `[Decided]`
- Participation resets, and sellers must reselect for the next session.

## 9. Product Participation

**Eligibility.** A product must be published, active (not archived or disabled), in stock, and explicitly selected by its seller before the session is locked. `[Decided]`

**Selection constraints**
- No new selections after `LOCKED`. `[Decided]`
- A seller must provide a display price and a minimum price when selecting. `[Decided]`
- There is no limit on the number of products a seller may select, and no eligibility rules beyond those above. `[Decided]`
- Category-based inclusion and stock thresholds are future options, not part of the MVP.

## 10. Pricing

- Buyers see a product's price according to the session state at the moment they view it. The system resolves the price at read time from the base price and the product's activation state. It does not edit every product's price at session start and end. `[Proposed]`
- The **minimum price** is set by the seller per product, per session, at selection. It must be greater than zero and not above the display price. `[Decided]`
- The minimum price is never shown to buyers and never leaves the service that owns it. `[Decided]`
- A product can have up to three prices: base price, display price and the hidden minimum price. `[Decided]`
- **Guardrails** against unreasonable price changes apply to every selected product. The numbers below are defaults, and an Admin can adjust them per session. `[Decided]`
    1. The base price of a selected product is frozen from `LOCKED` until close, and serves as the reference price (this stops inflate-then-discount).
    2. The display price may only be at or below the reference price (discount-only).
    3. The minimum price must be at least 50% of the reference price.
    4. A seller may edit a product's price at most 3 times during `ACTIVE`, and every change is recorded in an audit trail.
    5. No price changes in the last 30 minutes of the session.
- Currency is **Naira (NGN, ₦)**. This replaces the USD default in the Architecture Specification. `[Decided]`

## 11. Visibility

- A product that has been selected, with a valid display price and minimum price, becomes Market Day visible. There are no boost tiers or paid promotion in the MVP. `[Decided]`
- Selected products appear in a **dedicated Market Day section**. `[Decided]`
- Within that section, products appear in a random order that stays fixed for the whole session (fair to all sellers), and buyers can sort by price or discount. `[Proposed]`
- In the main product listing, a selected product shows its display price and a Market Day badge, with no ranking boost. `[Proposed]`

## 12. Buying and Negotiation

Buyers may **buy at the display price** or **negotiate**. Negotiation is optional. `[Decided]`

**Negotiation experience**
- It looks like a chat with the seller's assistant, but every offer is a structured action (a price entry or quick-reply chips). Free text is short, optional and never changes the outcome. `[Proposed]`
- The assistant has a name and is clearly labelled as the seller's assistant, for example "Ada, [Store name]'s assistant". It is never presented as the seller or as a human. `[Decided]`
- Tone is warm and local.

**Negotiation rules**
- Decisions are made by deterministic rules. A language model, if used, only phrases replies. `[Decided]`
- The system never accepts an offer below the seller's minimum price. `[Decided]`
- Each buyer may make **3 offers per product per Market Day**. `[Decided]`
- A rejection reveals nothing about how close the offer was. `[Proposed]`
- Counter-offers from the assistant are not part of the MVP. `[Proposed]`

**Accepted offers (price commitment)** `[Decided]`
- An accepted offer creates a price commitment: buyer, product, quantity, price and expiry. It is single use and expires at the earlier of a short time limit or the end of the session.
- The buyer adds the item to the cart at the agreed price. The order is created only at checkout, and the commitment is consumed at that moment.
- No order exists before checkout.

## 13. Cart Rules

Each cart line keeps the pricing it had when it was added: `[Decided]`

| Line type | Price | At Market Day close |
|---|---|---|
| **STANDARD** | Normal price, unaffected by Market Day | Stays in the cart |
| **MARKET_DAY** | Display price | Removed |
| **NEGOTIATED** | Agreed price (tied to a price commitment) | Removed |

- Items added before Market Day keep the normal price during and after Market Day. They never receive Market Day prices. `[Decided]`
- The price on a cart line is the price the buyer pays. There is no "switch to Market Day price" prompt, and checkout does not reprice lines, including `STANDARD` lines. `[Decided]`
- Carts that have been inactive for 30 days are cleared, so old prices do not linger indefinitely. `[Proposed]`
- Buyers must complete checkout before the session ends. `[Decided]`

## 14. Checkout and Orders

- Checkout validates on the server that the session is still open (or within the grace window for payments already started), that any price commitment is still valid, and that stock is available. Line prices are honoured as shown in the cart. `[Decided]`
- Repeated checkout submissions must not create duplicate orders. `[Proposed]`
- **Stock:** when checkout starts, the stock is reserved for 15 minutes. It is kept if payment succeeds, and released if payment fails or times out. Stock is not reserved when an item is added to the cart. `[Decided]`
- **Payments:** the platform integrates a payment provider behind a provider interface, starting with a stub. Which provider to use (for example Paystack or Flutterwave) is `[Open]`.

## 15. Notifications

Buyers and sellers are notified when a session is scheduled, opens for selection, starts and ends, when a cart is cleared at close, when an order is confirmed, and when a sale is made. Channels for the MVP: in-app, plus email if a provider is chosen. `[Proposed]`

## 16. Verse High-Level Flow

The authoritative sequence diagram is maintained as Mermaid source in `architecture/diagrams/`. It covers scheduling, selection, browsing, buying at the display price, negotiation, checkout and close.

## 17. MVP Scope

**Included**
- Registration, login and roles
- Seller setup and a minimal store profile
- Product management with base price, stock and publication status
- Admin-scheduled Market Day sessions with the full lifecycle
- Seller selection with display price and hidden minimum price
- Read-time Market Day pricing in Naira, with guardrails
- Visibility for selected products
- Negotiation through the named assistant, with the 3-offer limit and price commitments
- Cart with the three line types and close-time cleanup
- Checkout, one payment provider (or a stub first) and order history
- Notifications for session and order events
- A basic buyer, seller and admin interface

**Deferred**
- Counter-offers from the assistant
- Paid or tiered visibility boosts, category-based inclusion, stock thresholds
- Automatic session generation
- Logistics and delivery
- Reviews and ratings, refunds and disputes, analytics, push notifications
- Dedicated search engine and personalised ranking

## 18. Open Decisions

| # | Decision | Where it matters |
|---|---|---|
| 1 | Payment provider (Paystack, Flutterwave or another) | Section 14 |
| 2 | Confirm the 30-day cart inactivity rule | Section 13 |
| 3 | Confirm the proposed items: random fixed order in the Market Day section, sort options and the main-listing badge | Section 11 |
