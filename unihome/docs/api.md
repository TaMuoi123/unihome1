# UniHome — REST API Reference

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §7  
> **Maintained By:** Module owners  
> **Last Updated:** 2026-09-30

---

## 1. API Conventions

| Convention | Value |
|---|---|
| **Base URL** | `/wp-json/unihome/v1/` |
| **Auth** | WordPress cookie auth with REST nonce, or Application Passwords |
| **Permission** | Every route has a `permission_callback` |
| **Input** | `WP_REST_Request::get_param()` — never `$_POST` / `$_GET` |
| **Output** | `WP_REST_Response` with consistent JSON structure |
| **Errors** | `{ "message": "...", "code": "..." }` with appropriate HTTP status |

### Response Format

**Success:**
```json
{
    "data": { ... },
    "meta": {
        "page": 1,
        "per_page": 20,
        "total": 150
    }
}
```

**Error:**
```json
{
    "message": "Validation failed: price must be positive",
    "code": "validation_error"
}
```

### HTTP Status Codes

| Code | Usage |
|---|---|
| `200` | Successful GET, PUT, PATCH |
| `201` | Successful POST (resource created) |
| `204` | Successful DELETE |
| `400` | Validation error, bad input |
| `401` | Not authenticated |
| `403` | Authenticated but not authorized |
| `404` | Resource not found |
| `500` | Server error |

---

## 2. Health Check

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/health` | GET | None | Plugin health check |

**Response:**
```json
{
    "status": "ok",
    "version": "1.0.0",
    "phase": "Phase 1"
}
```

---

## 3. Room Endpoints (Phase 1)

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/rooms` | GET | Optional | — | List rooms with filters |
| `/rooms` | POST | Required | `unihome_create_room` | Create a room listing |
| `/rooms/{id}` | GET | Optional | — | Get room details |
| `/rooms/{id}` | PUT | Required | `unihome_edit_room` | Update room |
| `/rooms/{id}/status` | PATCH | Required | `unihome_manage_rooms` | Change room status |
| `/rooms/{id}/images` | POST | Required | `unihome_create_room` | Upload room image |
| `/rooms/search` | GET | Optional | — | Search rooms with filters |

### Query Parameters (GET /rooms, GET /rooms/search)

| Param | Type | Description |
|---|---|---|
| `page` | int | Page number (default: 1) |
| `per_page` | int | Results per page (default: 20, max: 100) |
| `status` | string | Filter by status |
| `room_type` | string | Filter by type |
| `price_min` | float | Minimum price |
| `price_max` | float | Maximum price |
| `latitude` | float | Center latitude for proximity search |
| `longitude` | float | Center longitude for proximity search |
| `radius` | float | Search radius in km |
| `owner_id` | int | Filter by owner |
| `sort` | string | `price_asc`, `price_desc`, `newest`, `nearest` |

---

## 4. Verification Endpoints (Phase 2)

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/verifications` | POST | Required | `unihome_create_room` | Submit verification |
| `/verifications/{id}` | GET | Required | Owner or verifier | Get verification details |
| `/verifications/queue` | GET | Required | `unihome_verify_rooms` | Verifier queue |
| `/verifications/{id}/assign` | PATCH | Required | `unihome_manage_verifications` | Assign verifier |
| `/verifications/{id}/approve` | PATCH | Required | `unihome_verify_rooms` | Approve verification |
| `/verifications/{id}/reject` | PATCH | Required | `unihome_verify_rooms` | Reject verification |

---

## 5. Roommate Endpoints (Phase 3)

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/profile/roommate` | GET | Required | `unihome_create_profile` | Get own profile |
| `/profile/roommate` | POST | Required | `unihome_create_profile` | Create/update profile |
| `/matches` | GET | Required | `unihome_create_profile` | Get match suggestions |
| `/matches/{id}/accept` | PATCH | Required | `unihome_create_profile` | Accept match |
| `/matches/{id}/decline` | PATCH | Required | `unihome_create_profile` | Decline match |

---

## 6. Chat Endpoints (Phase 4)

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/conversations` | GET | Required | `unihome_send_message` | List user conversations |
| `/conversations` | POST | Required | `unihome_send_message` | Start conversation |
| `/conversations/{id}/messages` | GET | Required | Participant only | Get messages |
| `/conversations/{id}/messages` | POST | Required | Participant only | Send message |
| `/conversations/{id}/read` | PATCH | Required | Participant only | Mark as read |

---

## 7. Recommendation Endpoints (Phase 5)

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/recommendations` | GET | Required | `unihome_view_rooms` | Personalized room feed |
| `/recommendations/refresh` | POST | Required | `unihome_view_rooms` | Force recalculation |

---

## 8. Notification Endpoints (Phase 0+)

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/notifications` | GET | Required | Authenticated | List user notifications |
| `/notifications/{id}/read` | PATCH | Required | Owner only | Mark as read |
| `/notifications/read-all` | PATCH | Required | Authenticated | Mark all as read |
| `/notifications/unread-count` | GET | Required | Authenticated | Get unread count |
| `/notifications/preferences` | GET | Required | Authenticated | Get notification prefs |
| `/notifications/preferences` | PUT | Required | Authenticated | Update notification prefs |

---

## 9. Payment Endpoints (Phase 6)

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/packages` | GET | Optional | — | List available packages |
| `/packages/{id}` | GET | Optional | — | Get package details |
| `/payments` | POST | Required | Authenticated | Purchase a package |
| `/payments/{id}` | GET | Required | Owner or admin | Get payment status |
| `/payments/history` | GET | Required | Authenticated | Payment history |

---

## 10. Admin Endpoints

| Endpoint | Method | Auth | Capability | Description |
|---|---|---|---|---|
| `/admin/rooms` | GET | Required | `unihome_manage_rooms` | All rooms (admin view) |
| `/admin/rooms/{id}/status` | PATCH | Required | `unihome_manage_rooms` | Change room status |
| `/admin/verifications` | GET | Required | `unihome_manage_verifications` | All verifications |
| `/admin/stats` | GET | Required | `manage_options` | Platform statistics |

---

## 11. Route Registration Pattern

All routes are registered in `api/class-rest-router.php`:

```php
class REST_Router {
    public function register_routes(): void {
        // Health check
        register_rest_route( 'unihome/v1', '/health', [
            'methods'             => WP_REST_Server::READABLE,
            'callback'            => [ $this, 'health_check' ],
            'permission_callback' => '__return_true',
        ] );

        // Room routes
        register_rest_route( 'unihome/v1', '/rooms', [
            [
                'methods'             => WP_REST_Server::READABLE,
                'callback'            => [ $this->room_controller, 'list_rooms' ],
                'permission_callback' => '__return_true',
            ],
            [
                'methods'             => WP_REST_Server::CREATABLE,
                'callback'            => [ $this->room_controller, 'create_room' ],
                'permission_callback' => [ $this->room_controller, 'can_create_room' ],
            ],
        ] );

        // ... additional routes per module
    }
}
```

**Rule:** No REST routes are registered outside `class-rest-router.php`. All routes are centralized.
