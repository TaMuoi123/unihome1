# UniHome — Phase 6: Monetization

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Not Started  
> **Target Duration:** ~4 weeks  
> **Maintained By:** Module Owner (@dev-2) & Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Goal

Establish the platform revenue infrastructure: listing packages, listing promotion boosts, payment gateway integration (Stripe / local payment provider), subscription tracking, automated invoice generation, and transaction auditing.

---

## 2. Strategic Payment Architecture

Monetization is intentionally deferred to Phase 6.  
**Strategic Rationale:** Maximizing marketplace liquidity and achieving product-market fit precedes monetization. Introducing paywalls too early suppresses user adoption and reduces listing volume.

### Full Monetization Scope (Phase 6 Core)
- Payment Gateway Adapter architecture (`StripeAdapter`, `VnPayAdapter` / `MoMoAdapter`).
- Asynchronous webhook processing with replay protection and signature validation.
- Automated package allocation and listing expiration tracking via `wp_cron`.
- Complete transaction ledger and revenue reporting in WordPress admin.

### Safety Valve: MVP-Lite Monetization Option (Phase 4.5 Candidate)
If stakeholders require earlier revenue validation before Phase 6, a lightweight flow can be deployed within 1 week without delaying the core roadmap:

| MVP-Lite Component | Implementation | Effort |
|---|---|---|
| Package Definition | Seed `wp_unihome_packages` table with 3 basic tiers | 2 days |
| Listing Boost Flag | Add `is_boosted` column to `wp_unihome_rooms` + search priority | 2 days |
| Manual Bank Transfer Flow | Show bank account & QR code; admin manually toggles confirmation | 1 day |
| Gateway Integration | Deferred completely; zero gateway API complexity | 0 days |

*Note: The default roadmap executes full Phase 6. MVP-lite is invoked only upon explicit business stakeholder mandate.*

---

## 3. Team Allocation & Responsibilities

| Area | Task | Owner | Key Deliverables |
|---|---|---|---|
| Database | Schema migration for `wp_unihome_packages` & `wp_unihome_payments` | Dev 2 | Package definitions, payment ledger tables, index on transaction IDs |
| Backend | `PaymentGatewayInterface` & Gateway Adapter implementation | Dev 2 | Stripe / payment provider client, charge creation, webhook verification |
| Backend | `PaymentService` — package purchase flow & transaction state machine | Dev 1 | Order creation, webhook processing (`pending` → `completed` / `failed`), audit log |
| Backend | Notification & Events integration | Dev 3 | `NotificationService::notify('payment_completed')`, `EventTracker::track('package_purchased')` |
| Frontend | Pricing table & checkout modal | Dev 3 | Package selection cards, feature comparison, payment form embed |
| Admin | Payment transaction ledger & manual verification UI | Dev 4 | Financial audit log, payment status overrides, invoice downloads |
| Testing | Webhook security & transaction concurrency tests | Dev 1 | Idempotency tests, fake webhook signature tests, state transition verification |

---

## 4. REST API Endpoints

| Method | Endpoint | Capability / Auth | Description |
|---|---|---|---|
| `GET` | `/wp-json/unihome/v1/packages` | Public | List available listing packages and boost options |
| `POST` | `/wp-json/unihome/v1/payments/checkout` | Authenticated (`manage_payments` / Landlord) | Create payment intent / checkout session |
| `POST` | `/wp-json/unihome/v1/payments/webhook` | Public (Signature-verified) | Webhook listener from payment provider |
| `GET` | `/wp-json/unihome/v1/payments/history` | Authenticated (Landlord / Admin) | View user's past purchase transactions and invoices |

---

## 5. Webhook Security & Idempotency Rules

1. **Signature Verification**: Every incoming webhook must have its cryptographic signature validated against the provider secret before any processing occurs.
2. **Idempotency**: Webhook events must check `transaction_id` in `wp_unihome_payments`. If already marked `completed`, return HTTP 200 immediately without re-crediting packages.
3. **Database Transactions**: Status transition and package credit allocation must occur inside an atomic database transaction.

---

## 6. Completion Criteria & Definition of Done

- [ ] Tables `wp_unihome_packages` and `wp_unihome_payments` deployed with index on `transaction_id`.
- [ ] Landlord can purchase listing packages and pay via the integrated payment gateway.
- [ ] Webhook receiver validates signatures and handles duplicate deliveries idempotently.
- [ ] Boosted listings receive elevated placement in room search results.
- [ ] Expiration cron job runs daily to deactivate expired boost packages.
- [ ] `NotificationService::notify('payment_completed', ...)` emails receipt to customer.
- [ ] Complete financial audit trail logged in `wp_unihome_audit_logs`.
- [ ] End-to-end sandbox payment test executes successfully in CI/staging.

---

## 7. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 6)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
