# UniHome — Phase 1: Room Marketplace

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Not Started  
> **Target Duration:** ~4 weeks  
> **Maintained By:** Module Owner (@dev-1) & Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Goal

Build the core Room Marketplace module: allow landlords to publish and manage room listings with image galleries, enable students to search, filter, and view detailed listing pages, and provide administrators with listing management capabilities.

---

## 2. Team Allocation & Responsibilities

| Area | Task | Owner | Key Deliverables |
|---|---|---|---|
| Database | Schema migration for `wp_unihome_rooms` & `wp_unihome_room_images` | Dev 1 | Migration classes, table definitions, FK constraints |
| Backend | `RoomService` — CRUD, status workflow, business rules | Dev 1 | Status transitions (`draft`, `pending`, `active`, `rented`, `inactive`), audit logging |
| Backend | `RoomRepository` — SQL data access & filtering queries | Dev 2 | High-performance prepared SQL, index utilization, spatial/attribute filters |
| Backend | `RoomController` — REST API endpoints (`/unihome/v1/rooms`) | Dev 2 | Route handlers, request validation, response serialization |
| Backend | Media handler — WordPress media integration for room images | Dev 3 | Image upload validation, attachment mapping, thumbnail generation |
| Frontend | Room listing catalog — grid/list layout with pagination | Dev 3 | Modern, responsive listing cards, price display, status indicators |
| Frontend | Room detail view — gallery, amenities list, contact CTA | Dev 4 | Image carousel/lightbox, location map embed, responsive action bar |
| Frontend | Search & filter panel | Dev 4 | Dynamic filtering (price range, room type, district/university distance, amenities) |
| Admin | WP Admin management dashboard for rooms | Dev 1 | Admin menu page, listing table, quick status toggles |
| Testing | Unit & integration tests for RoomService & RoomRepository | Dev 2 | Comprehensive test coverage for state machine, repository filters, auth |

---

## 3. Architecture & Data Flow

### 3.1 3-Layer Pattern Implementation
- **Controller (`RoomController`)**: Handles `/wp-json/unihome/v1/rooms` routes. Sanitizes input, checks capabilities (`edit_rooms`, `read_rooms`), delegates to `RoomService`, returns `WP_REST_Response`.
- **Service (`RoomService`)**: Enforces listing limits, validates price ranges, coordinates state transitions (`draft` → `pending` → `active`), triggers audit logs via `AuditService`, and emits events via `EventTracker`.
- **Repository (`RoomRepository`)**: Direct `$wpdb` interaction with `wp_unihome_rooms` and `wp_unihome_room_images`. Handles complex filtering queries with pagination.

### 3.2 Database Schema Reference
- `wp_unihome_rooms`: Primary listing data (landlord ID, title, description, address, price, deposit, room type, size, amenities JSON, status, verification status, timestamps).
- `wp_unihome_room_images`: Media links (room ID, WordPress attachment ID, image URL, display order, is_featured flag).

### 3.3 Analytics & Event Integration
- Track `room_viewed` in `RoomController::get_item()` whenever a room detail endpoint is requested.
- Track `room_search` in `RoomController::get_items()` with search query terms and applied filter dimensions.
- Track `room_saved` (if bookmarking action is invoked).

---

## 4. REST API Endpoints

| Method | Endpoint | Capability / Auth | Description |
|---|---|---|---|
| `GET` | `/wp-json/unihome/v1/rooms` | Public | Paginated room search and filtered catalog |
| `GET` | `/wp-json/unihome/v1/rooms/(?P<id>\d+)` | Public | Room detail view (triggers `room_viewed` tracking) |
| `POST` | `/wp-json/unihome/v1/rooms` | Authenticated (`create_rooms`) | Create a new room draft or submission |
| `PUT` | `/wp-json/unihome/v1/rooms/(?P<id>\d+)` | Authenticated (`edit_room`, owner or admin) | Update existing room listing |
| `DELETE` | `/wp-json/unihome/v1/rooms/(?P<id>\d+)` | Authenticated (`delete_room`, owner or admin) | Soft-delete room (marks status as `inactive`) |
| `POST` | `/wp-json/unihome/v1/rooms/(?P<id>\d+)/images` | Authenticated (owner or admin) | Attach uploaded media images to listing |

---

## 5. Completion Criteria & Definition of Done

- [ ] `wp_unihome_rooms` and `wp_unihome_room_images` tables created with indexes and foreign keys.
- [ ] Landlord can successfully create, edit, and toggle room listings via REST endpoints.
- [ ] Image upload links directly to WordPress attachment library with mime-type and file size validation.
- [ ] Students can query rooms by price range, district, room type, and amenity keywords.
- [ ] Pagination operates efficiently on large result sets without performance degradation.
- [ ] Room status transitions enforce authorized states and write audit log records.
- [ ] `EventTracker::track('room_viewed', ...)` and `EventTracker::track('room_search', ...)` fire reliably.
- [ ] Unit tests for `RoomService` cover valid transitions, invalid status attempts, and authorization checks.
- [ ] Integration tests verify repository search queries return exact filtered counts.

---

## 6. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 1)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
