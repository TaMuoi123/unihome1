# UniHome — Documentation Architecture Proposal

> **Document Type:** Architecture Proposal — Pending Team Approval  
> **Version:** 0.3.0  
> **Date:** 2026-09-30  
> **Author:** Technical Lead  
> **Status:** 🟡 FINAL DRAFT — Ready for Team Approval  
> **Audience:** Development Team (1 TL + 4 Devs), AI Coding Agents  
> **Revision Note:** Phase reorder, Notification Service, Analytics module, Decision Log, Payment strategy, architecture pattern review.

---

## Table of Contents

1. [Documentation Philosophy](#1-documentation-philosophy)
2. [Proposed Repository Documentation Structure](#2-proposed-repository-documentation-structure)
3. [UniHome Technical Architecture](#3-unihome-technical-architecture)
4. [WordPress Development Approach](#4-wordpress-development-approach)
5. [UniHome Plugin Architecture](#5-unihome-plugin-architecture)
6. [Database Architecture](#6-database-architecture)
7. [Development Rules](#7-development-rules)
8. [Development Phase Proposal](#8-development-phase-proposal)
9. [Example STATUS.md Preview](#9-example-statusmd-preview)
10. [Example CLAUDE.md Preview](#10-example-claudemd-preview)
11. [Decision Log](#11-decision-log)
12. [Team Approval Checklist](#12-team-approval-checklist)

---

## 1. Documentation Philosophy

### Why Documentation-Driven Development

UniHome is not a throwaway prototype. It is a multi-module marketplace platform with a 5-person team and AI coding agents operating concurrently. Without a single source of truth, the following failure modes are inevitable:

- **Architectural drift** — developers independently introduce patterns that contradict each other.
- **Knowledge silos** — only the original author understands a module's internal state.
- **AI context loss** — coding agents start fresh every session and hallucinate structures that don't exist.
- **Onboarding friction** — new developers (or replacement developers) cannot self-serve.

Documentation is not an afterthought. It is **infrastructure**. Every module, every phase transition, and every architectural decision must be recorded before work begins.

### Developer Workflow

```
1. Read STATUS.md              → Understand what phase the project is in
2. Read module documentation   → Understand the module you are assigned to
3. Check database.md           → Confirm table schemas before writing queries
4. Read architecture.md        → Confirm the pattern you must follow
5. Implement                   → Write code within the defined boundaries
6. Update STATUS.md            → Mark task as completed or in-progress
```

The developer **never** starts coding without reading `STATUS.md` first. This file is the project's heartbeat.

### AI Coding Agent Workflow

```
1. Read CLAUDE.md              → Load project identity, constraints, and rules
2. Read STATUS.md              → Understand current phase and progress
3. Read architecture.md        → Understand layer boundaries
4. Read target module docs     → Understand the specific module to work on
5. Read existing code          → Follow established patterns, do not reinvent
6. Implement                   → Generate code within documented constraints
7. Run tests                   → Verify generated code passes existing tests
8. Report changes              → Output summary of what was created/modified
```

AI agents are **stateless between sessions**. `CLAUDE.md` exists to reconstruct project context in under 60 seconds. Without it, every AI session starts from zero and risks producing incompatible code.

### Core Principle

> If it is not documented, it does not exist.  
> If it contradicts the documentation, the code is wrong.

---

## 2. Proposed Repository Documentation Structure

```
unihome/
│
├── CLAUDE.md                          # AI agent instruction file
├── STATUS.md                          # Current project state (single source of truth)
├── README.md                          # Project overview and setup instructions
├── CONTRIBUTING.md                    # PR template, commit format, review expectations
├── composer.json                      # Autoloader and dependency management
│
├── docs/
│   ├── architecture.md                # System architecture and layer definitions
│   ├── wordpress-development-guide.md # WordPress conventions and boundaries
│   ├── database.md                    # All table schemas, migrations, relationships
│   ├── modules.md                     # Module ownership, responsibilities, interfaces
│   ├── api.md                         # REST API endpoint catalog
│   ├── environment-setup.md           # Local dev environment: Docker/WAMP, WP config, debug
│   ├── testing.md                     # Test runner, conventions, how to run, CI
│   ├── security.md                    # Nonce strategy, capability matrix, OWASP checklist
│   ├── error-handling.md              # Error conventions, logging format, monitoring
│   ├── caching-strategy.md            # Transient keys, TTLs, invalidation rules
│   ├── deployment.md                  # Deploy process, staging vs production, DB migration
│   ├── roles-and-capabilities.md      # Full capability matrix per role per module
│   ├── decision-log.md                # Architecture Decision Records (ADRs)
│   │
│   └── history/
│       ├── phase-0-foundation.md
│       ├── phase-1-room-marketplace.md
│       ├── phase-2-verification.md
│       ├── phase-3-roommate.md
│       ├── phase-4-communication.md
│       ├── phase-5-recommendation.md
│       └── phase-6-monetization.md
│
├── wp-content/
│   ├── themes/unihome-theme/          # Custom theme (presentation layer)
│   └── plugins/unihome-plugin/        # Core application plugin (business logic)
│
└── ...
```

### File Purposes

| File | Purpose | Updated By |
|------|---------|------------|
| `CLAUDE.md` | AI agent bootstrap context. Contains project identity, architecture rules, and hard constraints. Read-only for agents. | Technical Lead |
| `STATUS.md` | Current phase, completed tasks, in-progress work, next milestone. Updated after every meaningful change. | Any developer completing a task |
| `README.md` | Setup instructions, environment requirements, local development guide. | Technical Lead |
| `CONTRIBUTING.md` | PR template, commit message format (`type(module): description`), branch naming rules, code review expectations, and definition of done. | Technical Lead |
| `docs/architecture.md` | Layer diagram, component responsibilities, data flow, integration points, module dependency graph. | Technical Lead |
| `docs/wordpress-development-guide.md` | How the team uses WordPress APIs. What is allowed, what is forbidden. Hook conventions, nonce patterns, capability checks. | Technical Lead |
| `docs/database.md` | Complete schema reference. Every custom table, every field, every index, every migration version. Soft-delete policies. | Technical Lead + assigned developer |
| `docs/modules.md` | Module boundaries, ownership assignments, public interfaces between modules, dependency direction rules. | Technical Lead |
| `docs/api.md` | REST endpoint catalog. Route, method, parameters, auth requirements, response format. | Module owner |
| `docs/environment-setup.md` | Local development environment setup: Docker or WAMP/MAMP, WordPress configuration, plugin symlink, `WP_DEBUG` settings, Xdebug configuration. | Technical Lead |
| `docs/testing.md` | PHPUnit setup, `WP_UnitTestCase` configuration, test file naming, how to run tests, minimum coverage expectations, CI integration. | Technical Lead |
| `docs/security.md` | Nonce strategy per endpoint type, capability matrix per role, input sanitization reference, OWASP top-10 checklist mapped to UniHome. | Technical Lead |
| `docs/error-handling.md` | `WP_Error` return conventions, logging format (`[UniHome:{module}]`), monitoring approach, cron failure surfacing. | Technical Lead |
| `docs/caching-strategy.md` | Transient key naming (`unihome_{module}_{entity}_{id}`), TTLs per cache target, invalidation rules per write operation. | Technical Lead |
| `docs/deployment.md` | Deployment checklist, staging vs production environment differences, database migration execution on deploy, rollback procedure. | Technical Lead |
| `docs/roles-and-capabilities.md` | Full capability matrix: which role (`student`, `landlord`, `verifier`, `admin`) can perform which action across all modules. | Technical Lead |
| `docs/decision-log.md` | Architecture Decision Records (ADRs). Tracks every significant technical decision with context, alternatives, and rationale. See [§11](#11-decision-log). | Technical Lead |
| `docs/history/phase-*.md` | Immutable record of each phase: what was planned, what was delivered, what decisions were made, what was deferred. | Technical Lead at phase completion |

### Rules

- `STATUS.md` is updated **within the same PR** that completes a task. Not after. Not tomorrow.
- `docs/history/` files are **append-only**. Once a phase is closed, its history file is never modified.
- `CLAUDE.md` is maintained **exclusively** by the Technical Lead. Developers do not modify it.
- `docs/decision-log.md` is **append-only**. New ADRs are added; existing ADRs are never edited (only superseded by a new ADR referencing the old one).

---

## 3. UniHome Technical Architecture

### Environment Requirements

| Requirement | Minimum Version | Rationale |
|---|---|---|
| **WordPress** | 6.4+ | Stable REST API, modern hook system, block editor compat |
| **PHP** | 8.0+ | Union types, named arguments, LTS support, namespace maturity |
| **MySQL** | 8.0+ | Native JSON support, window functions, CTEs for recommendation queries |
| **Node.js** | Not required | Vanilla JS, no build step |
| **Composer** | 2.x | PSR-4 autoloading for plugin classes |

WordPress Multisite is **out of scope** for the initial release. The `$wpdb->prefix` convention is followed throughout to preserve future Multisite compatibility if needed.

### System Layer Diagram

```
┌─────────────────────────────────────────────────┐
│                  User Browser                   │
│         (HTML / CSS / Vanilla JS / AJAX)        │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│            WordPress Theme Layer                │
│                                                 │
│  Templates · Page Routing · Asset Enqueuing     │
│  Shortcodes · Template Tags · Layout            │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│          UniHome Custom Plugin Layer            │
│                                                 │
│  ┌──────┐ ┌──────┐ ┌─────┐ ┌────┐ ┌──────────┐ │
│  │ Room │ │Verify│ │Match│ │Chat│ │Recommend │ │
│  │  Mod │ │  Mod │ │ Mod │ │Mod │ │   Mod    │ │
│  └──┬───┘ └──┬───┘ └──┬──┘ └─┬──┘ └────┬─────┘ │
│     │        │        │      │         │       │
│  ┌──┴────────┴────────┴──────┴─────────┴─────┐ │
│  │       Cross-Cutting Shared Services       │ │
│  │  Validation · Sanitization · Permissions  │ │
│  │  Logging · Config · Caching               │ │
│  │  Notification Service · Event Tracker     │ │
│  └──────────────────┬────────────────────────┘ │
└─────────────────────┬──────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│         WordPress Core Services                 │
│                                                 │
│  WP_User · wpdb · WP_REST_Server · Transients  │
│  wp_options · Media · WP-Cron · Roles/Caps     │
│  wp_mail                                        │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│              MySQL 8.0 Database                 │
│                                                 │
│  wp_users · wp_options · wp_posts (limited use) │
│  wp_unihome_rooms · wp_unihome_verifications    │
│  wp_unihome_notifications · wp_unihome_events   │
│  wp_unihome_audit_log                           │
└─────────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│            External Services                    │
│                                                 │
│  Payment Gateway    │  Google Maps API          │
│  Gemini API         │  Cloud Storage (future)   │
│  Email Service      │  Push Notifications (fut.)│
└─────────────────────────────────────────────────┘
```

### Layer Responsibilities

**User Browser Layer**
- Renders HTML served by WordPress theme templates.
- Sends REST API requests to the plugin layer for dynamic operations.
- No business logic lives here. The browser is a presentation terminal.

**WordPress Theme Layer**
- Controls visual presentation and page routing.
- Loads CSS/JS assets via `wp_enqueue_scripts`.
- Uses template hierarchy (`page-*.php`, `single-*.php`) for page rendering.
- Calls UniHome plugin functions via template tags or shortcodes.
- **Does not** contain business logic. If a template file has an `if` statement that decides business outcomes, it is in the wrong place.

**UniHome Custom Plugin Layer**
- Contains **all** UniHome business logic.
- Organized into discrete **business modules** (Room, Verification, Roommate, Chat, Recommendation, Payment, Marketplace, Ads).
- Each module exposes a **service class** (business logic) and a **controller class** (HTTP interface).
- Modules communicate through defined service interfaces, not by reaching into each other's internals.
- Each module is **isolated via error boundaries** — a fatal error in one module does not crash the entire plugin (see §5).
- **Cross-cutting shared services** (Notification, Event Tracker, Logging, Caching) live in `shared/` and are available to all modules.

**WordPress Core Services**
- Provides user authentication, session management, role/capability system.
- Provides `$wpdb` for database access.
- Provides `WP_REST_Server` for API routing.
- Provides media upload, transient caching, cron scheduling, `wp_mail`.
- We **consume** these services. We do **not** modify WordPress core.

**MySQL Database**
- Stores all persistent data.
- WordPress default tables are used for users, options, and limited post-type data.
- UniHome custom tables store structured business data (rooms, verifications, profiles, transactions).
- All business-critical state changes are logged in `wp_unihome_audit_log`.

**External Services**
- Payment Gateway — processes financial transactions (Stripe / local gateway TBD).
- Google Maps API — geocoding, map rendering for room locations.
- Gemini API — powers recommendation engine and potential AI-assisted matching.
- Cloud Storage — **future phase**. MVP uses WordPress local media storage (`wp-content/uploads/`).
- Email Service — MVP uses `wp_mail()`. SMTP transport or transactional email service (Mailgun/SES) is a production infrastructure decision.
- Push Notifications — **future phase**. Out of scope for initial release.

### Module Dependency Graph

Business modules may only depend on modules they have an explicit dependency arrow toward. **Circular dependencies are forbidden.** Cross-cutting shared services (Notification, Event Tracker, Logger, Cache) are available to all modules and are not shown in this graph.

```
                  ┌──────────────┐
                  │     Room     │  (no dependencies on other business modules)
                  └──────┬───────┘
                         │
          ┌──────────────┼──────────────────┐
          │              │                  │
          ▼              ▼                  ▼
   ┌──────────────┐ ┌──────────────┐ ┌───────────────┐
   │ Verification │ │Recommendation│ │    Payment     │
   └──────────────┘ └──────┬───────┘ └───────┬───────┘
                           │                 │
                           │                 ▼
                           │          ┌──────────────┐
                           │          │     Ads      │
                           │          └──────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │     Chat     │
                    └──────────────┘

   ┌──────────────┐
   │   Roommate   │  (no dependencies on other business modules)
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │     Chat     │  (also depends on Roommate for match context)
   └──────────────┘

   ┌──────────────┐
   │  Marketplace │  (no dependencies on other business modules)
   └──────────────┘
```

**Dependency summary (one line per module):**

| Module | Depends On |
|---|---|
| Room | — |
| Verification | Room |
| Roommate | — |
| Chat | Roommate |
| Recommendation | Room, Chat |
| Payment | Room |
| Ads | Payment |
| Marketplace | — |

**Rules:**
- A module may **only** call the Service class of a module it depends on.
- If module A depends on module B, module B **must never** depend on module A (no cycles).
- Cross-cutting shared services (Notification, Event Tracker, Logger, Cache, Config) live in `shared/` and may be called by any module. They are **not** business modules.

### Notification Service Architecture

Notification is a **cross-cutting shared service**, not a business module. It does not own business logic — it receives events from modules and delivers notifications to users.

```
Business Module (e.g., Verification)
    │
    │  fires event: 'room_approved'
    ▼
Notification Service (shared/)
    │
    ├── Check user notification preferences
    ├── Create notification record
    │
    ├──► Email         (wp_mail / SMTP)
    ├──► In-App        (wp_unihome_notifications table)
    └──► Push          (future — external service)
```

**Responsibilities:**

| Concern | Who Handles It |
|---|---|
| Deciding WHEN to notify | Business module (fires the event) |
| Deciding HOW to deliver | Notification Service (checks user preferences, dispatches to channels) |
| Creating notification content | Notification Service (templates per event type) |
| Storing notification records | Notification Service (`wp_unihome_notifications`) |
| Tracking delivery status | Notification Service |
| User preference management | Notification Service (stored in `wp_usermeta`) |

**Supported Events (initial set):**

| Event | Triggered By | Default Channels |
|---|---|---|
| `room_approved` | Verification module | Email, In-App |
| `room_rejected` | Verification module | Email, In-App |
| `new_match_found` | Roommate module | Email, In-App |
| `new_message` | Chat module | In-App |
| `payment_completed` | Payment module | Email, In-App |
| `listing_expired` | Room module (cron) | Email, In-App |
| `verification_assigned` | Verification module | Email, In-App |

**Key design decisions:**
- Modules call `NotificationService::notify( $event_type, $user_id, $context_data )`. They do NOT send emails directly.
- The Notification Service is synchronous for MVP. Async queue (via `wp_cron` or external queue) is a production optimization.
- User notification preferences are stored in `wp_usermeta` as a JSON object: `unihome_notification_prefs = {"email": true, "in_app": true, "push": false}`.

### Analytics / Event Tracking Architecture

Analytics is a **cross-cutting shared service** that collects user behavior data. It is a data pipeline, not a business logic engine.

```
User Action (e.g., views a room)
    │
    ▼
Event Tracker (shared/)
    │
    ├── Validate event type
    ├── Enrich with context (user_id, timestamp, session)
    ├── Write to wp_unihome_events
    │
    ▼
Consumers:
    ├── Recommendation Engine (reads events for scoring)
    ├── Admin Analytics Dashboard (future)
    └── Export to external analytics (future SaaS)
```

**Analytics does NOT contain business logic.** It records what happened. Other modules (Recommendation) consume the event data to make decisions.

**Tracked Events (initial set):**

| Event Name | Trigger | Data Captured |
|---|---|---|
| `room_viewed` | Student opens room detail | `room_id`, `user_id`, `duration` (future) |
| `room_saved` | Student saves/bookmarks a room | `room_id`, `user_id` |
| `room_search` | Student performs a search | `filters` (JSON), `result_count`, `user_id` |
| `contact_landlord` | Student initiates conversation | `room_id`, `landlord_id`, `user_id` |
| `match_viewed` | Student views match profile | `match_id`, `user_id` |
| `match_accepted` | Student accepts a match | `match_id`, `user_id` |
| `message_sent` | User sends a message | `conversation_id`, `user_id` |

**Key design decisions:**
- Events are written to `wp_unihome_events` (append-only table).
- The Event Tracker is a thin wrapper: `EventTracker::track( $event_name, $user_id, $data )`.
- Modules call the Event Tracker at the **Controller** level (after the action succeeds), not inside the Service layer. This keeps business logic pure.
- Event data is consumed by the Recommendation Engine via `EventRepository::get_user_events()`.
- For MVP, analytics is consumed internally only. Exporting to external analytics platforms (Mixpanel, BigQuery) is a future SaaS capability.

---

## 4. WordPress Development Approach

### Foundational Principle

> We are not building a WordPress website.  
> We are building an application on top of WordPress.

WordPress is chosen as the **application framework**, not the application itself. The decision is pragmatic:

- WordPress provides a mature authentication system, admin panel, media management, plugin lifecycle, and database abstraction out of the box.
- Building these from scratch in a custom PHP framework would cost 2-3 months of development time with no business value.
- WordPress's hook system (`add_action`, `add_filter`) provides a clean extension architecture.

This does **not** mean we use WordPress page builders, visual editors, or third-party marketplace plugins for business logic. Every UniHome feature is **custom-built**.

### WordPress Responsibilities

| Capability | WordPress Provides | UniHome Customizes |
|---|---|---|
| **Authentication** | Login, logout, session, password reset | Custom registration flows, role assignment |
| **User Roles** | Role/capability system (`add_role`, `current_user_can`) | Custom roles: `student`, `landlord`, `verifier`, `admin` |
| **Admin Dashboard** | `/wp-admin/` panel framework | Custom admin pages for room management, verification queue, analytics |
| **Media Upload** | `wp_handle_upload`, attachment system | Room photos, verification documents, profile pictures. Stored locally for MVP. |
| **Plugin Loading** | Plugin activation, deactivation, lifecycle hooks | UniHome plugin bootstrap, module initialization |
| **Security** | Nonce generation/verification, input sanitization functions, CSRF protection | Applied consistently in every endpoint |
| **Database** | `$wpdb` prepared statements, table prefix handling | Custom table creation, migration system |
| **Routing** | REST API route registration (`register_rest_route`) | Custom REST endpoints for all UniHome operations |
| **Cron** | `wp_schedule_event`, `wp_cron` | Scheduled tasks: recommendation recalculation, listing expiration, notification dispatch |
| **Transients** | `set_transient`, `get_transient` | Caching search results, recommendation scores, rate limiting |
| **Email** | `wp_mail()` | Notification delivery (verification results, new matches, payment confirmations) |

#### `wp_cron` Limitations

WordPress cron is **not real cron** — it is triggered by page loads, not by the system clock. This means:
- On low-traffic sites, scheduled tasks may be delayed by hours.
- On high-traffic sites, tasks may fire concurrently.

**Policy:**
- For **MVP/development**, `wp_cron` is acceptable.
- For **production**, a real system cron must be configured:
  ```
  * * * * * wget -q -O - https://example.com/wp-cron.php?doing_wp_cron > /dev/null 2>&1
  ```
  And WordPress pseudo-cron must be disabled in `wp-config.php`:
  ```php
  define( 'DISABLE_WP_CRON', true );
  ```
- For **time-critical tasks** (listing expiration, payment status polling), real system cron is mandatory even in staging.

### Custom Plugin Responsibilities

Everything that is UniHome-specific is implemented inside the `unihome-plugin`:

**Business Modules:**

| Module | Responsibility |
|---|---|
| **Room** | Room CRUD, listing lifecycle (draft → active → rented → archived), search/filter, pricing logic |
| **Verification** | Document upload, verification workflow (draft → pending → approved/rejected), trust scoring |
| **Roommate Matching** | Profile questionnaire, compatibility algorithm, match scoring, match suggestions |
| **Chat** | Messaging between students and landlords, conversation threading |
| **Recommendation** | Behavioral scoring model (rule-based → ML), personalized feed |
| **Payment** | Package definitions, subscription management, payment gateway integration, invoice generation |
| **Second-hand Marketplace** | Item listing, category management, item lifecycle |
| **Advertisement** | Ad placement zones, impression tracking, ad package management |

**Cross-Cutting Shared Services:**

| Service | Responsibility | Location |
|---|---|---|
| **Notification** | Event-driven notification dispatch (email, in-app). User preference management. | `shared/class-notification-service.php` |
| **Event Tracker** | Behavioral event recording for analytics and recommendation. | `shared/class-event-tracker.php` |
| **Logger** | Structured error/audit logging with module prefix. | `shared/class-logger.php` |
| **Cache Manager** | Transient wrapper with key naming convention and invalidation. | `shared/class-cache-manager.php` |
| **Config** | Environment configuration from `wp-config.php` constants. | `shared/class-config.php` |
| **Validator** | Input validation utilities. | `shared/class-validator.php` |
| **Sanitizer** | Input sanitization wrappers. | `shared/class-sanitizer.php` |

### What We Do NOT Use WordPress For

- **WooCommerce** — we do not use WooCommerce for payments. UniHome payment logic is custom.
- **Custom Post Types for core data** — rooms are NOT stored as `wp_posts`. See [Database Architecture](#6-database-architecture) for rationale.
- **Page builders** — no Elementor, WPBakery, or Gutenberg blocks for application UI.
- **Third-party plugins for business logic** — no "booking plugin" or "matching plugin." All business logic is ours.
- **WordPress theme customizer** — the theme is code-controlled, not admin-panel-controlled.

---

## 5. UniHome Plugin Architecture

### Autoloading Strategy

UniHome uses **Composer with PSR-4 autoloading**. This is a deliberate departure from the traditional WordPress `require_once` approach, chosen because:
- The project has 8+ modules with 3+ classes each — manual `require_once` chains become unmaintainable.
- Namespaces (`UniHome\Modules\Rooms\RoomService`) are unenforceable without an autoloader.
- Composer is already the standard in modern WordPress plugin development (ACF, WooCommerce, etc.).

**`composer.json` (plugin root):**
```json
{
    "name": "unihome/unihome-plugin",
    "autoload": {
        "psr-4": {
            "UniHome\\": "includes/"
        }
    },
    "require": {
        "php": ">=8.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^9.0",
        "yoast/phpunit-polyfills": "^2.0"
    }
}
```

**Bootstrap in `unihome.php`:**
```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Rule:** `vendor/` is committed to the repository (WordPress convention — no Composer on production servers). `.gitignore` does NOT exclude `vendor/`.

### Module Isolation Strategy

All business modules live inside one plugin. A PHP fatal error in one module must **not** crash the entire plugin. This is achieved through two mechanisms:

**1. Error Boundaries per Module**

Each module is initialized inside a try/catch in the plugin bootstrap:

```php
foreach ( $modules as $module_slug => $module_class ) {
    try {
        $module = new $module_class();
        $module->init();
    } catch ( \Throwable $e ) {
        error_log( "[UniHome:{$module_slug}] Module init failed: " . $e->getMessage() );
        // Module is skipped. Other modules continue loading.
    }
}
```

**2. Feature Flags**

Individual modules can be disabled from `wp_options` without code changes:

```php
// In wp-admin > UniHome Settings
// unihome_disabled_modules = ['ads', 'marketplace']

if ( in_array( $module_slug, get_option( 'unihome_disabled_modules', [] ), true ) ) {
    continue; // Skip this module
}
```

This allows the team to disable a broken module in production without a code deploy.

### Directory Structure

```
wp-content/plugins/unihome-plugin/
│
├── unihome.php                        # Plugin entry point. Bootstrap only.
├── composer.json                      # PSR-4 autoload, dev dependencies
├── vendor/                            # Composer autoloader (committed)
├── phpunit.xml                        # Test runner configuration
│
├── includes/
│   ├── class-unihome-loader.php       # Hook registration orchestrator
│   ├── class-unihome-activator.php    # Activation logic (DB tables, roles, defaults)
│   ├── class-unihome-deactivator.php  # Deactivation cleanup
│   │
│   ├── database/
│   │   ├── schema.php                 # Table definitions (CREATE TABLE statements)
│   │   ├── migrations.php             # Versioned migration runner
│   │   ├── foreign-keys.php           # FK constraints applied post-dbDelta via ALTER TABLE
│   │   └── seeder.php                 # Development seed data
│   │
│   ├── modules/
│   │   ├── rooms/
│   │   │   ├── class-room-service.php
│   │   │   ├── class-room-controller.php
│   │   │   └── class-room-repository.php
│   │   │
│   │   ├── verification/
│   │   │   ├── class-verification-service.php
│   │   │   ├── class-verification-controller.php
│   │   │   └── class-verification-repository.php
│   │   │
│   │   ├── roommate/
│   │   │   ├── class-matching-engine.php
│   │   │   ├── class-roommate-service.php
│   │   │   ├── class-roommate-controller.php
│   │   │   └── class-roommate-repository.php
│   │   │
│   │   ├── chat/
│   │   │   ├── class-message-service.php
│   │   │   ├── class-chat-controller.php
│   │   │   └── class-message-repository.php
│   │   │
│   │   ├── recommendation/
│   │   │   ├── class-recommendation-engine.php
│   │   │   ├── class-recommendation-service.php
│   │   │   └── class-recommendation-controller.php
│   │   │
│   │   ├── payment/
│   │   │   ├── class-package-service.php
│   │   │   ├── class-payment-gateway.php
│   │   │   ├── class-payment-controller.php
│   │   │   └── class-payment-repository.php
│   │   │
│   │   ├── marketplace/
│   │   │   ├── class-item-service.php
│   │   │   ├── class-marketplace-controller.php
│   │   │   └── class-item-repository.php
│   │   │
│   │   └── ads/
│   │       ├── class-ad-service.php
│   │       ├── class-ad-controller.php
│   │       └── class-ad-repository.php
│   │
│   └── shared/
│       ├── class-validator.php
│       ├── class-sanitizer.php
│       ├── class-response-formatter.php
│       ├── class-permission-checker.php
│       ├── class-config.php
│       ├── class-logger.php
│       ├── class-cache-manager.php
│       ├── class-notification-service.php    # Cross-cutting: event-driven notifications
│       ├── class-notification-repository.php # Data access for notifications table
│       └── class-event-tracker.php           # Cross-cutting: behavioral event recording
│
├── api/
│   └── class-rest-router.php
│
├── admin/
│   ├── class-admin-pages.php
│   ├── views/
│   └── assets/
│
├── assets/
│   ├── js/
│   │   ├── room-listing.js
│   │   ├── room-detail.js
│   │   ├── search.js
│   │   ├── chat.js
│   │   └── matching.js
│   └── css/
│       ├── main.css
│       └── components/
│
├── templates/
│   ├── room-card.php
│   ├── search-filters.php
│   └── profile-form.php
│
└── tests/
    ├── bootstrap.php
    ├── modules/
    │   ├── rooms/
    │   │   ├── test-room-service.php
    │   │   └── test-room-repository.php
    │   ├── verification/
    │   │   └── test-verification-service.php
    │   └── ...
    └── shared/
        ├── test-validator.php
        └── test-notification-service.php
```

### Module Architecture Pattern

Every business module follows the same three-layer pattern:

```
Controller (HTTP boundary — REST API)
    │
    ▼
Service (Business logic — returns WP_Error on failure)
    │
    ▼
Repository (Data access — $wpdb only)
```

#### Why This Pattern Is Appropriate for WordPress

This pattern was reviewed for suitability given the team composition (student-level developers + AI agents) and the WordPress context. The decision is to **keep it**, with the following rationale:

**Why not simplify to 2 layers (Controller + Service)?**
- Without a Repository layer, SQL queries leak into Service classes. When the database schema changes, you must read business logic to find all affected queries. With 4 developers and AI agents working in parallel, this creates constant merge conflicts and broken queries.
- The Repository layer is the **thinnest** of the three — most methods are 5-10 lines wrapping a `$wpdb->prepare()` call. The overhead is minimal.

**Why not go more enterprise (DTOs, Interfaces, Dependency Injection)?**
- WordPress does not have a service container. Introducing one (e.g., PHP-DI) adds framework complexity that conflicts with WordPress's hook-based architecture.
- DTOs add boilerplate without meaningful benefit at this scale. Plain arrays and `WP_Error` are sufficient.
- Interfaces for every service would be appropriate if we had multiple implementations (e.g., swapping payment gateways). We define interfaces **only when there is a concrete second implementation** — not preemptively.

**Pragmatic guidelines for the team:**

| Situation | Pattern |
|---|---|
| Module has REST endpoints + business logic + database queries | **Controller → Service → Repository** (full pattern) |
| Shared service with no REST endpoints (Notification, EventTracker) | **Service + Repository** (no Controller — called by other modules directly) |
| Simple utility with no state (Validator, Sanitizer) | **Static class or pure functions** (no pattern needed) |
| Module in early phase with only 1-2 endpoints | Start with **Controller → Service → Repository** anyway. Consistency across modules is more valuable than saving 1 file. |

**Controller** — receives `WP_REST_Request` objects, validates input using `shared/class-validator.php`, calls the service, formats the response. Controllers never contain business logic. Controllers never write SQL.

**Service** — contains all business rules, state transitions, and orchestration. Services call repositories for data access. Services call other services (not other controllers or repositories) for cross-module operations, **respecting the dependency direction graph**. Services return data on success and `WP_Error` on failure. Services **never throw exceptions**.

**Repository** — encapsulates all `$wpdb` interactions. Returns plain PHP objects or arrays. Repositories never enforce business rules. If a query changes, only the repository changes.

---

## 6. Database Architecture

### Why Not Just `wp_posts` and `wp_postmeta`

WordPress developers often store custom data as Custom Post Types using `wp_posts` and `wp_postmeta`. This approach has critical limitations for UniHome:

| Concern | wp_posts / wp_postmeta | Custom Tables |
|---|---|---|
| **Query performance** | `wp_postmeta` stores key-value pairs. Filtering rooms by price AND location AND availability requires multiple JOINs on the same table. | Direct column queries. Standard WHERE clauses. Orders of magnitude faster. |
| **Data integrity** | No column types. Price is stored as a string. No foreign keys. No constraints. | Typed columns. Foreign keys. CHECK constraints. Data integrity enforced at the database level. |
| **Schema clarity** | A room's "price" is a row in `wp_postmeta` with `meta_key = 'price'`. Invisible to schema inspection tools. | `wp_unihome_rooms.price DECIMAL(15,2)`. Self-documenting. |
| **Indexing** | Cannot create targeted indexes on business fields without hacking `wp_postmeta`. | Standard MySQL indexes on any column. |
| **Migration** | Schema changes require meta key renames and data migration scripts that iterate over rows. | Standard `ALTER TABLE` statements. |

**Decision:** UniHome uses custom tables for all business data. WordPress default tables are used only for what WordPress already manages (users, options, basic media references).

### WordPress Default Tables (Used As-Is)

| Table | UniHome Usage |
|---|---|
| `wp_users` | All user accounts: students, landlords, verifiers, admins. Extended via `wp_usermeta` for profile fields that don't warrant a custom table. |
| `wp_usermeta` | User preferences, notification preferences, onboarding state. |
| `wp_options` | Plugin configuration, feature flags, API keys (encrypted), global settings. |
| `wp_posts` | **Limited use only.** Static content pages (About, Terms, Privacy Policy). NOT used for rooms, items, or any business entity. |

### Deletion Policy

| Entity | Strategy | Rationale |
|---|---|---|
| Rooms | **Soft delete** via `status='archived'` | Preserved for verification history, message context, audit trail. |
| Users | **Never deleted** | WordPress convention. Disabled via role removal if needed. |
| Verifications | **Never deleted** | Legal and trust audit requirements. |
| Messages | **Never deleted** | Legal retention and dispute resolution. |
| Matches | **Retained** | Historical record. Status set to `'declined'` or `'expired'`. |
| Payments | **Never deleted** | Financial audit trail. |
| Notifications | **Retained** | User can mark as read but not delete from database. |
| Events | **Never deleted** | Append-only analytics data. Purge policy defined post-launch. |
| Marketplace Items | **Soft delete** via `deleted_at` | Preserved for transaction history. |
| Ads | **Soft delete** via `deleted_at` | Impression history preserved for billing. |

### Custom UniHome Tables

#### `wp_unihome_rooms`

Stores structured room listing data.

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `owner_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `title` | `VARCHAR(255)` | Room listing title |
| `description` | `TEXT` | Full description |
| `address` | `VARCHAR(500)` | Full address string |
| `latitude` | `DECIMAL(10,8)` | Geocoded latitude |
| `longitude` | `DECIMAL(11,8)` | Geocoded longitude |
| `price` | `DECIMAL(15,2)` | Monthly rent (VND stored as whole number with .00) |
| `currency` | `CHAR(3) DEFAULT 'VND'` | ISO 4217 currency code |
| `room_type` | `ENUM('single','shared','studio','apartment')` | Room classification |
| `status` | `ENUM('draft','pending_review','active','rented','archived')` | Listing lifecycle state |
| `amenities` | `JSON` | Structured amenities data |
| `max_occupants` | `TINYINT UNSIGNED` | Maximum occupants |
| `available_from` | `DATE` | Availability start date |
| `created_at` | `DATETIME` | Record creation timestamp |
| `updated_at` | `DATETIME` | Last modification timestamp |

Indexes: `owner_id`, `status`, `(latitude, longitude)`, `price`, `room_type`

---

#### `wp_unihome_room_images`

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `room_id` | `BIGINT UNSIGNED` | FK → `wp_unihome_rooms.id` |
| `attachment_id` | `BIGINT UNSIGNED` | FK → `wp_posts.ID` (WordPress media) |
| `sort_order` | `TINYINT UNSIGNED` | Display order |
| `is_primary` | `TINYINT(1) DEFAULT 0` | Primary listing image flag |

Indexes: `room_id`

---

#### `wp_unihome_verifications`

Stores verification workflow state.

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `room_id` | `BIGINT UNSIGNED` | FK → `wp_unihome_rooms.id` |
| `verifier_id` | `BIGINT UNSIGNED NULL` | FK → `wp_users.ID` (assigned verifier) |
| `status` | `ENUM('draft','pending_review','in_review','approved','rejected')` | Verification state |
| `document_urls` | `JSON` | Uploaded verification documents |
| `rejection_reason` | `TEXT NULL` | Reason if rejected |
| `verified_at` | `DATETIME NULL` | Approval timestamp |
| `created_at` | `DATETIME` | Submission timestamp |
| `updated_at` | `DATETIME` | Last state change |

Indexes: `room_id`, `verifier_id`, `status`

---

#### `wp_unihome_roommate_profiles`

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `user_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `sleep_schedule` | `ENUM('early_bird','night_owl','flexible')` | Sleep preference |
| `noise_tolerance` | `TINYINT UNSIGNED` | 1-5 scale |
| `cleanliness` | `TINYINT UNSIGNED` | 1-5 scale |
| `guest_policy` | `ENUM('no_guests','occasional','frequent')` | Guest preference |
| `smoking` | `TINYINT(1)` | Smoking flag |
| `pet_friendly` | `TINYINT(1)` | Pet tolerance flag |
| `budget_min` | `DECIMAL(15,2)` | Budget range low |
| `budget_max` | `DECIMAL(15,2)` | Budget range high |
| `preferred_location` | `VARCHAR(255)` | Preferred area |
| `bio` | `TEXT` | Free-text description |
| `created_at` | `DATETIME` | Profile creation |
| `updated_at` | `DATETIME` | Last update |

Indexes: `user_id` (UNIQUE)

---

#### `wp_unihome_matches`

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `user_a_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` (always the lower ID) |
| `user_b_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` (always the higher ID) |
| `compatibility_score` | `DECIMAL(5,2)` | Computed score (0-100) |
| `status` | `ENUM('suggested','viewed','accepted','declined')` | Match state |
| `computed_at` | `DATETIME` | Score computation timestamp |

Indexes: `(user_a_id, user_b_id)` (UNIQUE), `compatibility_score`

**Convention:** `user_a_id < user_b_id` always. This canonicalizes pair ordering and prevents duplicate match records.

---

#### `wp_unihome_conversations`

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `room_id` | `BIGINT UNSIGNED NULL` | FK → `wp_unihome_rooms.id` (optional room context) |
| `participant_a_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `participant_b_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `last_message_at` | `DATETIME NULL` | Denormalized for sort performance |
| `status` | `ENUM('active','archived')` | Conversation state |
| `created_at` | `DATETIME` | Creation timestamp |

Indexes: `(participant_a_id, participant_b_id)` (UNIQUE), `participant_a_id`, `participant_b_id`, `last_message_at`

---

#### `wp_unihome_messages`

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `conversation_id` | `BIGINT UNSIGNED` | FK → `wp_unihome_conversations.id` |
| `sender_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `message` | `TEXT` | Message content |
| `is_read` | `TINYINT(1) DEFAULT 0` | Read status |
| `created_at` | `DATETIME` | Sent timestamp |

Indexes: `(conversation_id, created_at)`, `(sender_id)`, `(conversation_id, is_read)`

---

#### `wp_unihome_notifications`

Stores in-app notification records.

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `user_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` (recipient) |
| `event_type` | `VARCHAR(50)` | `'room_approved'`, `'new_match_found'`, etc. |
| `title` | `VARCHAR(255)` | Notification title |
| `body` | `TEXT` | Notification body |
| `context` | `JSON NULL` | Related entity data (`room_id`, `match_id`, etc.) |
| `is_read` | `TINYINT(1) DEFAULT 0` | Read status |
| `created_at` | `DATETIME` | Notification timestamp |

Indexes: `(user_id, is_read, created_at)`, `event_type`

---

#### `wp_unihome_events`

Append-only behavioral event log for analytics and recommendation.

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `user_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `event_name` | `VARCHAR(50)` | `'room_viewed'`, `'room_saved'`, `'room_search'`, etc. |
| `event_data` | `JSON NULL` | Event-specific payload |
| `created_at` | `DATETIME` | Event timestamp |

Indexes: `(user_id, event_name)`, `(event_name, created_at)`, `created_at`

**Rules:**
- This table is **append-only**. Rows are never updated or deleted.
- Purge policy (e.g., events older than 12 months) will be defined post-launch based on table size.

---

#### `wp_unihome_packages`

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `name` | `VARCHAR(100)` | Display name |
| `slug` | `VARCHAR(100)` | URL-safe identifier (UNIQUE) |
| `type` | `ENUM('room_boost','featured','premium_landlord','ad_package')` | Package category |
| `price` | `DECIMAL(15,2)` | Package price |
| `currency` | `CHAR(3) DEFAULT 'VND'` | ISO 4217 |
| `duration_days` | `INT UNSIGNED` | Package validity period |
| `features` | `JSON` | Package feature flags and limits |
| `is_active` | `TINYINT(1) DEFAULT 1` | Available for purchase |
| `created_at` | `DATETIME` | Creation timestamp |
| `updated_at` | `DATETIME` | Last modification |

Indexes: `slug` (UNIQUE), `type`, `is_active`

---

#### `wp_unihome_payments`

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `user_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `package_id` | `BIGINT UNSIGNED` | FK → `wp_unihome_packages.id` |
| `amount` | `DECIMAL(15,2)` | Charged amount |
| `currency` | `CHAR(3)` | ISO 4217 |
| `gateway` | `VARCHAR(50)` | Payment gateway identifier |
| `gateway_transaction_id` | `VARCHAR(255)` | External transaction reference |
| `status` | `ENUM('pending','completed','failed','refunded')` | Payment state |
| `created_at` | `DATETIME` | Transaction timestamp |

Indexes: `user_id`, `package_id`, `status`, `gateway_transaction_id`

---

#### `wp_unihome_audit_log`

Immutable log of all business-critical state changes.

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `entity_type` | `VARCHAR(50)` | `'room'`, `'verification'`, `'payment'`, `'match'`, etc. |
| `entity_id` | `BIGINT UNSIGNED` | ID of the affected record |
| `action` | `VARCHAR(50)` | `'status_changed'`, `'created'`, `'updated'`, `'deleted'` |
| `old_value` | `JSON NULL` | Previous state |
| `new_value` | `JSON NULL` | New state |
| `performed_by` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `ip_address` | `VARCHAR(45)` | IPv4 or IPv6 address |
| `created_at` | `DATETIME` | Timestamp |

Indexes: `(entity_type, entity_id)`, `performed_by`, `created_at`

**Note:** Table schemas for marketplace items and ads modules will be defined in their respective phase documents.

---

### Database Migration Rules

1. **Versioned migrations.** Every schema change is a numbered migration. The `migrations.php` runner tracks applied migrations via `wp_options`.
2. **Forward-only.** Migrations are never modified after commit. Create a new migration to change a table.
3. **Activation hook.** `class-unihome-activator.php` runs all pending migrations using `dbDelta()` for initial creation and raw SQL for alterations.
4. **Foreign Key constraints.** `dbDelta()` does **not** support FK constraints (silently drops them). FK constraints are applied in a **separate step** using raw `$wpdb->query("ALTER TABLE ... ADD CONSTRAINT ...")` in `foreign-keys.php`. All tables must use **InnoDB**.
5. **No manual SQL.** All schema changes go through the migration system.
6. **Table prefix.** All custom tables use `{$wpdb->prefix}unihome_` prefix. Never hardcode `wp_`.

---

## 7. Development Rules

### Before Writing Any Code

Every developer **must**:

1. **Read `STATUS.md`** — confirm the current phase and in-progress tasks.
2. **Read the relevant module documentation** — understand responsibilities and interfaces.
3. **Check `docs/database.md`** — confirm table schemas exist and are current.
4. **Review existing code in the target module** — follow established patterns.
5. **Confirm your task assignment** — do not work on unassigned tasks without coordination.

### PHP Coding Standards

- Follow [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/) for formatting (indentation, spacing, brace style).
- **Namespace convention:** We use PHP namespaces (PSR-4 style) which departs from traditional WordPress class prefixing. This is a deliberate decision for code organization at our scale. The WordPress Coding Standards apply to formatting, not naming.
- Class files: `class-{name}.php` (e.g., `class-room-service.php`).
- One class per file.
- Namespaces: `namespace UniHome\Modules\Rooms;`
- Type hints on all method parameters and return types (PHP 8.0+).
- No global functions for business logic. All logic lives in classes.

### REST API Endpoint Rules (Primary Pattern)

UniHome uses the **WordPress REST API** (`register_rest_route`) as the primary HTTP interface. Legacy AJAX (`admin-ajax.php`) is **not used** for new endpoints.

Every REST endpoint **must**:

```php
// In class-rest-router.php — route registration
register_rest_route( 'unihome/v1', '/rooms', [
    'methods'             => WP_REST_Server::CREATABLE,
    'callback'            => [ $this->room_controller, 'create_room' ],
    'permission_callback' => [ $this->room_controller, 'can_create_room' ],
] );

// In class-room-controller.php — permission callback
public function can_create_room( WP_REST_Request $request ): bool {
    return current_user_can( 'unihome_create_room' );
}

// In class-room-controller.php — endpoint handler
public function create_room( WP_REST_Request $request ): WP_REST_Response {
    // 1. Sanitize input
    $title = sanitize_text_field( $request->get_param( 'title' ) );
    $price = floatval( $request->get_param( 'price' ) );

    // 2. Validate via shared Validator
    $validation = $this->validator->validate_room_data( $data );
    if ( is_wp_error( $validation ) ) {
        return new WP_REST_Response( [ 'message' => $validation->get_error_message() ], 400 );
    }

    // 3. Execute business logic via Service
    $result = $this->room_service->create_room( $data );

    // 4. Track event (analytics)
    if ( ! is_wp_error( $result ) ) {
        EventTracker::track( 'room_created', get_current_user_id(), [ 'room_id' => $result['id'] ] );
    }

    // 5. Return response
    if ( is_wp_error( $result ) ) {
        return new WP_REST_Response( [ 'message' => $result->get_error_message() ], $result->get_error_data() ?: 500 );
    }
    return new WP_REST_Response( [ 'room' => $result ], 201 );
}
```

### Error Handling Conventions

- **Services** return `WP_Error` on failure, never throw exceptions.
- **Repositories** return `false`/`null` on not-found, `WP_Error` on database errors.
- **Controllers** translate `WP_Error` into appropriate HTTP status codes.
- All errors logged via `Logger::error( 'module_name', 'message' )`.
- **Cron job failures** logged and surfaced as admin notice.
- Never use `die()`, `exit()`, or `wp_die()` in Service or Repository classes.

### Environment Configuration

Environment-specific values managed through `wp-config.php` constants, wrapped by `shared/class-config.php`:

```php
// In wp-config.php
define( 'UNIHOME_ENV', 'development' );
define( 'UNIHOME_PAYMENT_MODE', 'sandbox' );
define( 'UNIHOME_MAPS_API_KEY', 'AIza...' );
define( 'UNIHOME_GEMINI_API_KEY', 'AIza...' );
```

**Rule:** Secrets are **never** committed. They live only in `wp-config.php` or server environment variables.

### Testing Strategy

| Test Type | Tool | Location | Scope |
|---|---|---|---|
| **Unit tests** | PHPUnit | `tests/modules/{module}/test-{class}.php` | Service classes |
| **Integration tests** | WP_UnitTestCase | `tests/modules/{module}/test-{class}.php` | Repository classes |
| **Shared utils** | PHPUnit | `tests/shared/test-{class}.php` | Validator, Notification, EventTracker |

**Rules:**
- Every Service class public method must have at least one test.
- AI-generated code **must include tests** or it is rejected in review.
- Tests run via `vendor/bin/phpunit` from the plugin root.

### Forbidden Practices

| Rule | Reason |
|---|---|
| **Never put business logic in JavaScript** | Business decisions happen server-side. |
| **Never modify database schema manually** | All changes through migration system. |
| **Never create PHP files outside module structure** | Files belong in module dirs, `shared/`, or `admin/`. |
| **Never use `global $post` for business data** | Use Repository classes. |
| **Never call `$wpdb` outside Repository class** | Data access centralized in repositories. |
| **Never hardcode table names** | Use `$wpdb->prefix . 'unihome_rooms'`. |
| **Never commit secrets** | Secrets in `wp-config.php` only. |
| **Never bypass the Service layer** | Controllers → Services → Repositories. |
| **Never use `admin-ajax.php`** | REST API only. |
| **Never send notifications directly** | Use `NotificationService::notify()`. |

### Git Workflow

- `main` — production. Protected. PR only.
- `develop` — integration branch.
- `feature/{module}/{description}` — feature branches.
- Every PR requires 1 code review.
- Every PR updates `STATUS.md` if task status changed.
- Commit format: `type(module): description` (e.g., `feat(rooms): add CRUD endpoints`).

### Code Review Checklist

- [ ] Follows Controller → Service → Repository pattern
- [ ] Permission callback on all REST routes
- [ ] Input sanitized with WordPress functions
- [ ] No SQL outside Repository classes
- [ ] No business logic in Controller or JavaScript
- [ ] `$wpdb->prepare()` for all user input
- [ ] `WP_Error` returned (not exceptions thrown)
- [ ] Audit log for state changes
- [ ] Notifications via `NotificationService::notify()`
- [ ] Events tracked via `EventTracker::track()` where applicable
- [ ] Tests included for new Service methods
- [ ] `STATUS.md` updated
- [ ] No hardcoded table names or secrets
- [ ] Module dependency direction respected

---

## 8. Development Phase Proposal

### Phase Ordering Rationale

v0.3.0 reorders the development phases. The key change: **Communication (Chat) moves before Recommendation**.

**Why:**

The Recommendation Engine's value depends on behavioral signals. The richest signals come from user interactions:
- `contact_landlord` — indicates strong interest in a room
- `message_sent` — indicates active engagement
- Conversation patterns reveal which room attributes drive real action vs. passive browsing

Without Chat data, Phase 5 (Recommendation) launches with only search and view signals — a significantly weaker dataset. By building Chat first, the recommendation engine starts with a richer signal set from day one.

**New phase order:**

| Phase | Module | Rationale |
|---|---|---|
| 0 | Foundation | Infrastructure must come first |
| 1 | Room Marketplace | Core product — everything depends on rooms existing |
| 2 | Verification | Trust layer on top of rooms — required for marketplace credibility |
| 3 | Roommate Matching | Expands product from room search to social matching |
| 4 | Communication | Enables user-to-user interaction; generates behavioral signals |
| 5 | Recommendation | Consumes signals from phases 1-4 for personalized ranking |
| 6 | Monetization | Revenue layer — deferred until product is validated |
| 7 | Marketplace Expansion | Second-hand marketplace — independent product line |
| 8 | Advertisement | Revenue diversification — requires user base scale |

---

### Phase 0 — Foundation

**Goal:** Establish the WordPress application framework and development infrastructure.

**Duration:** ~3 weeks (0a: TL-only, 1 week. 0b: full team, 2 weeks.)

**Tasks:**

| Task | Owner | Phase |
|---|---|---|
| WordPress installation and configuration | TL | 0a |
| Setup `composer.json` with PSR-4 autoloading | TL | 0a |
| Create `unihome-plugin` scaffold with module directories | TL | 0a |
| Implement database migration system (including FK workaround) | TL | 0a |
| Setup PHPUnit and `WP_UnitTestCase` bootstrap | TL | 0a |
| Write `CLAUDE.md`, `STATUS.md`, initial `docs/` files | TL | 0a |
| Setup Git repository and branch protection rules | TL | 0a |
| Define custom user roles + capabilities | Dev 1 | 0b |
| Create base theme with template structure | Dev 2 | 0b |
| Setup REST API router + health-check endpoint | Dev 3 | 0b |
| Create shared utilities (Validator, Sanitizer, Logger, Config, NotificationService scaffold, EventTracker scaffold) | Dev 3 | 0b |
| Create module isolation bootstrap (error boundaries + feature flags) | Dev 4 | 0b |
| Write `docs/environment-setup.md`, `docs/security.md`, `docs/decision-log.md` | Dev 4 | 0b |

**Completion Criteria:**
- Plugin activates without errors. Composer autoloader resolves all classes.
- Migration system creates tables and applies FK constraints.
- REST API responds to `/wp-json/unihome/v1/health`.
- PHPUnit runs with at least one passing test.
- Module isolation works (disabling a module via flag doesn't break others).
- NotificationService and EventTracker scaffolds exist (empty implementations, correct interfaces).
- All documentation files in place.

---

### Phase 1 — Room Marketplace

**Goal:** Landlords can list rooms, students can search and view them, admins can manage listings.

**Duration:** ~4 weeks

**Tasks:**

| Area | Task | Owner |
|---|---|---|
| Database | `wp_unihome_rooms`, `wp_unihome_room_images` tables | Dev 1 |
| Backend | Room Service — CRUD, status transitions, audit logging | Dev 1 |
| Backend | Room Repository — SQL queries, search filters | Dev 2 |
| Backend | Room Controller — REST endpoints | Dev 2 |
| Backend | Image upload with WordPress media library | Dev 3 |
| Frontend | Room listing page — grid/list view, pagination | Dev 3 |
| Frontend | Room detail page — gallery, map, contact CTA | Dev 4 |
| Frontend | Search and filter interface | Dev 4 |
| Admin | Room management page | Dev 1 |
| Testing | RoomService + RoomRepository tests | Dev 2 |

**Completion Criteria:**
- Landlord can create room listings with images.
- Student can search/filter/view rooms.
- Admin can manage listings.
- Room status changes generate audit log entries.
- `EventTracker::track('room_viewed', ...)` wired in room detail controller.

---

### Phase 2 — Room Verification

**Goal:** Trust system — rooms verified through review workflow.

**Duration:** ~3 weeks

**Verification Workflow:**

```
Draft → Pending Review → In Review → Approved / Rejected
                                         │
                                    Rejected → Draft (resubmit)
                                    Approved → Active (verified badge)
```

**Tasks:**

| Area | Task | Owner |
|---|---|---|
| Database | `wp_unihome_verifications` table | Dev 2 |
| Backend | Verification Service — state machine, audit logging | Dev 2 |
| Backend | Verification Controller — submit, review, approve/reject | Dev 1 |
| Backend | Notification integration — `room_approved` / `room_rejected` events | Dev 3 |
| Frontend | Landlord verification form | Dev 3 |
| Admin | Verifier queue — approve/reject UI | Dev 4 |
| Testing | VerificationService state machine tests | Dev 1 |

**Completion Criteria:**
- Full verification workflow functional.
- Verified badge on approved rooms.
- Landlord receives notification on approval/rejection.

---

### 🔄 Integration Checkpoint 1 (1 week)

After Phase 2:
- Room + Verification end-to-end integration testing.
- Notification Service validation — emails actually send for verification events.
- Performance profiling — slow query identification.
- Security audit — capability coverage on all endpoints.
- Documentation catchup.

---

### Phase 3 — Roommate Matching

**Goal:** Students find compatible roommates.

**Duration:** ~3 weeks

**Tasks:**

| Area | Task | Owner |
|---|---|---|
| Database | `wp_unihome_roommate_profiles`, `wp_unihome_matches` tables | Dev 3 |
| Backend | Profile Service — questionnaire save/update | Dev 3 |
| Backend | Matching Engine — compatibility scoring | Dev 1 |
| Backend | Match Controller — get/accept/decline matches | Dev 1 |
| Backend | Notification — `new_match_found` event | Dev 2 |
| Frontend | Profile questionnaire form | Dev 4 |
| Frontend | Match results page — compatibility %, profile cards | Dev 2 |
| Testing | MatchingEngine scoring tests | Dev 1 |

**Matching Algorithm (v1 — Rule-Based):**

```
score = 0
if sleep_schedule matches:    score += 20
if noise_tolerance within 1:  score += 15
if cleanliness within 1:      score += 15
if guest_policy matches:      score += 15
if smoking matches:           score += 10
if pet_friendly matches:      score += 10
if budget overlaps:           score += 15
```

**Completion Criteria:**
- Student fills out profile, sees matches with scores.
- Accept/decline functional. Match uniqueness enforced.
- Notification sent for new matches.

---

### Phase 4 — Communication System

**Goal:** Enable messaging between students and landlords. Generate behavioral signals for recommendation.

**Duration:** ~3 weeks

**Tasks:**

| Area | Task | Owner |
|---|---|---|
| Database | `wp_unihome_conversations`, `wp_unihome_messages` tables | Dev 4 |
| Backend | Message Service — send, read, conversation management | Dev 4 |
| Backend | Chat Controller — REST endpoints for messaging | Dev 1 |
| Backend | Notification — `new_message` event | Dev 2 |
| Backend | Event tracking — `contact_landlord`, `message_sent` | Dev 2 |
| Frontend | Chat interface — conversation list, message thread | Dev 3 |
| Frontend | "Contact Landlord" button integration on room detail | Dev 3 |
| Testing | MessageService tests | Dev 1 |

**Completion Criteria:**
- Student can message landlord from room detail page.
- Conversation threading works.
- Notification for new messages.
- `contact_landlord` and `message_sent` events tracked for recommendation.

---

### Phase 5 — Recommendation Engine

**Goal:** Personalized room recommendations using behavioral signals from phases 1-4.

**Duration:** ~4 weeks

**Phase 5a — Rule-Based Scoring:**

| Signal | Weight | Source |
|---|---|---|
| Price match | 20% | Profile budget vs. room price |
| Location match | 20% | Profile location vs. room address |
| Room type match | 10% | Profile preference vs. room type |
| Recency | 10% | Room `created_at` |
| Verification status | 10% | Room verification badge |
| Landlord rating | 10% | (future, placeholder weight) |
| Search similarity | 10% | `room_search` events |
| Contact signal | 10% | `contact_landlord` events (from Phase 4) |

**Phase 5b — Behavioral Tracking (Future):**

Full behavioral model via Gemini API. Consumes all events from `wp_unihome_events`.

**Tasks:**

| Area | Task | Owner |
|---|---|---|
| Database | Recommendation scoring/cache tables | Dev 4 |
| Backend | Recommendation Engine — rule-based scoring | Dev 1 |
| Backend | EventRepository — read behavioral events | Dev 2 |
| Backend | Recommendation Controller — personalized feed | Dev 2 |
| Frontend | "Recommended for You" section | Dev 3 |
| Research | Gemini API integration prototype | Dev 4 |
| Testing | RecommendationEngine scoring tests | Dev 1 |

**Completion Criteria:**
- Student sees personalized recommendations on dashboard.
- Recommendations factor in preferences, budget, location, AND behavioral signals.

---

### 🔄 Integration Checkpoint 2 (1 week)

After Phase 5:
- Full end-to-end testing across 5 modules.
- Performance profiling (1000+ rooms, 500+ users, 10k+ events).
- Security audit — complete capability review.
- Database clean-install verification.
- Team retrospective for Phase 6+.

---

### Phase 6 — Monetization

**Goal:** Revenue layer — packages, payment gateway, subscription management.

**Duration:** ~4 weeks

#### Payment Strategy

Payment is intentionally deferred to Phase 6. This is a **business decision**: build user base and validate product-market fit before monetizing.

**Full monetization (Phase 6) includes:**
- Payment gateway integration (Stripe / local gateway)
- Package purchase flow
- Subscription management
- Invoice generation
- Payment status tracking

**MVP-lite monetization option:**

If the business requires earlier revenue validation, a lightweight monetization can be introduced as a Phase 4.5 side-task without disrupting the main roadmap:

| MVP-Lite Feature | Implementation | Effort |
|---|---|---|
| Package database | Create `wp_unihome_packages` table, seed with initial packages | 2 days |
| "Boost" listing badge | Flag on `wp_unihome_rooms` + sort priority in search | 2 days |
| Manual payment confirmation | Admin marks payment as "confirmed" manually after bank transfer | 1 day |
| No gateway integration | Payment instructions displayed; admin confirms offline | 0 |

This gives the product a monetization path for early validation (~1 week effort) without the complexity of gateway integration. The full payment module in Phase 6 replaces this MVP-lite flow.

**Recommendation:** Do **not** prioritize monetization over product validation. The MVP-lite option exists as a safety valve if the business demands it, but the default plan is Phase 6.

---

### Future Phases (Not Scoped)

| Phase | Module | Notes |
|---|---|---|
| Phase 7 | Marketplace Expansion | Second-hand item listings, categories, student-to-student. |
| Phase 8 | Advertisement | Ad zones, impression tracking, ad packages. |

---

## 9. Example STATUS.md Preview

```markdown
# UniHome — Project Status

> Last Updated: 2026-10-21 by @dev-lead
> Current Phase: Phase 1 — Room Marketplace

---

## Completed ✅

- [x] Plugin scaffold created
- [x] Composer autoloader configured
- [x] Database migration system (including FK pass)
- [x] Module isolation — error boundaries + feature flags
- [x] Custom roles: `student`, `landlord`, `verifier`
- [x] REST API router — health check returns 200
- [x] Shared utilities: Validator, Sanitizer, Logger, Config
- [x] NotificationService scaffold (empty, correct interface)
- [x] EventTracker scaffold (empty, correct interface)
- [x] PHPUnit bootstrap passing
- [x] `wp_unihome_rooms` + `wp_unihome_room_images` tables
- [x] Room Repository — CRUD queries
- [x] Room Service — create, update, get by ID

## In Progress 🚧

- [ ] Room Service — status transitions — @dev-1
- [ ] Room Controller — REST endpoints — @dev-2
- [ ] Room listing page — @dev-3
- [ ] Search/filter interface — @dev-4

## Next Milestone

> Room Marketplace MVP — Target: 2026-11-04
```

---

## 10. Example CLAUDE.md Preview

```markdown
# UniHome — AI Coding Agent Instructions

## Project Identity

UniHome is a Student Housing Marketplace built on WordPress as an application framework.
WordPress provides auth, admin, media, database. All business logic is custom-built
inside `unihome-plugin`.

## Before You Write Any Code

1. Read `STATUS.md` — current phase and progress
2. Read `docs/architecture.md` — layers and dependency graph
3. Read target module docs — module you're working on
4. Read existing code — follow established patterns
5. Check `docs/database.md` — table schemas
6. Run existing tests — confirm they pass

## Hard Constraints

- Stack: WordPress + PHP 8.0+ + MySQL 8.0 + Vanilla JS + Composer PSR-4
- Do NOT introduce React, Vue, NestJS, Laravel, or any external framework
- Do NOT create Custom Post Types for business data
- Do NOT put business logic in JavaScript
- Do NOT write SQL outside Repository classes
- Do NOT throw exceptions from Services — return WP_Error
- Do NOT use admin-ajax.php — REST API only
- Do NOT send notifications directly — use NotificationService::notify()
- Do NOT create circular module dependencies

## Module Pattern

Controller → Service → Repository

- Controller: REST boundary. Sanitize, validate, call Service, return response.
- Service: Business logic. State transitions, rules. Returns WP_Error on failure.
- Repository: Data access. $wpdb only. Returns data.

Cross-cutting services (Notification, EventTracker, Logger, Cache) live in shared/.

## Module Dependencies

Room: —
Verification → Room
Roommate: —
Chat → Roommate
Recommendation → Room, Chat
Payment → Room
Ads → Payment
Marketplace: —

## File Locations

| What | Where |
|---|---|
| Plugin entry | unihome-plugin/unihome.php |
| Modules | includes/modules/{module}/ |
| Database | includes/database/ |
| Shared | includes/shared/ |
| REST routes | api/class-rest-router.php |
| Tests | tests/modules/{module}/ |

## Decision Log

Read `docs/decision-log.md` for ADRs explaining why specific architectural
decisions were made. When in doubt about an approach, check if an ADR covers it.
```

---

## 11. Decision Log

### Purpose

The Decision Log (`docs/decision-log.md`) records every significant architectural decision using the ADR (Architecture Decision Record) format. This helps:

- **Developers** understand WHY something is built a certain way, not just HOW.
- **AI agents** avoid proposing alternatives that were already considered and rejected.
- **Future team members** understand the reasoning behind non-obvious choices.

### ADR Format

```markdown
## ADR-{NNN}: {Title}

**Date:** YYYY-MM-DD  
**Status:** Accepted | Superseded by ADR-{NNN} | Deprecated  
**Decision:** {What was decided}  
**Context:** {Why this decision was needed}  
**Alternatives Considered:**
- {Alternative 1} — rejected because {reason}
- {Alternative 2} — rejected because {reason}  
**Consequences:** {What changes as a result of this decision}
```

### Initial ADRs (to be created in `docs/decision-log.md`)

**ADR-001: Use custom database tables instead of wp_posts/wp_postmeta**

- **Decision:** All UniHome business data (rooms, verifications, profiles, matches, messages, payments) is stored in custom database tables with typed columns and foreign keys.
- **Context:** UniHome requires multi-field filtering (price + location + type + status), data integrity enforcement, and query performance that `wp_postmeta`'s EAV structure cannot provide.
- **Alternatives:** Custom Post Types with `wp_postmeta` — rejected due to query performance degradation at scale and lack of data type enforcement.
- **Consequences:** We lose WordPress admin UI for content management (custom admin pages required). We gain query performance, schema clarity, and data integrity.

**ADR-002: Use WordPress as application foundation instead of custom PHP framework**

- **Decision:** WordPress is used as the application framework, providing authentication, admin panel, media management, plugin lifecycle, and database abstraction.
- **Context:** Building auth, admin, media upload, and session management from scratch would cost 2-3 months with no business value. WordPress provides all of this out of the box.
- **Alternatives:** Laravel — rejected due to team WordPress familiarity and infrastructure requirements. Custom PHP — rejected due to development time cost.
- **Consequences:** We are constrained by WordPress conventions (hook system, plugin lifecycle). We gain rapid development velocity and a mature ecosystem.

**ADR-003: Controller → Service → Repository pattern**

- **Decision:** Every business module uses a 3-layer architecture.
- **Context:** With 4 developers + AI agents working in parallel, clear separation prevents SQL leaking into business logic and business logic leaking into HTTP handlers.
- **Alternatives:** 2-layer (Controller + Service) — rejected because SQL in Services causes merge conflicts when schema changes. Enterprise (DI + DTOs + Interfaces) — rejected as over-engineering for WordPress context.
- **Consequences:** Each module has 3 files minimum. The overhead is acceptable for the consistency and maintainability it provides.

**ADR-004: Communication System before Recommendation Engine**

- **Decision:** Chat (Phase 4) is built before Recommendation (Phase 5).
- **Context:** The recommendation engine's scoring model depends on behavioral signals. The `contact_landlord` and `message_sent` events generated by the Chat module are among the strongest interest signals. Building recommendation without them produces a significantly weaker model.
- **Alternatives:** Recommendation first (original v0.2.0 order) — rejected because recommendation would launch with incomplete data, requiring rework when Chat is added later.
- **Consequences:** The recommendation engine starts with a richer signal set. The interface contract between Chat and Recommendation is defined upfront.

**ADR-005: Notification as cross-cutting service, not business module**

- **Decision:** Notification is implemented as a shared service, not a standalone business module.
- **Context:** Every business module needs to notify users (verification results, new matches, messages, payments). If Notification were a module, every other module would depend on it, creating a dependency hub that violates the acyclic dependency principle.
- **Alternatives:** Notification as a business module — rejected because it would become a universal dependency. Inline email sending per module — rejected because it duplicates delivery logic and ignores user preferences.
- **Consequences:** `NotificationService` lives in `shared/` and is available to all modules. Modules call `NotificationService::notify()` instead of `wp_mail()` directly.

---

## 12. Team Approval Checklist

This proposal requires explicit approval before development begins. Each item must be reviewed by the Technical Lead and at least 2 developers.

| # | Item | Approved | Reviewer | Date |
|---|---|---|---|---|
| 1 | WordPress as application framework — accepted | ☐ | | |
| 2 | Module separation (10 modules: Room, Verification, Roommate, Chat, Recommendation, Payment, Marketplace, Ads + Notification, Analytics as shared services) — accepted | ☐ | | |
| 3 | Module dependency graph (acyclic, direction-enforced) — accepted | ☐ | | |
| 4 | Controller → Service → Repository pattern (with pragmatic simplification guidelines) — accepted | ☐ | | |
| 5 | Composer PSR-4 autoloading — accepted | ☐ | | |
| 6 | Module isolation (error boundaries + feature flags) — accepted | ☐ | | |
| 7 | Custom database tables (not CPT) — accepted | ☐ | | |
| 8 | All tables defined (rooms, images, verifications, profiles, matches, conversations, messages, notifications, events, packages, payments, audit log) — accepted | ☐ | | |
| 9 | Database migration system (including `dbDelta()` FK workaround) — accepted | ☐ | | |
| 10 | Soft-delete, audit trail, and deletion policies — accepted | ☐ | | |
| 11 | REST API as primary pattern — accepted | ☐ | | |
| 12 | Development rules and forbidden practices — accepted | ☐ | | |
| 13 | Testing strategy (PHPUnit + WP_UnitTestCase) — accepted | ☐ | | |
| 14 | Environment requirements (WP 6.4+, PHP 8.0+, MySQL 8.0+) — accepted | ☐ | | |
| 15 | Notification Service architecture — accepted | ☐ | | |
| 16 | Analytics / Event Tracking architecture — accepted | ☐ | | |
| 17 | Decision Log (ADR format) — accepted | ☐ | | |
| 18 | Phase ordering (0→1→2→3→4:Chat→5:Recommend→6:Payment) — accepted | ☐ | | |
| 19 | Payment deferral to Phase 6 (business decision) with MVP-lite option documented — accepted | ☐ | | |
| 20 | Git workflow — accepted | ☐ | | |
| 21 | Documentation-driven development — accepted | ☐ | | |
| 22 | AI coding agent integration via CLAUDE.md — accepted | ☐ | | |
| 23 | STATUS.md as single source of truth — accepted | ☐ | | |

### How to Approve

1. Each reviewer reads this proposal in full.
2. Comments and objections raised in review meeting or PR comments.
3. Disputed items discussed and resolved.
4. Once all checked, Technical Lead merges and Phase 0 begins.

### After Approval

1. Technical Lead creates repository with approved structure.
2. Technical Lead writes `CLAUDE.md`, `STATUS.md`, `docs/decision-log.md`, and initial `docs/` files.
3. Phase 0a begins (TL-only, 1 week).
4. Phase 0b assigned to full team (2 weeks).
5. Development begins.

---

> **This document is a proposal. It is not final documentation.**  
> **The purpose is to align the team on architecture and workflow before writing any code.**  
> **Once approved, each section will be expanded into its own dedicated document.**

---

### Revision History

| Version | Date | Changes |
|---|---|---|
| 0.1.0-draft | 2026-09-30 | Initial proposal |
| 0.2.0-revised | 2026-09-30 | Architecture review: added conversations + packages tables, audit log, autoloading, module isolation, testing, error handling, environment config, dependency graph, REST API clarification, `dbDelta()` FK workaround, soft-delete, `DECIMAL(15,2)`, integration checkpoints, Phase 0 extended, chat decoupled from Phase 3, 8 missing docs added, checklist expanded to 20 items. |
| 0.3.0 | 2026-09-30 | Phase reorder (Chat before Recommendation), Notification Service as cross-cutting shared service, Analytics/Event Tracking with `wp_unihome_events` and `wp_unihome_notifications` tables, Decision Log with ADR format and 5 initial ADRs, Payment strategy review with MVP-lite option, Controller→Service→Repository pattern reviewed and affirmed with pragmatic guidelines, module dependency graph updated (Recommendation now depends on Chat), phase history files updated for new ordering, approval checklist expanded to 23 items. |
