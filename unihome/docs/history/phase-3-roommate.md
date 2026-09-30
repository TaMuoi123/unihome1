# UniHome — Phase 3: Roommate Matching

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Not Started  
> **Target Duration:** ~3 weeks  
> **Maintained By:** Module Owner (@dev-3) & Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Goal

Empower verified students to discover compatible roommates through a standardized lifestyle questionnaire and a transparent, rule-based compatibility matching algorithm.

---

## 2. Matching Engine Algorithm (v1 — Rule-Based)

Calculated between Profile $A$ and Profile $B$ (Maximum total score: 100):

| Dimension | Rule / Tolerance | Points |
|---|---|---|
| **Sleep Schedule** | Exact match on sleep/wake ranges | +20 pts |
| **Noise Tolerance** | Absolute difference $\le 1$ on scale 1–5 | +15 pts |
| **Cleanliness** | Absolute difference $\le 1$ on scale 1–5 | +15 pts |
| **Guest Policy** | Exact match on overnight guest preferences | +15 pts |
| **Smoking Habits** | Exact match on smoking tolerance/habits | +10 pts |
| **Pet Friendliness** | Exact match on pet preferences | +10 pts |
| **Budget Compatibility** | Budget ranges overlap ($[\min_A, \max_A] \cap [\min_B, \max_B] \neq \emptyset$) | +15 pts |

*Matches with score $\ge 60\%$ are surfaced as eligible candidate matches.*

---

## 3. Team Allocation & Responsibilities

| Area | Task | Owner | Key Deliverables |
|---|---|---|---|
| Database | Schema migration for `wp_unihome_roommate_profiles` & `wp_unihome_matches` | Dev 3 | Table definitions, unique pair constraint `(user_id_1, user_id_2)`, indexes |
| Backend | `ProfileService` — questionnaire CRUD & validation | Dev 3 | Profile state management, field range sanitization |
| Backend | `MatchingEngine` — deterministic scoring algorithm | Dev 1 | Scoring engine calculation, query candidate profile pooling |
| Backend | `MatchController` — REST API endpoints | Dev 1 | Match listing, accept/decline action handlers |
| Backend | Notification — `new_match_found` event integration | Dev 2 | Trigger `NotificationService::notify()` upon candidate generation |
| Frontend | Lifestyle questionnaire form | Dev 4 | Multi-step interactive slider and option form |
| Frontend | Match discovery dashboard | Dev 2 | Candidate cards, compatibility breakdown bar, accept/decline actions |
| Testing | Scoring engine unit tests | Dev 1 | Complete test suite for scoring boundary conditions and edge cases |

---

## 4. REST API Endpoints

| Method | Endpoint | Capability / Auth | Description |
|---|---|---|---|
| `GET` | `/wp-json/unihome/v1/roommates/profile` | Authenticated (`manage_roommate_profile`) | Retrieve current student's roommate lifestyle profile |
| `POST` | `/wp-json/unihome/v1/roommates/profile` | Authenticated (`manage_roommate_profile`) | Create or update roommate lifestyle questionnaire |
| `GET` | `/wp-json/unihome/v1/roommates/matches` | Authenticated (`manage_roommate_profile`) | Fetch scored candidate matches for current student |
| `POST` | `/wp-json/unihome/v1/roommates/matches/(?P<id>\d+)/accept` | Authenticated (`manage_roommate_profile`) | Accept match request |
| `POST` | `/wp-json/unihome/v1/roommates/matches/(?P<id>\d+)/decline` | Authenticated (`manage_roommate_profile`) | Decline match request |

---

## 5. Completion Criteria & Definition of Done

- [ ] Tables `wp_unihome_roommate_profiles` and `wp_unihome_matches` created with strict constraints.
- [ ] Match pair uniqueness constraint strictly prevents duplicate reversed pairs (`user_id_1 < user_id_2` convention).
- [ ] Students can submit and update their questionnaire without data corruption.
- [ ] `MatchingEngine` accurately calculates compatibility scores according to the point table.
- [ ] Students can view their list of potential matches sorted by compatibility percentage.
- [ ] Accepting or declining a match updates match status (`pending`, `accepted`, `declined`).
- [ ] `NotificationService::notify('new_match_found', ...)` triggers when a compatible match ($\ge 70\%$) is identified.
- [ ] Unit tests verify all scoring criteria edge cases (budget overlap, boundaries, noise/cleanliness tolerance).

---

## 6. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 3)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
