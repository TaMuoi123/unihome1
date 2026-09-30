# UniHome — Module Documentation

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §3, §5  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Module Classification

UniHome has two types of components:

| Type | Examples | Pattern | Location |
|---|---|---|---|
| **Business Modules** | Room, Verification, Roommate, Chat, Recommendation, Payment, Marketplace, Ads | Controller → Service → Repository | `includes/modules/{name}/` |
| **Cross-Cutting Services** | Notification, EventTracker, Logger, Cache, Config, Validator, Sanitizer | Service (+ Repository if needed) | `includes/shared/` |

---

## 2. Business Modules

### 2.1 Room

| Property | Value |
|---|---|
| **Directory** | `includes/modules/rooms/` |
| **Depends On** | — (no UniHome module dependencies) |
| **Depended On By** | Verification, Recommendation, Payment |
| **Phase** | Phase 1 |
| **Owner** | Dev 1 (service), Dev 2 (repository/controller) |

**Responsibilities:**
- Room CRUD operations
- Listing lifecycle: `draft → pending_review → active → rented → archived`
- Search and filter (price, location, type, amenities)
- Image management (via WordPress media library)
- Pricing logic

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-room-controller.php` | Controller | REST endpoints: create, read, update, list, search |
| `class-room-service.php` | Service | Business logic: status transitions, validation, audit logging |
| `class-room-repository.php` | Repository | SQL queries: CRUD, search filters, geolocation queries |

**Tables:** `wp_unihome_rooms`, `wp_unihome_room_images`

**Public Service Interface (callable by dependent modules):**
```php
RoomService::get_room_by_id( int $id ): array|WP_Error
RoomService::mark_verified( int $room_id ): bool|WP_Error
RoomService::get_rooms_by_owner( int $owner_id ): array
```

---

### 2.2 Verification

| Property | Value |
|---|---|
| **Directory** | `includes/modules/verification/` |
| **Depends On** | Room |
| **Depended On By** | — |
| **Phase** | Phase 2 |
| **Owner** | Dev 2 (service/repository), Dev 1 (controller) |

**Responsibilities:**
- Verification document upload
- Workflow state machine: `draft → pending_review → in_review → approved/rejected`
- Verifier assignment and queue management
- Trust scoring

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-verification-controller.php` | Controller | REST endpoints: submit, review, approve, reject |
| `class-verification-service.php` | Service | State machine, document handling, calls RoomService::mark_verified() |
| `class-verification-repository.php` | Repository | SQL queries for verification records |

**Tables:** `wp_unihome_verifications`

**Notifications fired:** `room_approved`, `room_rejected`, `verification_assigned`

---

### 2.3 Roommate

| Property | Value |
|---|---|
| **Directory** | `includes/modules/roommate/` |
| **Depends On** | — (no UniHome module dependencies) |
| **Depended On By** | Chat |
| **Phase** | Phase 3 |
| **Owner** | Dev 3 (service/repository), Dev 1 (matching engine) |

**Responsibilities:**
- Profile questionnaire management
- Compatibility scoring algorithm
- Match generation and suggestion
- Match accept/decline

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-roommate-controller.php` | Controller | REST endpoints: profile CRUD, get matches, accept/decline |
| `class-roommate-service.php` | Service | Profile management, match status transitions |
| `class-matching-engine.php` | Service | Compatibility scoring algorithm |
| `class-roommate-repository.php` | Repository | SQL queries for profiles and matches |

**Tables:** `wp_unihome_roommate_profiles`, `wp_unihome_matches`

**Notifications fired:** `new_match_found`

**Matching Algorithm (v1):**
```
score = 0
sleep_schedule matches:   +20
noise_tolerance within 1: +15
cleanliness within 1:     +15
guest_policy matches:     +15
smoking matches:          +10
pet_friendly matches:     +10
budget overlaps:          +15
Total possible: 100
```

---

### 2.4 Chat

| Property | Value |
|---|---|
| **Directory** | `includes/modules/chat/` |
| **Depends On** | Roommate |
| **Depended On By** | Recommendation |
| **Phase** | Phase 4 |
| **Owner** | Dev 4 (service/repository), Dev 1 (controller) |

**Responsibilities:**
- Messaging between users (student↔landlord, student↔student)
- Conversation creation and threading
- Message read status tracking
- "Contact Landlord" flow from room detail

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-chat-controller.php` | Controller | REST endpoints: send message, get conversations, get messages |
| `class-message-service.php` | Service | Message sending, conversation management, read status |
| `class-message-repository.php` | Repository | SQL queries for conversations and messages |

**Tables:** `wp_unihome_conversations`, `wp_unihome_messages`

**Notifications fired:** `new_message`

**Events tracked:** `contact_landlord`, `message_sent`

---

### 2.5 Recommendation

| Property | Value |
|---|---|
| **Directory** | `includes/modules/recommendation/` |
| **Depends On** | Room, Chat |
| **Depended On By** | — |
| **Phase** | Phase 5 |
| **Owner** | Dev 1 (engine), Dev 2 (controller/service) |

**Responsibilities:**
- Rule-based room scoring model
- Behavioral signal consumption (from EventTracker)
- Personalized room feed generation
- Future: Gemini API integration for ML-based scoring

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-recommendation-controller.php` | Controller | REST endpoint: personalized feed |
| `class-recommendation-service.php` | Service | Feed generation, score aggregation |
| `class-recommendation-engine.php` | Service | Scoring model (rule-based, later ML) |

**Consumes:** `wp_unihome_events` (via EventTracker), `wp_unihome_rooms` (via RoomService)

**Scoring signals:** Price match (20%), Location (20%), Room type (10%), Recency (10%), Verification (10%), Landlord rating (10%), Search similarity (10%), Contact signal (10%)

---

### 2.6 Payment

| Property | Value |
|---|---|
| **Directory** | `includes/modules/payment/` |
| **Depends On** | Room |
| **Depended On By** | Ads |
| **Phase** | Phase 6 |
| **Owner** | TBD |

**Responsibilities:**
- Package definitions and management
- Payment gateway abstraction
- Subscription and purchase flow
- Invoice generation
- Payment status tracking

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-payment-controller.php` | Controller | REST endpoints: purchase, status, history |
| `class-package-service.php` | Service | Package management, subscription logic |
| `class-payment-gateway.php` | Service | Gateway abstraction (Stripe, local) |
| `class-payment-repository.php` | Repository | SQL queries for packages and payments |

**Tables:** `wp_unihome_packages`, `wp_unihome_payments`

**Notifications fired:** `payment_completed`

---

### 2.7 Marketplace

| Property | Value |
|---|---|
| **Directory** | `includes/modules/marketplace/` |
| **Depends On** | — (independent) |
| **Depended On By** | — |
| **Phase** | Phase 7 |
| **Owner** | TBD |

**Responsibilities:**
- Second-hand item listings
- Category management
- Item lifecycle (active → sold → archived)
- Student-to-student transactions

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-marketplace-controller.php` | Controller | REST endpoints: list, search, CRUD |
| `class-item-service.php` | Service | Item management, status transitions |
| `class-item-repository.php` | Repository | SQL queries for items |

**Tables:** TBD (defined in Phase 7)

---

### 2.8 Ads

| Property | Value |
|---|---|
| **Directory** | `includes/modules/ads/` |
| **Depends On** | Payment |
| **Depended On By** | — |
| **Phase** | Phase 8 |
| **Owner** | TBD |

**Responsibilities:**
- Ad placement zone management
- Impression and click tracking
- Ad package management
- Ad scheduling and rotation

**Files:**
| File | Layer | Purpose |
|---|---|---|
| `class-ad-controller.php` | Controller | REST endpoints: serve ad, track impression |
| `class-ad-service.php` | Service | Ad selection, scheduling, tracking |
| `class-ad-repository.php` | Repository | SQL queries for ads and tracking |

**Tables:** TBD (defined in Phase 8)

---

## 3. Cross-Cutting Shared Services

These live in `includes/shared/` and are available to all modules.

| Service | File | Purpose |
|---|---|---|
| **NotificationService** | `class-notification-service.php` | Event-driven notification dispatch (email, in-app) |
| **NotificationRepository** | `class-notification-repository.php` | Data access for `wp_unihome_notifications` |
| **EventTracker** | `class-event-tracker.php` | Behavioral event recording to `wp_unihome_events` |
| **Logger** | `class-logger.php` | Structured logging with `[UniHome:{module}]` prefix + audit trail |
| **CacheManager** | `class-cache-manager.php` | Transient wrapper with key naming conventions |
| **Config** | `class-config.php` | Environment config from `wp-config.php` constants |
| **Validator** | `class-validator.php` | Input validation utilities |
| **Sanitizer** | `class-sanitizer.php` | Input sanitization wrappers |
| **ResponseFormatter** | `class-response-formatter.php` | Consistent API response structure |
| **PermissionChecker** | `class-permission-checker.php` | Capability check helpers |

---

## 4. Module Communication Rules

1. **Only call dependent module's Service.** If Module A depends on Module B, A may call `B_Service::method()`. Never call B's Repository or Controller directly.
2. **No reverse dependencies.** If A → B, then B must never call A.
3. **No circular dependencies.** If A → B → C, then C must never depend on A.
4. **Cross-cutting services are exceptions.** Logger, NotificationService, EventTracker, etc. may be called by any module.
5. **New cross-module dependencies require TL approval** and must be added to the dependency graph in `docs/architecture.md`.
