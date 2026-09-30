# UniHome — Database Architecture

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §6  
> **Maintained By:** Technical Lead + assigned developer  
> **Last Updated:** 2026-09-30

---

## 1. Design Principle

UniHome uses **custom database tables** for all business data. WordPress default tables are used only for what WordPress already manages (users, options, media references).

See `docs/decision-log.md` ADR-001 for the full rationale (query performance, data integrity, schema clarity, indexing, migration).

---

## 2. WordPress Default Tables (Used As-Is)

| Table | UniHome Usage |
|---|---|
| `wp_users` | All user accounts: students, landlords, verifiers, admins |
| `wp_usermeta` | User preferences, notification preferences (`unihome_notification_prefs`), onboarding state |
| `wp_options` | Plugin configuration, feature flags, migration version tracking, API keys (encrypted) |
| `wp_posts` | **Limited.** Static content pages only (About, Terms, Privacy). NOT used for business entities. |

---

## 3. Deletion Policy

| Entity | Strategy | Rationale |
|---|---|---|
| Rooms | Soft delete via `status='archived'` | Verification history, message context, audit trail |
| Users | Never deleted | WordPress convention. Disable via role removal. |
| Verifications | Never deleted | Legal and trust audit |
| Messages | Never deleted | Legal retention, dispute resolution |
| Matches | Retained | Status set to `'declined'`/`'expired'` |
| Payments | Never deleted | Financial audit trail |
| Notifications | Retained | User marks read, never deleted from DB |
| Events | Never deleted | Append-only analytics. Purge policy post-launch. |
| Marketplace Items | Soft delete via `deleted_at` | Transaction history preservation |
| Ads | Soft delete via `deleted_at` | Impression history for billing |

---

## 4. Custom Table Schemas

### 4.1 `wp_unihome_rooms`

Stores structured room listing data.

```sql
CREATE TABLE {$prefix}unihome_rooms (
    id              BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    owner_id        BIGINT UNSIGNED NOT NULL,
    title           VARCHAR(255) NOT NULL,
    description     TEXT,
    address         VARCHAR(500),
    latitude        DECIMAL(10,8),
    longitude       DECIMAL(11,8),
    price           DECIMAL(15,2) NOT NULL,
    currency        CHAR(3) DEFAULT 'VND',
    room_type       ENUM('single','shared','studio','apartment') NOT NULL,
    status          ENUM('draft','pending_review','active','rented','archived') DEFAULT 'draft',
    amenities       JSON,
    max_occupants   TINYINT UNSIGNED,
    available_from  DATE,
    created_at      DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at      DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_owner (owner_id),
    INDEX idx_status (status),
    INDEX idx_location (latitude, longitude),
    INDEX idx_price (price),
    INDEX idx_type (room_type)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

| Column | Type | Description |
|---|---|---|
| `id` | `BIGINT UNSIGNED AUTO_INCREMENT` | Primary key |
| `owner_id` | `BIGINT UNSIGNED` | FK → `wp_users.ID` |
| `title` | `VARCHAR(255)` | Listing title |
| `description` | `TEXT` | Full description |
| `address` | `VARCHAR(500)` | Full address string |
| `latitude` | `DECIMAL(10,8)` | Geocoded latitude |
| `longitude` | `DECIMAL(11,8)` | Geocoded longitude |
| `price` | `DECIMAL(15,2)` | Monthly rent (VND stored as whole number with .00) |
| `currency` | `CHAR(3)` | ISO 4217 currency code |
| `room_type` | `ENUM(...)` | Room classification |
| `status` | `ENUM(...)` | Listing lifecycle state |
| `amenities` | `JSON` | Structured amenities data |
| `max_occupants` | `TINYINT UNSIGNED` | Maximum occupants |
| `available_from` | `DATE` | Availability start |
| `created_at` | `DATETIME` | Record creation |
| `updated_at` | `DATETIME` | Last modification |

---

### 4.2 `wp_unihome_room_images`

```sql
CREATE TABLE {$prefix}unihome_room_images (
    id              BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    room_id         BIGINT UNSIGNED NOT NULL,
    attachment_id   BIGINT UNSIGNED NOT NULL,
    sort_order      TINYINT UNSIGNED DEFAULT 0,
    is_primary      TINYINT(1) DEFAULT 0,

    INDEX idx_room (room_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

| Column | Type | Description |
|---|---|---|
| `room_id` | `BIGINT UNSIGNED` | FK → `wp_unihome_rooms.id` |
| `attachment_id` | `BIGINT UNSIGNED` | FK → `wp_posts.ID` (WordPress media) |
| `sort_order` | `TINYINT UNSIGNED` | Display order |
| `is_primary` | `TINYINT(1)` | Primary listing image flag |

---

### 4.3 `wp_unihome_verifications`

```sql
CREATE TABLE {$prefix}unihome_verifications (
    id               BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    room_id          BIGINT UNSIGNED NOT NULL,
    verifier_id      BIGINT UNSIGNED NULL,
    status           ENUM('draft','pending_review','in_review','approved','rejected') DEFAULT 'draft',
    document_urls    JSON,
    rejection_reason TEXT NULL,
    verified_at      DATETIME NULL,
    created_at       DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at       DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_room (room_id),
    INDEX idx_verifier (verifier_id),
    INDEX idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

| Column | Type | Description |
|---|---|---|
| `room_id` | `BIGINT UNSIGNED` | FK → `wp_unihome_rooms.id` |
| `verifier_id` | `BIGINT UNSIGNED NULL` | FK → `wp_users.ID` |
| `status` | `ENUM(...)` | Verification workflow state |
| `document_urls` | `JSON` | Uploaded document references |
| `rejection_reason` | `TEXT NULL` | Reason if rejected |
| `verified_at` | `DATETIME NULL` | Approval timestamp |

---

### 4.4 `wp_unihome_roommate_profiles`

```sql
CREATE TABLE {$prefix}unihome_roommate_profiles (
    id                  BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id             BIGINT UNSIGNED NOT NULL,
    sleep_schedule      ENUM('early_bird','night_owl','flexible'),
    noise_tolerance     TINYINT UNSIGNED,
    cleanliness         TINYINT UNSIGNED,
    guest_policy        ENUM('no_guests','occasional','frequent'),
    smoking             TINYINT(1) DEFAULT 0,
    pet_friendly        TINYINT(1) DEFAULT 0,
    budget_min          DECIMAL(15,2),
    budget_max          DECIMAL(15,2),
    preferred_location  VARCHAR(255),
    bio                 TEXT,
    created_at          DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at          DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    UNIQUE INDEX idx_user (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 4.5 `wp_unihome_matches`

```sql
CREATE TABLE {$prefix}unihome_matches (
    id                   BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_a_id            BIGINT UNSIGNED NOT NULL,
    user_b_id            BIGINT UNSIGNED NOT NULL,
    compatibility_score  DECIMAL(5,2),
    status               ENUM('suggested','viewed','accepted','declined') DEFAULT 'suggested',
    computed_at          DATETIME DEFAULT CURRENT_TIMESTAMP,

    UNIQUE INDEX idx_pair (user_a_id, user_b_id),
    INDEX idx_score (compatibility_score)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Convention:** `user_a_id < user_b_id` always. Canonicalizes pair ordering, prevents duplicates.

---

### 4.6 `wp_unihome_conversations`

```sql
CREATE TABLE {$prefix}unihome_conversations (
    id                BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    room_id           BIGINT UNSIGNED NULL,
    participant_a_id  BIGINT UNSIGNED NOT NULL,
    participant_b_id  BIGINT UNSIGNED NOT NULL,
    last_message_at   DATETIME NULL,
    status            ENUM('active','archived') DEFAULT 'active',
    created_at        DATETIME DEFAULT CURRENT_TIMESTAMP,

    UNIQUE INDEX idx_participants (participant_a_id, participant_b_id),
    INDEX idx_participant_a (participant_a_id),
    INDEX idx_participant_b (participant_b_id),
    INDEX idx_last_message (last_message_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 4.7 `wp_unihome_messages`

```sql
CREATE TABLE {$prefix}unihome_messages (
    id               BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    conversation_id  BIGINT UNSIGNED NOT NULL,
    sender_id        BIGINT UNSIGNED NOT NULL,
    message          TEXT NOT NULL,
    is_read          TINYINT(1) DEFAULT 0,
    created_at       DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_conversation_time (conversation_id, created_at),
    INDEX idx_sender (sender_id),
    INDEX idx_unread (conversation_id, is_read)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Note:** `recipient_id` is not stored. Recipient is derived from the conversation's participant list.

---

### 4.8 `wp_unihome_notifications`

```sql
CREATE TABLE {$prefix}unihome_notifications (
    id          BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id     BIGINT UNSIGNED NOT NULL,
    event_type  VARCHAR(50) NOT NULL,
    title       VARCHAR(255) NOT NULL,
    body        TEXT,
    context     JSON NULL,
    is_read     TINYINT(1) DEFAULT 0,
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_user_unread (user_id, is_read, created_at),
    INDEX idx_event (event_type)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 4.9 `wp_unihome_events`

```sql
CREATE TABLE {$prefix}unihome_events (
    id          BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id     BIGINT UNSIGNED NOT NULL,
    event_name  VARCHAR(50) NOT NULL,
    event_data  JSON NULL,
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_user_event (user_id, event_name),
    INDEX idx_event_time (event_name, created_at),
    INDEX idx_time (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Rules:** Append-only. Never update or delete rows.

---

### 4.10 `wp_unihome_packages`

```sql
CREATE TABLE {$prefix}unihome_packages (
    id             BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name           VARCHAR(100) NOT NULL,
    slug           VARCHAR(100) NOT NULL,
    type           ENUM('room_boost','featured','premium_landlord','ad_package') NOT NULL,
    price          DECIMAL(15,2) NOT NULL,
    currency       CHAR(3) DEFAULT 'VND',
    duration_days  INT UNSIGNED NOT NULL,
    features       JSON,
    is_active      TINYINT(1) DEFAULT 1,
    created_at     DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at     DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    UNIQUE INDEX idx_slug (slug),
    INDEX idx_type (type),
    INDEX idx_active (is_active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 4.11 `wp_unihome_payments`

```sql
CREATE TABLE {$prefix}unihome_payments (
    id                      BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id                 BIGINT UNSIGNED NOT NULL,
    package_id              BIGINT UNSIGNED NOT NULL,
    amount                  DECIMAL(15,2) NOT NULL,
    currency                CHAR(3) NOT NULL,
    gateway                 VARCHAR(50),
    gateway_transaction_id  VARCHAR(255),
    status                  ENUM('pending','completed','failed','refunded') DEFAULT 'pending',
    created_at              DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_user (user_id),
    INDEX idx_package (package_id),
    INDEX idx_status (status),
    INDEX idx_gateway_tx (gateway_transaction_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 4.12 `wp_unihome_audit_log`

```sql
CREATE TABLE {$prefix}unihome_audit_log (
    id            BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    entity_type   VARCHAR(50) NOT NULL,
    entity_id     BIGINT UNSIGNED NOT NULL,
    action        VARCHAR(50) NOT NULL,
    old_value     JSON NULL,
    new_value     JSON NULL,
    performed_by  BIGINT UNSIGNED NOT NULL,
    ip_address    VARCHAR(45),
    created_at    DATETIME DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_entity (entity_type, entity_id),
    INDEX idx_performer (performed_by),
    INDEX idx_time (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Rules:**
- Rows are **never updated or deleted**.
- Every state change (verification, payment, room status) **must** log an audit entry.
- Logging is centralized in `shared/class-logger.php`, not duplicated per module.

---

## 5. Foreign Key Constraints

`dbDelta()` does **not** support FK constraints (silently drops them). FKs are applied in a **separate migration step** using raw SQL in `includes/database/foreign-keys.php`.

```php
// Example: foreign-keys.php
$wpdb->query("
    ALTER TABLE {$prefix}unihome_rooms
    ADD CONSTRAINT fk_rooms_owner
    FOREIGN KEY (owner_id) REFERENCES {$prefix}users(ID)
    ON DELETE RESTRICT
");

$wpdb->query("
    ALTER TABLE {$prefix}unihome_room_images
    ADD CONSTRAINT fk_images_room
    FOREIGN KEY (room_id) REFERENCES {$prefix}unihome_rooms(id)
    ON DELETE CASCADE
");
```

### FK Reference Map

| Table | Column | References | On Delete |
|---|---|---|---|
| `rooms` | `owner_id` | `wp_users.ID` | RESTRICT |
| `room_images` | `room_id` | `rooms.id` | CASCADE |
| `room_images` | `attachment_id` | `wp_posts.ID` | RESTRICT |
| `verifications` | `room_id` | `rooms.id` | RESTRICT |
| `verifications` | `verifier_id` | `wp_users.ID` | SET NULL |
| `roommate_profiles` | `user_id` | `wp_users.ID` | CASCADE |
| `matches` | `user_a_id` | `wp_users.ID` | CASCADE |
| `matches` | `user_b_id` | `wp_users.ID` | CASCADE |
| `conversations` | `room_id` | `rooms.id` | SET NULL |
| `conversations` | `participant_a_id` | `wp_users.ID` | RESTRICT |
| `conversations` | `participant_b_id` | `wp_users.ID` | RESTRICT |
| `messages` | `conversation_id` | `conversations.id` | CASCADE |
| `messages` | `sender_id` | `wp_users.ID` | RESTRICT |
| `notifications` | `user_id` | `wp_users.ID` | CASCADE |
| `events` | `user_id` | `wp_users.ID` | CASCADE |
| `packages` | — | — | — |
| `payments` | `user_id` | `wp_users.ID` | RESTRICT |
| `payments` | `package_id` | `packages.id` | RESTRICT |
| `audit_log` | `performed_by` | `wp_users.ID` | RESTRICT |

---

## 6. Migration Rules

1. **Versioned:** `001_create_rooms`, `002_create_room_images`, etc. Tracked in `wp_options`.
2. **Forward-only:** Never modify a committed migration. Create a new one.
3. **Activation hook:** `class-unihome-activator.php` runs pending migrations.
4. **FK constraints:** Applied in separate `foreign-keys.php` pass after `dbDelta()`.
5. **No manual SQL:** All schema changes through the migration system.
6. **Table prefix:** `{$wpdb->prefix}unihome_` — never hardcode `wp_`.
7. **Engine:** All tables use InnoDB (MySQL 8.0 default).

---

## 7. Table Summary

| # | Table | Module | Created In |
|---|---|---|---|
| 1 | `wp_unihome_rooms` | Room | Phase 1 |
| 2 | `wp_unihome_room_images` | Room | Phase 1 |
| 3 | `wp_unihome_verifications` | Verification | Phase 2 |
| 4 | `wp_unihome_roommate_profiles` | Roommate | Phase 3 |
| 5 | `wp_unihome_matches` | Roommate | Phase 3 |
| 6 | `wp_unihome_conversations` | Chat | Phase 4 |
| 7 | `wp_unihome_messages` | Chat | Phase 4 |
| 8 | `wp_unihome_notifications` | Shared (Notification) | Phase 0 |
| 9 | `wp_unihome_events` | Shared (Analytics) | Phase 0 |
| 10 | `wp_unihome_packages` | Payment | Phase 6 |
| 11 | `wp_unihome_payments` | Payment | Phase 6 |
| 12 | `wp_unihome_audit_log` | Shared (Logger) | Phase 0 |
