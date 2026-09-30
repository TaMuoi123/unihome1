# UniHome — System Architecture

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §3  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Environment Requirements

| Requirement | Minimum Version | Rationale |
|---|---|---|
| **WordPress** | 6.4+ | Stable REST API, modern hook system |
| **PHP** | 8.0+ | Union types, named arguments, namespace maturity |
| **MySQL** | 8.0+ | Native JSON, window functions, CTEs |
| **Composer** | 2.x | PSR-4 autoloading |
| **Node.js** | Not required | Vanilla JS, no build step |

WordPress Multisite is **out of scope**. `$wpdb->prefix` is used throughout for future compatibility.

---

## 2. System Layer Diagram

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
│  wp_users · wp_options · wp_posts (limited)     │
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

---

## 3. Layer Responsibilities

### User Browser Layer

- Renders HTML from WordPress theme templates.
- Sends REST API requests for dynamic operations.
- **No business logic.** The browser is a presentation terminal.

### WordPress Theme Layer

- Controls visual presentation and page routing.
- Loads CSS/JS via `wp_enqueue_scripts`.
- Uses template hierarchy (`page-*.php`, `single-*.php`).
- Calls plugin functions via template tags or shortcodes.
- **Does not** contain business logic. If a template has an `if` that decides business outcomes, the logic is in the wrong place.

### UniHome Custom Plugin Layer

- Contains **all** UniHome business logic.
- Organized into discrete **business modules**: Room, Verification, Roommate, Chat, Recommendation, Payment, Marketplace, Ads.
- **Cross-cutting shared services**: Notification, EventTracker, Logger, Cache, Config, Validator, Sanitizer.
- Each module is isolated via error boundaries — a fatal error in one module does not crash others.
- Modules communicate through Service interfaces, respecting the dependency graph.

### WordPress Core Services

We **consume** these. We do **not** modify WordPress core.

| Service | Usage |
|---|---|
| `WP_User` | Authentication, sessions, role/capability |
| `$wpdb` | Database access via prepared statements |
| `WP_REST_Server` | REST API routing |
| `Transients` | Caching (search results, recommendation scores) |
| `wp_options` | Plugin config, feature flags |
| `Media` | File uploads (room photos, documents) |
| `WP-Cron` | Scheduled tasks (with system cron in production) |
| `wp_mail` | Email delivery (via NotificationService) |

### MySQL Database

- WordPress default tables for users, options, limited post data.
- UniHome custom tables (`wp_unihome_*`) for all business data.
- All state changes logged in `wp_unihome_audit_log`.
- See `docs/database.md` for complete schema reference.

### External Services

| Service | Purpose | Phase |
|---|---|---|
| Payment Gateway (Stripe / local) | Financial transactions | Phase 6 |
| Google Maps API | Geocoding, map rendering | Phase 1 |
| Gemini API | Recommendation engine, AI matching | Phase 5 |
| Cloud Storage (S3/GCS) | Media offloading | Future |
| Email Service (SMTP/SES) | Transactional email | Production |
| Push Notifications | Mobile push | Future |

---

## 4. Module Architecture Pattern

### Three-Layer Pattern

```
Controller (HTTP boundary — REST API)
    │
    ▼
Service (Business logic — returns WP_Error on failure)
    │
    ▼
Repository (Data access — $wpdb only)
```

See `docs/modules.md` for detailed per-module documentation.  
See `decision-log.md` ADR-003 for the rationale behind this pattern choice.

### When to Use Which Pattern

| Situation | Pattern |
|---|---|
| Module with REST endpoints + logic + DB | Controller → Service → Repository |
| Shared service (Notification, EventTracker) | Service + Repository (no Controller) |
| Utility (Validator, Sanitizer) | Static class or functions |
| New module with 1-2 endpoints | Full 3-layer (consistency > saving 1 file) |

---

## 5. Module Dependency Graph

**Circular dependencies are forbidden.** Arrows point toward the dependency.

```
                  ┌──────────────┐
                  │     Room     │
                  └──────┬───────┘
                         │
          ┌──────────────┼──────────────────┐
          │              │                  │
          ▼              ▼                  ▼
   ┌──────────────┐ ┌──────────────┐ ┌───────────────┐
   │ Verification │ │Recommendation│ │    Payment     │
   └──────────────┘ └──────┬───────┘ └───────┬───────┘
                           │                 │
                           ▼                 ▼
                    ┌──────────────┐   ┌──────────────┐
                    │     Chat     │   │     Ads      │
                    └──────────────┘   └──────────────┘

   ┌──────────────┐
   │   Roommate   │
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │     Chat     │
   └──────────────┘

   ┌──────────────┐
   │  Marketplace │  (independent)
   └──────────────┘
```

### Dependency Table

| Module | Depends On | Depended On By |
|---|---|---|
| Room | — | Verification, Recommendation, Payment |
| Verification | Room | — |
| Roommate | — | Chat |
| Chat | Roommate | Recommendation |
| Recommendation | Room, Chat | — |
| Payment | Room | Ads |
| Ads | Payment | — |
| Marketplace | — | — |

### Rules

- A module may **only** call the Service class of modules it depends on.
- If A depends on B, B must **never** depend on A.
- Cross-cutting services (`shared/`) are available to all modules and are not shown in this graph.

---

## 6. Cross-Cutting Services

### Notification Service

Event-driven notification dispatch. Not a business module — a shared service.

```
Business Module fires event
    │
    ▼
NotificationService::notify( $event_type, $user_id, $context )
    │
    ├── Check user preferences (wp_usermeta)
    ├── Create notification record (wp_unihome_notifications)
    │
    ├──► Email (wp_mail / SMTP)
    ├──► In-App (database record, polled by frontend)
    └──► Push (future)
```

**Supported events:** `room_approved`, `room_rejected`, `new_match_found`, `new_message`, `payment_completed`, `listing_expired`, `verification_assigned`.

### Event Tracker (Analytics)

Behavioral event recording for recommendation and analytics.

```
Controller (after successful action)
    │
    ▼
EventTracker::track( $event_name, $user_id, $data )
    │
    ▼
wp_unihome_events (append-only table)
    │
    ▼
Consumed by: Recommendation Engine, Admin Dashboard (future)
```

**Tracked events:** `room_viewed`, `room_saved`, `room_search`, `contact_landlord`, `match_viewed`, `match_accepted`, `message_sent`.

**Key rule:** EventTracker is called in the **Controller** (after success), NOT in the Service. This keeps business logic pure.

---

## 7. Data Flow Examples

### Room Creation Flow

```
Browser: POST /wp-json/unihome/v1/rooms
    │
    ▼
RoomController::create_room( WP_REST_Request )
    ├── permission_callback: current_user_can('unihome_create_room')
    ├── Sanitize input (sanitize_text_field, floatval)
    ├── Validate (Validator::validate_room_data)
    │
    ▼
RoomService::create_room( $data )
    ├── Business rules (status = 'draft', validate owner)
    ├── RoomRepository::insert( $data )
    ├── Logger::audit( 'room', $id, 'created', null, $data )
    │
    ▼
RoomController returns WP_REST_Response( ['room' => $result], 201 )
    ├── EventTracker::track( 'room_created', $user_id, ['room_id' => $id] )
```

### Verification Approval Flow

```
Admin: PUT /wp-json/unihome/v1/verifications/{id}/approve
    │
    ▼
VerificationController::approve( WP_REST_Request )
    ├── permission_callback: current_user_can('unihome_verify_rooms')
    │
    ▼
VerificationService::approve( $verification_id, $verifier_id )
    ├── VerificationRepository::get_by_id( $id )
    ├── Validate state transition (in_review → approved)
    ├── VerificationRepository::update_status( $id, 'approved' )
    ├── RoomService::mark_verified( $room_id )
    ├── Logger::audit( 'verification', $id, 'status_changed', 'in_review', 'approved' )
    ├── NotificationService::notify( 'room_approved', $landlord_id, ['room_id' => $room_id] )
```

---

## 8. Plugin Directory Structure

See `docs/modules.md` for the full directory tree with per-file descriptions.

```
wp-content/plugins/unihome-plugin/
├── unihome.php                     # Entry point
├── composer.json                   # PSR-4 autoload
├── vendor/                         # Autoloader (committed)
├── phpunit.xml                     # Test config
├── includes/
│   ├── class-unihome-loader.php
│   ├── class-unihome-activator.php
│   ├── class-unihome-deactivator.php
│   ├── database/                   # Schema, migrations, FK, seeder
│   ├── modules/                    # Business modules (rooms, verification, etc.)
│   └── shared/                     # Cross-cutting services
├── api/                            # REST route registration
├── admin/                          # WP admin pages
├── assets/                         # JS/CSS
├── templates/                      # Reusable template parts
└── tests/                          # PHPUnit tests
```
