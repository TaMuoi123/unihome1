# UniHome — Phase 2: Room Verification

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Not Started  
> **Target Duration:** ~3 weeks (+ 1 week Integration Checkpoint 1)  
> **Maintained By:** Module Owner (@dev-2) & Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Goal

Establish the platform trust and safety layer. Allow landlords to submit rooms for physical/document verification, provide verifiers and administrators with a dedicated review workflow, attach verified badges to listings, and notify landlords of approval or rejection.

---

## 2. Verification State Machine Workflow

```
[Draft / Unverified]
         │
         ▼  (Landlord submits documentation)
[Pending Review]
         │
         ▼  (Verifier claims submission)
   [In Review]
     │       │
     │       ▼ (Defects found, feedback provided)
     │   [Rejected] ───► Landlord edits & resubmits ───► [Pending Review]
     │
     ▼ (Checks passed)
 [Approved] ───► Room marked is_verified=1 (Verified Badge active)
```

---

## 3. Team Allocation & Responsibilities

| Area | Task | Owner | Key Deliverables |
|---|---|---|---|
| Database | Schema migration for `wp_unihome_verifications` | Dev 2 | Migration class, table creation, foreign keys to rooms & users |
| Backend | `VerificationService` — state machine logic & audit log | Dev 2 | Transition validations, rejection reason handling, audit logging |
| Backend | `VerificationController` — REST API endpoints | Dev 1 | Submit, claim, approve, and reject REST handlers |
| Backend | Notification integration for verification events | Dev 3 | Wire `NotificationService::notify()` for `room_approved` & `room_rejected` |
| Frontend | Landlord verification request UI | Dev 3 | Document upload, lease agreement submission, status tracker |
| Admin | Verifier review dashboard & queue | Dev 4 | Pending list, verification checklist, approve/reject modal |
| Testing | Unit tests for Verification state machine & auth checks | Dev 1 | Edge case test coverage for transitions, unauthorized actions |

---

## 4. REST API Endpoints

| Method | Endpoint | Capability / Auth | Description |
|---|---|---|---|
| `POST` | `/wp-json/unihome/v1/verifications` | Authenticated (Landlord / Room Owner) | Submit room for verification with proof documents |
| `GET` | `/wp-json/unihome/v1/verifications/pending` | Authenticated (`verify_rooms` / Verifier / Admin) | Fetch pending verification queue |
| `GET` | `/wp-json/unihome/v1/verifications/(?P<id>\d+)` | Authenticated (Room Owner or Verifier) | Get verification inspection details and history |
| `POST` | `/wp-json/unihome/v1/verifications/(?P<id>\d+)/claim` | Authenticated (`verify_rooms`) | Verifier assigns submission to themselves (`in_review`) |
| `POST` | `/wp-json/unihome/v1/verifications/(?P<id>\d+)/approve` | Authenticated (`verify_rooms`) | Approve verification; updates room badge and notifies landlord |
| `POST` | `/wp-json/unihome/v1/verifications/(?P<id>\d+)/reject` | Authenticated (`verify_rooms`) | Reject with detailed comments; notifies landlord to resubmit |

---

## 5. Integration Checkpoint 1 (Post-Phase 2)

Following completion of Phase 2, a 1-week dedicated Integration Checkpoint occurs before starting Phase 3:

1. **End-to-End Workflow Testing**: Test complete lifecycle from room creation (Phase 1) through verification submission, verifier claim, approval/rejection, and badge display.
2. **Notification Delivery Validation**: Confirm actual email/in-app delivery of `room_approved` and `room_rejected` messages via `NotificationService`.
3. **Database & Index Profiling**: Audit query execution plans on `wp_unihome_rooms` and `wp_unihome_verifications` under simulated load.
4. **Security & Capability Audit**: Validate that normal students or unauthorized landlords cannot access verification endpoints.
5. **Documentation Catchup**: Review and synchronize any API or schema adjustments in `docs/api.md` and `docs/database.md`.

---

## 6. Completion Criteria & Definition of Done

- [ ] `wp_unihome_verifications` table deployed with valid foreign keys and indexes.
- [ ] Verification state machine prevents illegal state hops (e.g., cannot approve directly from draft).
- [ ] Approved rooms immediately show the `is_verified` badge in room listings and detail pages.
- [ ] Rejection requires a descriptive reason string and logs the decision in the audit trail.
- [ ] `NotificationService::notify('room_approved', ...)` and `'room_rejected'` send notifications to the room owner.
- [ ] Verifier queue displays pending items sorted by submission recency.
- [ ] Unit tests for `VerificationService` achieve 100% state machine branch coverage.
- [ ] Integration Checkpoint 1 checklist fully signed off by Technical Lead.

---

## 7. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 2)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
