# UniHome — Phase 7: Marketplace Expansion (Second-Hand Goods)

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Future Scope (Not Yet Scoped)  
> **Target Duration:** TBD (Estimated 4 weeks)  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Overview

Phase 7 expands the UniHome platform from accommodation to a broader student ecosystem marketplace. It enables students moving into or out of dorms and apartments to buy and sell second-hand goods (furniture, textbooks, appliances, electronics) within their campus community.

---

## 2. Anticipated Scope

- **Listing Categories**: Furniture, textbooks, kitchenware, electronics, transport (bicycles/scooters).
- **Campus Geo-filtering**: Item discovery scoped to specific universities or campus radii.
- **Student-to-Student Communication**: Re-use Phase 4 Communication Module with an `item_id` context.
- **Verification Integration**: Priority listing badges for students verified with university emails.
- **Status Lifecycle**: `draft` → `active` → `reserved` → `sold` → `archived`.

---

## 3. Architecture Alignment

When this phase is formally scheduled, it will adhere strictly to the established UniHome architectural standards:

1. **Custom Database Tables**: `wp_unihome_items`, `wp_unihome_item_images` (not WordPress posts or custom post types).
2. **3-Layer Pattern**: `ItemController` → `ItemService` → `ItemRepository`.
3. **Cross-Cutting Reuse**:
   - `NotificationService` for buyer inquiries and purchase notifications.
   - `EventTracker` for `item_viewed`, `item_saved`, and `item_contact` events.
   - `MediaService` for image processing and storage via WordPress media library.
4. **Autonomous Module**: Lives in `includes/Modules/Marketplace/` with zero direct coupling to Room or Payment internals.

---

## 4. Phase Status & Next Steps

This phase is currently **unscoped** and pending business prioritization following the successful completion and rollout of Phases 0 through 6. Formal task breakdown, owner assignments, and completion criteria will be defined during the Phase 6 retrospective.

---

## 5. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 7)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
