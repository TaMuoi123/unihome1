# UniHome — Phase 0: Foundation

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Not Started  
> **Target Duration:** ~3 weeks (Phase 0a: 1 week TL-only; Phase 0b: 2 weeks full team)  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Goal

Establish the WordPress application foundation, engineering standards, core scaffolding, database migration system, shared cross-cutting services, and developer tooling before any business module is built.

---

## 2. Team Allocation & Sub-Phases

### Sub-Phase 0a — Technical Lead Bootstrap (Week 1)
Solo execution by Technical Lead to guarantee foundational integrity without team coordination overhead.

| Task | Owner | Deliverable / Target File |
|---|---|---|
| WordPress installation & hardening | TL | `wp-config.php`, security salts, debug config |
| Setup `composer.json` with PSR-4 | TL | `composer.json` (`UniHome\` namespace autoloading) |
| Plugin skeleton & module structure | TL | `wp-content/plugins/unihome-plugin/` directory tree |
| Database migration runner & FK script | TL | `includes/Database/Migrator.php`, schema runner |
| PHPUnit & `WP_UnitTestCase` bootstrap | TL | `phpunit.xml`, `tests/bootstrap.php` |
| AI agent instructions & core docs | TL | `CLAUDE.md`, `STATUS.md`, `README.md`, `docs/` |
| Git repository & branch protection | TL | Main branch rules, PR templates, commit guidelines |

### Sub-Phase 0b — Team Infrastructure Scaffolding (Weeks 2–3)
Full team execution across shared services and runtime infrastructure.

| Task | Owner | Deliverable / Target File |
|---|---|---|
| Custom user roles & capabilities | Dev 1 | `includes/Auth/Roles.php` (`student`, `landlord`, `verifier`) |
| Base theme & template layout | Dev 2 | `wp-content/themes/unihome-theme/` structure |
| REST API router & health check | Dev 3 | `includes/Rest/Router.php`, `/wp-json/unihome/v1/health` |
| Shared utilities & cross-cutting scaffolds | Dev 3 | `Logger.php`, `Validator.php`, `NotificationService.php` (scaffold), `EventTracker.php` (scaffold) |
| Module isolation & feature flags | Dev 4 | `includes/Core/ModuleManager.php`, feature toggle config |
| Operational documentation completion | Dev 4 | `docs/environment-setup.md`, `docs/security.md`, `docs/decision-log.md` |

---

## 3. Detailed Task Breakdown

### 3.1 Plugin & Autoloading Architecture
- Plugin entry point: `unihome-plugin.php` with activation/deactivation hooks.
- Composer PSR-4 autoloading root: `UniHome\` mapped to `includes/`.
- Strict type declarations (`declare(strict_types=1);`) in all PHP class files.
- Separation of modules into autonomous folders: `includes/Modules/{ModuleName}/`.

### 3.2 Database Migration System
- Migration version tracking table: `wp_unihome_migrations`.
- Schema runner utilizing `dbDelta()` for table creation.
- Secondary execution pass for foreign key constraints (`ADD CONSTRAINT ... FOREIGN KEY ...`).
- Rollback and version tracking logic to prevent partial migration states.

### 3.3 Shared Cross-Cutting Services (Scaffolding)
- `UniHome\Shared\NotificationService`: Public interface with `notify(string $event, array $payload, array $channels): bool`. Stubbed in Phase 0 to return `true` and log via Logger.
- `UniHome\Shared\EventTracker`: Public interface with `track(string $eventType, array $properties, ?int $userId): bool`. Writes to log or stub in Phase 0.
- `UniHome\Shared\Logger`: Standardized logging helper writing structured errors to `wp-content/debug.log`.
- `UniHome\Shared\CacheManager`: Wrapper around WordPress transients with defined TTL constants and key prefixes.

### 3.4 REST API Foundation
- API namespace: `unihome/v1`.
- Router registering base endpoints and permission callback patterns.
- Health check endpoint: `GET /wp-json/unihome/v1/health` returning status, database connectivity, and timestamp.
- Consistent JSON response envelopes: `{ success: true, data: { ... } }` and standard error formats.

### 3.5 Role & Capability Matrix
- Register custom roles on plugin activation:
  - `student` (Read-only browsing, message landlords, manage roommate profile).
  - `landlord` (Create/manage rooms, upload images, receive inquiries).
  - `verifier` (Review submitted rooms, approve/reject verifications).
- Capability assignment and cleanup on plugin deactivation.

---

## 4. Completion Criteria & Definition of Done

To exit Phase 0 and begin Phase 1, all of the following criteria must pass:

- [ ] Plugin activates cleanly in WordPress without PHP warnings or notices.
- [ ] Composer PSR-4 autoloader correctly resolves classes in `UniHome\` namespace.
- [ ] Migration system executes successfully, creates `wp_unihome_migrations`, and completes a test migration.
- [ ] `GET /wp-json/unihome/v1/health` returns HTTP 200 with `{ "success": true, "status": "ok" }`.
- [ ] Roles `student`, `landlord`, and `verifier` exist in WordPress database with correct capabilities.
- [ ] PHPUnit test suite executes with at least one passing test case using `WP_UnitTestCase`.
- [ ] Feature flag mechanism can toggle a dummy module on/off without affecting other subsystems.
- [ ] NotificationService and EventTracker interfaces are registered and callable without runtime errors.
- [ ] All 25 architecture, operational, and phase specification documentation files are verified.

---

## 5. Phase Outcomes & Retrospective

*(This section will be populated by the Technical Lead upon completion of Phase 0)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
