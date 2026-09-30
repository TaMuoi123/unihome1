# UniHome — AI Coding Agent Instructions

> **Last Updated:** 2026-09-30  
> **Maintained By:** Technical Lead only  
> **Read Time:** ~3 minutes  
> **Rule:** AI agents read this file FIRST before any task.

---

## Project Identity

UniHome is a **Student Housing Marketplace** platform built on **WordPress as an application framework**.

- WordPress is **NOT** used as a website builder.
- WordPress provides: authentication, admin panel, media management, database abstraction, plugin lifecycle, cron, email.
- **All business logic** is custom-built inside the `unihome-plugin`.
- The frontend uses **Vanilla JS** — no React, Vue, or Angular.

---

## Before You Write Any Code

Execute these steps in order. Do not skip any.

```
1. Read this file (CLAUDE.md)       → Load project constraints
2. Read STATUS.md                   → Current phase, in-progress tasks, blockers
3. Read docs/architecture.md        → Layer structure, module dependency graph
4. Read target module docs          → Module you are working on
5. Read existing code in module     → Follow established patterns exactly
6. Check docs/database.md           → Confirm table schemas before writing queries
7. Check docs/decision-log.md       → Check if your approach contradicts an existing ADR
8. Run existing tests               → vendor/bin/phpunit — confirm they pass
```

---

## Architecture Rules — Hard Constraints

These rules are non-negotiable. Violating them will result in rejected code.

### Stack

| Layer | Technology | Version |
|---|---|---|
| CMS Framework | WordPress | 6.4+ |
| Language | PHP | 8.0+ |
| Database | MySQL | 8.0+ |
| Frontend | Vanilla JS + CSS | — |
| Autoloading | Composer PSR-4 | 2.x |
| Testing | PHPUnit + WP_UnitTestCase | 9.x |

### Forbidden Actions

- **Do NOT** introduce React, Vue, Angular, NestJS, Laravel, or any framework outside WordPress.
- **Do NOT** bypass the WordPress plugin architecture. All code lives inside `unihome-plugin/`.
- **Do NOT** create Custom Post Types for business data. Rooms, verifications, profiles, matches, messages, payments — all use **custom database tables**.
- **Do NOT** put business logic in JavaScript. JS handles UI. PHP handles decisions.
- **Do NOT** write SQL outside Repository classes.
- **Do NOT** create PHP files outside the defined module structure.
- **Do NOT** throw exceptions from Service classes. Return `WP_Error` on failure.
- **Do NOT** use `admin-ajax.php` for new endpoints. All endpoints use `register_rest_route`.
- **Do NOT** send emails or notifications directly. Use `NotificationService::notify()`.
- **Do NOT** create circular dependencies between modules. Check the dependency graph below.
- **Do NOT** add manual `require_once` statements for plugin classes. Use Composer autoloader.
- **Do NOT** store API keys or secrets in code. Use `wp-config.php` constants via `Config::get()`.

---

## Module Pattern

Every **business module** follows the Controller → Service → Repository pattern:

```
Controller (HTTP boundary)
    │  Receives WP_REST_Request
    │  Sanitizes input, validates via Validator
    │  Calls Service
    │  Returns WP_REST_Response
    ▼
Service (Business logic)
    │  State transitions, rules, orchestration
    │  Returns data on success, WP_Error on failure
    │  NEVER throws exceptions
    │  Calls Logger::audit() on state changes
    │  Calls NotificationService::notify() for user-facing events
    ▼
Repository (Data access)
    │  $wpdb queries only
    │  Returns data, null, or false
    │  NEVER enforces business rules
```

**Do NOT skip layers. Do NOT merge layers.**

**Cross-cutting shared services** (Notification, EventTracker, Logger, Cache, Config, Validator, Sanitizer) live in `shared/` and do not follow this 3-layer pattern. They are utilities available to all modules.

---

## Module Dependencies

```
Room            → (none)
Verification    → Room
Roommate        → (none)
Chat            → Roommate
Recommendation  → Room, Chat
Payment         → Room
Ads             → Payment
Marketplace     → (none)
```

**Rules:**
- A module may ONLY call the Service class of modules it depends on.
- No reverse calls. No cycles.
- Cross-cutting services (`shared/`) may be called by any module.

---

## Autoloading

Classes are autoloaded via Composer PSR-4.

- Namespace root: `UniHome\` maps to `includes/`
- Example: `UniHome\Modules\Rooms\RoomService` → `includes/modules/rooms/class-room-service.php`
- Do NOT add manual `require_once` for plugin classes.

---

## Code Conventions

| Convention | Rule |
|---|---|
| Class file naming | `class-{name}.php` (e.g., `class-room-service.php`) |
| Namespace | `UniHome\Modules\{ModuleName}` |
| Formatting | WordPress PHP Coding Standards (indentation, spacing, braces) |
| REST routes | `permission_callback` required on every route |
| Database queries | `$wpdb->prepare()` for all user input |
| API responses | `WP_REST_Response` always |
| State changes | `Logger::audit()` called in Service layer |
| Notifications | `NotificationService::notify()` — never `wp_mail()` directly |
| Analytics events | `EventTracker::track()` called in Controller layer (after success) |
| New Service methods | Must include unit tests |
| Type hints | Required on all parameters and return types (PHP 8.0+) |

---

## File Locations

| What | Path (relative to plugin root) |
|---|---|
| Plugin entry | `unihome.php` |
| Composer config | `composer.json` |
| Modules | `includes/modules/{module}/` |
| Database schemas | `includes/database/schema.php` |
| Migrations | `includes/database/migrations.php` |
| Foreign keys | `includes/database/foreign-keys.php` |
| Shared utilities | `includes/shared/` |
| REST routes | `api/class-rest-router.php` |
| Admin pages | `admin/` |
| Frontend JS | `assets/js/` |
| Frontend CSS | `assets/css/` |
| Templates | `templates/` |
| Tests | `tests/modules/{module}/` |
| Shared tests | `tests/shared/` |

**Full plugin path:** `wp-content/plugins/unihome-plugin/`  
**Theme path:** `wp-content/themes/unihome-theme/`

---

## When Creating New Files

1. Place the file in the correct module directory.
2. Follow naming: `class-{descriptive-name}.php`.
3. Add the `namespace` declaration matching PSR-4.
4. Register hooks in `class-unihome-loader.php`.
5. If creating a new table:
   - Add CREATE TABLE in `includes/database/schema.php`
   - Add migration in `includes/database/migrations.php`
   - Add FK constraints in `includes/database/foreign-keys.php`
   - Document in `docs/database.md`
6. Create a test file in `tests/modules/{module}/`.

---

## When Modifying Existing Code

1. Do NOT refactor code unrelated to your task.
2. Do NOT change method signatures without confirming downstream impact.
3. Do NOT rename database columns — create a migration that adds a new column.
4. Preserve all existing comments and docblocks.
5. Run `vendor/bin/phpunit` — all existing tests must still pass.
6. Update `STATUS.md` if your change completes or advances a task.

---

## Common Mistakes to Avoid

| Mistake | Correct Approach |
|---|---|
| Creating `functions.php` with loose functions | Put logic in the appropriate Service class |
| Using `get_posts()` or `WP_Query` for UniHome data | Use the module's Repository class |
| Storing room data in `wp_postmeta` | Use `wp_unihome_rooms` custom table |
| Registering REST routes in random files | Use `api/class-rest-router.php` |
| Adding `require_once` for plugin classes | Composer autoloader handles this |
| Using `$_POST` or `$_GET` directly | Use `WP_REST_Request::get_param()` |
| Forgetting audit log on state changes | Call `Logger::audit()` in the Service method |
| Sending `wp_mail()` directly | Use `NotificationService::notify()` |
| Writing tests that call external services | Mock external dependencies |
| Skipping `$wpdb->prepare()` | Always use prepared statements |

---

## Decision Log Reference

Before proposing an alternative architecture approach, check `docs/decision-log.md` for existing ADRs:

- **ADR-001:** Custom tables over CPT — do not propose `wp_posts` for business data
- **ADR-002:** WordPress as foundation — do not propose Laravel/Symfony migration
- **ADR-003:** Controller → Service → Repository — do not propose 2-layer or DI containers
- **ADR-004:** Chat before Recommendation — do not reorder phases
- **ADR-005:** Notification as shared service — do not create a Notification module

If your task requires a decision not covered by an existing ADR, document your rationale and flag it for the Technical Lead.

---

## Quick Reference: What Goes Where

| I need to... | File to create/modify |
|---|---|
| Add a REST endpoint | `api/class-rest-router.php` + `modules/{mod}/class-{mod}-controller.php` |
| Add business logic | `modules/{mod}/class-{mod}-service.php` |
| Add a database query | `modules/{mod}/class-{mod}-repository.php` |
| Add a database table | `database/schema.php` + `database/migrations.php` + `database/foreign-keys.php` |
| Add a notification | Call `NotificationService::notify()` from the Service layer |
| Track a user action | Call `EventTracker::track()` from the Controller layer |
| Add an admin page | `admin/class-admin-pages.php` + `admin/views/` |
| Add frontend JS | `assets/js/` + enqueue in theme or controller |
| Add a test | `tests/modules/{mod}/test-{class}.php` |
| Record a decision | `docs/decision-log.md` (new ADR entry) |
| Mark task complete | `STATUS.md` |
