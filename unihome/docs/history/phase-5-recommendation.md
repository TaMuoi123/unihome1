# UniHome — Phase 5: Recommendation Engine

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Not Started  
> **Target Duration:** ~4 weeks (+ 1 week Integration Checkpoint 2)  
> **Maintained By:** Module Owner (@dev-1) & Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Goal

Deliver personalized room recommendations to students by synthesizing explicit preference data (budget, district, room type) with implicit behavioral signals harvested during Phases 1–4 (`room_search`, `room_viewed`, and `contact_landlord`).

---

## 2. Recommendation Architecture

The engine is phased into two progressive tiers:

### Phase 5a — Rule-Based Multi-Signal Scorer (Current Phase Scope)
Calculates a relevance score $S \in [0, 100]$ for room $R$ against user profile $U$:

| Factor | Weight | Signal Source | Scoring Logic |
|---|---|---|---|
| **Price Match** | 20% | Profile budget vs. room price | Maximum score if room price falls comfortably within profile budget |
| **Location Match** | 20% | Profile preferred campus/district vs. room address | Distance calculation or exact district match |
| **Room Type Match** | 10% | Profile preferred room type vs. room type | Exact match on single room, shared room, studio |
| **Listing Recency** | 10% | `created_at` timestamp | Exponential decay curve based on days since posting |
| **Verification Status** | 10% | Room `is_verified` flag | Verified listings receive full 10% boost |
| **Landlord Rating** | 10% | Historical rating or response rate | Default neutral score until rating system introduced |
| **Search Similarity** | 10% | `room_search` events from `wp_unihome_events` | Matches user's recent keyword and filter searches |
| **Contact Signal** | 10% | `contact_landlord` events from Phase 4 | Boosts rooms similar to those the user reached out to |

### Phase 5b — AI Behavioral Embeddings (Future SaaS Capability)
Prototype integration with Gemini API to evaluate unstructured room descriptions against student lifestyle profiles and interaction vector embeddings.

---

## 3. Team Allocation & Responsibilities

| Area | Task | Owner | Key Deliverables |
|---|---|---|---|
| Database | Recommendation score cache schema & index optimizations | Dev 4 | Temporary scoring/transient cache table or indexed metadata |
| Backend | `RecommendationEngine` — multi-signal scoring service | Dev 1 | Algorithmic scoring loop, fallback for cold-start users |
| Backend | `EventRepository` — aggregate behavioral signals from `wp_unihome_events` | Dev 2 | High-efficiency event aggregation queries for user signal profiles |
| Backend | `RecommendationController` — REST API feed endpoint | Dev 2 | `/wp-json/unihome/v1/recommendations` endpoint with pagination & caching |
| Frontend | "Recommended for You" carousel & section | Dev 3 | Dynamic dashboard widget highlighting personalized listings with match badges |
| Research | Gemini API integration prototype (Phase 5b exploration) | Dev 4 | Technical feasibility spike, cost estimate, prompt architecture |
| Testing | Scoring engine unit & performance tests | Dev 1 | Benchmark scoring latency over 1,000+ candidate rooms |

---

## 4. REST API Endpoints

| Method | Endpoint | Capability / Auth | Description |
|---|---|---|---|
| `GET` | `/wp-json/unihome/v1/recommendations` | Authenticated (`read_rooms`) | Returns personalized room recommendations for current user |
| `GET` | `/wp-json/unihome/v1/recommendations/guest` | Public | Returns popularity- and recency-based trending rooms (cold start) |

---

## 5. Integration Checkpoint 2 (Post-Phase 5)

Following completion of Phase 5, the team executes a 1-week major architectural review:

1. **Cross-Module Verification**: Full end-to-end user journey across all 5 built modules (Room creation → Verification → Roommate search → Messaging → Personalized recommendations).
2. **Performance Stress Test**: Benchmark database query load with 1,000+ rooms, 500+ active users, and 10,000+ event records. Verify transient cache hit ratios.
3. **Capability & Security Review**: Ensure zero permission leaks across all REST namespaces.
4. **Clean-Install Testing**: Verify clean database deployment from empty database using the migration runner.
5. **Phase 6+ Roadmap Alignment**: Retrospective meeting to review metrics and schedule Phase 6 (Monetization).

---

## 6. Completion Criteria & Definition of Done

- [ ] `RecommendationEngine` computes scores factoring in preferences AND behavioral events (`contact_landlord`, `room_search`).
- [ ] Cold-start students (with no search/contact history) gracefully fall back to profile preferences and trending listings.
- [ ] Recommendation endpoint response latency remains $< 250$ms under load through transient caching.
- [ ] Students see a distinct "Recommended for You" module on their dashboard.
- [ ] Unit tests validate weight distribution and scoring logic correctness.
- [ ] Integration Checkpoint 2 successfully executed and signed off by Technical Lead.

---

## 7. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 5)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
