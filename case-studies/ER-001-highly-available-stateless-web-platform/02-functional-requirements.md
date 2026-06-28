# ER-001: Functional Requirements

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 02 — Functional Requirements  
**Status:** Discovery  
**Date:** 2026-06-28

---

## Purpose

Functional requirements describe **what the system must do** for the business and its users. They are independent of hosting platform, architecture, or technology selection.

Each requirement includes a **justification** — why it exists — so that future architecture decisions can be traced to business need rather than habit.

---

## Actors

| Actor | Description |
|-------|-------------|
| **Anonymous visitor** | Browses catalogue without authentication |
| **Registered customer** | Logs in, views account history, saves preferences |
| **Guest checkout customer** | Completes purchase without creating an account |
| **Storefront application** | Customer-facing web application (stateless compute tier) |
| **Internal admin** | Manages catalogue visibility, promotions, or content — scope TBC |
| **Downstream services** | Payment, inventory, order fulfilment, email notification — boundary TBC |

---

## Core Customer Journeys

The following journeys must be supported. Degrading any journey during peak trading is a business risk.

1. **Discover** — land on site, search or browse categories, view product detail
2. **Select** — add/remove items from basket, apply promotional codes where valid
3. **Checkout** — enter delivery and contact details, select shipping option
4. **Pay** — submit payment and receive authorisation outcome
5. **Confirm** — receive on-screen and email order confirmation with reference

**Justification:** These journeys map directly to revenue. A migration that preserves infrastructure but breaks checkout continuity is a failed migration.

---

## Functional Requirements

### FR-1: Product catalogue browsing

The system must allow customers to browse and search the product catalogue, view product details (price, availability indicator, description, images), and navigate category hierarchies.

**Justification:** Catalogue browsing is the entry point for all commercial activity. Unavailability blocks discovery and invalidates marketing spend driving traffic to the site.

**Priority:** Must have

---

### FR-2: Session and basket continuity

The system must allow a customer to add items to a basket and retain basket contents for the duration of a reasonable shopping session, including across page reloads and navigation within the storefront.

**Justification:** Lost baskets directly reduce conversion. Seasonal sales often involve comparison shopping — session continuity is revenue-critical even if the application tier is stateless.

**Priority:** Must have

**Discovery note:** "Stateless application" implies basket/session state is externalised or tokenised. The mechanism is an architecture decision — the **behaviour** is a functional requirement.

---

### FR-3: Promotional pricing and codes

The system must apply valid promotional codes and display correct promotional pricing during checkout, consistent with rules defined in the commerce backend.

**Justification:** Seasonal sales are the primary peak traffic driver. Incorrect pricing or failed code application during campaigns causes revenue leakage, customer complaints, and support load.

**Priority:** Must have

---

### FR-4: Checkout and order submission

The system must collect delivery address, contact information, and shipping preferences, calculate order totals (including VAT display appropriate for UK customers), and submit a completed order to downstream order processing.

**Justification:** Checkout is the conversion point. Partial or failed order submission during peak periods is indistinguishable from an outage from the customer's perspective.

**Priority:** Must have

---

### FR-5: Payment initiation and outcome handling

The system must initiate payment with the authorised payment provider and present a clear outcome to the customer — success, failure, or retry guidance — without duplicate charges for a single checkout attempt.

**Justification:** Payment failure handling affects revenue, customer trust, and dispute volume. Idempotent payment behaviour is a functional expectation, not merely a technical nicety.

**Priority:** Must have

**Discovery note:** Payment provider integration boundary must be confirmed — in-scope vs delegated to existing service.

---

### FR-6: Order confirmation

The system must display order confirmation on successful checkout and trigger customer notification (email as minimum).

**Justification:** Confirmation closes the psychological and legal loop of purchase. Missing confirmation drives support contacts and chargeback uncertainty.

**Priority:** Must have

---

### FR-7: Customer authentication (registered users)

The system must allow registered customers to log in, log out, and access account-specific views (order history, saved addresses — scope TBC).

**Justification:** Returning customers represent higher lifetime value. Authentication failures during peak periods affect a loyal segment disproportionately.

**Priority:** Must have

---

### FR-8: Content and campaign landing pages

The system must serve marketing landing pages and campaign entry URLs referenced in seasonal promotions without manual deployment for each campaign where a CMS or content mechanism already exists.

**Justification:** Marketing plans campaigns around specific URLs and timelines. Inability to serve campaign pages during a sale negates campaign investment.

**Priority:** Should have — confirm with marketing

---

### FR-9: Health and readiness signalling

The system must expose a mechanism for automated health verification so that traffic is not directed to instances that cannot serve customer requests.

**Justification:** High availability requires the ability to detect and isolate failed capacity. This is stated as functional because it defines observable behaviour the platform must support — not how it is implemented.

**Priority:** Must have

---

## Integration Points (Boundaries to Confirm)

| Integration | Direction | Purpose | In scope for ER-001? |
|-------------|-----------|---------|----------------------|
| Product/inventory source | Inbound to storefront | Stock and catalogue data | TBC |
| Payment provider | Outbound | Authorise and capture payment | TBC |
| Order management / ERP | Outbound | Persist and fulfil orders | TBC |
| Email / notification service | Outbound | Order confirmation, password reset | TBC |
| Authentication / identity | Inbound/outbound | Registered customer login | TBC |
| Analytics / tag manager | Outbound | Campaign and conversion tracking | TBC |

**Justification for documenting boundaries now:** Architecture complexity is driven as much by integrations as by compute. Unconfirmed boundaries are a discovery risk.

---

## Explicitly Out of Scope (Pending Confirmation)

The following are **not confirmed** as in scope for this migration. Treating them as out of scope without validation is a risk.

- Back-office admin and warehouse systems
- Mobile native applications (unless responsive web is the sole mobile channel)
- Marketplace or third-party seller integrations
- PCI audit scope beyond standard UK business data protection practice
- International storefronts beyond UK pricing/VAT presentation

---

## Requirements Traceability

| Requirement | Business driver |
|-------------|-----------------|
| FR-1 to FR-7 | Revenue and conversion during normal and peak trading |
| FR-8 | Marketing campaign effectiveness |
| FR-9 | Stated high availability objective |

---

## Open Questions

1. Is inventory availability real-time or cached? What is acceptable staleness during peak?
2. Are guest checkout and registered checkout both required at launch?
3. Which integrations are migrated vs remain on existing endpoints?
4. Is there a separate admin interface, and is it in scope?
5. What languages or locales beyond UK English are required?

---

## Related Documents

- [01 — Business Problem](01-business-problem.md)
- [03 — Non-functional Requirements](03-non-functional-requirements.md)
- [04 — Assumptions](04-assumptions.md)
