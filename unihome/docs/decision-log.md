# UniHome — Decision Log (Architecture Decision Records)

> **Format:** ADR (Architecture Decision Record)  
> **Rule:** Append-only. Never edit existing ADRs. Supersede with a new ADR referencing the old one.  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## ADR-001: Use Custom Database Tables Instead of wp_posts/wp_postmeta

**Date:** 2026-09-30  
**Status:** Accepted  

**Decision:** All UniHome business data (rooms, verifications, profiles, matches, messages, payments) is stored in custom database tables with typed columns and foreign keys.

**Context:** UniHome requires multi-field filtering (price + location + type + status), data integrity enforcement (typed columns, FK constraints), and query performance that `wp_postmeta`'s EAV (Entity-Attribute-Value) structure cannot provide. A room search query on `wp_postmeta` requires N self-joins for N filter fields, degrading exponentially with scale.

**Alternatives Considered:**
- Custom Post Types with `wp_postmeta` — rejected due to query performance degradation at scale, lack of data type enforcement (price stored as string), and inability to create targeted indexes on business fields.

**Consequences:**
- We lose WordPress admin UI for content management (custom admin pages required for every management view).
- We gain query performance, schema clarity, data integrity, and standard MySQL indexing.
- All table schemas are documented in `docs/database.md`.

---

## ADR-002: Use WordPress as Application Foundation

**Date:** 2026-09-30  
**Status:** Accepted  

**Decision:** WordPress is used as the application framework, providing authentication, admin panel, media management, plugin lifecycle, and database abstraction.

**Context:** Building auth, admin dashboard, media upload, session management, and CSRF protection from scratch would cost 2-3 months with zero business value. WordPress provides all of this out of the box with a mature, well-tested codebase. The team has WordPress familiarity.

**Alternatives Considered:**
- Laravel — rejected due to team unfamiliarity, additional infrastructure requirements (Homestead/Valet), and loss of the WordPress admin panel ecosystem.
- Custom PHP framework — rejected due to development time cost and maintenance burden of rolling our own auth, session, and admin systems.
- NestJS / Express — rejected due to team skill set (PHP-focused) and requirement for a fundamentally different hosting infrastructure.

**Consequences:**
- We are constrained by WordPress conventions (hook system, plugin lifecycle, PHP templating).
- We gain rapid development velocity, a mature ecosystem, and battle-tested security (WordPress auth, nonce system).
- WordPress is the framework, not the product. We do not use page builders, visual editors, or third-party business logic plugins.

---

## ADR-003: Controller → Service → Repository Pattern

**Date:** 2026-09-30  
**Status:** Accepted  

**Decision:** Every business module uses a three-layer architecture: Controller (HTTP boundary) → Service (business logic) → Repository (data access).

**Context:** With 4 developers and AI coding agents working in parallel on 8+ modules, clear layer separation is critical. Without it, SQL queries leak into business logic (causing merge conflicts when schemas change), and business logic leaks into HTTP handlers (making it untestable and duplicated).

**Alternatives Considered:**
- 2-layer (Controller + Service, with SQL in Service) — rejected because database schema changes would require reading business logic to find affected queries, leading to merge conflicts and broken code across developers.
- Enterprise pattern (Dependency Injection container, DTOs, Interface per Service, CQRS) — rejected as over-engineering for WordPress context. WordPress has no service container, and adding one conflicts with its hook-based architecture.

**Consequences:**
- Each module has 3 files minimum (Controller, Service, Repository). This is accepted overhead.
- Repository methods are thin (5-10 lines wrapping `$wpdb->prepare()`). The cost is low.
- AI agents can work on a Repository without understanding business logic, and on a Service without understanding HTTP handling.
- The pattern applies to business modules. Shared services (Notification, EventTracker) use Service + Repository (no Controller).

---

## ADR-004: Communication System Before Recommendation Engine

**Date:** 2026-09-30  
**Status:** Accepted  

**Decision:** Chat (Phase 4) is built before Recommendation (Phase 5) in the development roadmap.

**Context:** The recommendation engine's scoring model depends on behavioral signals. The strongest interest signals come from user interactions: `contact_landlord` (user initiated a conversation about a room) and `message_sent` (user is actively engaged). Building the recommendation engine without these signals produces a significantly weaker model — limited to search views and filter patterns only.

**Alternatives Considered:**
- Original ordering (Recommendation as Phase 4, Chat as Phase 5) — rejected because recommendation would launch with incomplete behavioral data, then require rework and re-tuning once Chat signals become available.

**Consequences:**
- The recommendation engine starts with a richer signal set from day one.
- Phase 3 (Roommate) defines an interface contract for Chat integration rather than a dead-code implementation.
- The `contact_landlord` event is wired into the recommendation scoring model immediately in Phase 5.

---

## ADR-005: Notification as Cross-Cutting Shared Service

**Date:** 2026-09-30  
**Status:** Accepted  

**Decision:** Notification is implemented as a shared service in `includes/shared/`, not as a standalone business module in `includes/modules/`.

**Context:** Every business module needs to notify users: Verification sends approval/rejection notices, Roommate sends match notifications, Chat sends new message alerts, Payment sends confirmation emails. If Notification were a module, every other module would depend on it, creating a dependency hub that violates the acyclic dependency principle.

**Alternatives Considered:**
- Notification as a business module — rejected because it would become a universal dependency (every module → Notification), creating a bottleneck and architectural hub.
- Inline email sending per module (`wp_mail()` in each Service) — rejected because it duplicates delivery logic, ignores user notification preferences, and makes it impossible to add new channels (push, SMS) without modifying every module.

**Consequences:**
- `NotificationService` lives in `shared/` and is available to all modules without creating module-level dependencies.
- Modules call `NotificationService::notify($event_type, $user_id, $context)` instead of `wp_mail()` directly.
- The "Never send notifications directly" rule is added to the forbidden practices list.
- Adding a new notification channel (e.g., push) only requires changes in `NotificationService`, not across all modules.

---

## Template for New ADRs

```markdown
## ADR-{NNN}: {Title}

**Date:** YYYY-MM-DD  
**Status:** Accepted | Superseded by ADR-{NNN} | Deprecated  

**Decision:** {What was decided}

**Context:** {Why this decision was needed — the problem or question that prompted it}

**Alternatives Considered:**
- {Alternative 1} — rejected because {reason}
- {Alternative 2} — rejected because {reason}

**Consequences:** {What changes as a result. Both positive outcomes and accepted trade-offs.}
```
