# UniHome — Phase 8: Advertisement

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Future Scope (Not Yet Scoped)  
> **Target Duration:** TBD (Estimated 3–4 weeks)  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Overview

Phase 8 provides commercial monetization diversification through targeted campus advertising zones, sponsored partner placements, impression and click tracking, and self-serve advertising packages for local businesses, tutoring centers, and campus merchants.

---

## 2. Anticipated Scope

- **Ad Zones**: Homepage banner, room search interstitial, roommate recommendation sponsor cards, sidebar banners.
- **Audience Targeting**: Geo-targeting by university campus, student vs. landlord personas, category affinity.
- **Analytics & Tracking**: High-throughput impression logging, click-through rate (CTR) calculation, fraud/bot filtering.
- **Monetization Integration**: Purchasing ad inventory through Phase 6 Payment Gateway infrastructure.
- **Self-Serve Advertiser Portal**: Merchant dashboard to upload creatives, set budgets, and monitor campaign performance.

---

## 3. Architecture Alignment

When scheduled for implementation, Phase 8 will adhere strictly to UniHome architectural standards:

1. **Custom Database Tables**: `wp_unihome_ad_campaigns`, `wp_unihome_ad_creatives`, `wp_unihome_ad_impressions`.
2. **3-Layer Pattern**: `AdController` → `AdService` → `AdRepository`.
3. **High-Performance Tracking**: Asynchronous or batched impression logging to prevent database lock contention on high-traffic pages.
4. **Autonomous Module**: Placed in `includes/Modules/Advertisement/` with zero direct coupling to Room or Roommate core modules.

---

## 4. Phase Status & Next Steps

This phase is currently **unscoped** and deferred until the platform reaches substantial active user scale across universities. Requirements, detailed database schemas, and performance benchmarks will be architected after the platform achieves significant traffic metrics.

---

## 5. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 8)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
