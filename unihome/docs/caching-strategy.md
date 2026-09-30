# UniHome — Caching Strategy

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §4  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Caching Layer

UniHome uses WordPress **Transients** for caching. Transients store temporary data in `wp_options` (or in an external object cache like Redis/Memcached if configured).

Access is centralized through `shared/class-cache-manager.php`.

---

## 2. Key Naming Convention

```
unihome_{module}_{entity}_{identifier}
```

Examples:
| Key | Description |
|---|---|
| `unihome_rooms_room_42` | Cached room data for room ID 42 |
| `unihome_rooms_search_abc123` | Cached search results (hash of query params) |
| `unihome_rooms_count_active` | Count of active rooms |
| `unihome_recommendation_user_15` | Recommendation scores for user 15 |
| `unihome_notification_unread_15` | Unread notification count for user 15 |
| `unihome_roommate_matches_15` | Match results for user 15 |

**Rules:**
- Always use the `unihome_` prefix to avoid collisions with other plugins.
- Use module name as second segment for easy identification and bulk invalidation.
- Keep keys under 172 characters (WordPress transient key limit).

---

## 3. TTL Definitions

| Cache Target | TTL | Rationale |
|---|---|---|
| Room detail | 1 hour | Room data changes infrequently |
| Search results | 15 minutes | Balance between freshness and load reduction |
| Active room count | 30 minutes | Used in dashboard stats |
| Recommendation scores | 2 hours | Computed scores don't change rapidly |
| Unread notification count | 5 minutes | Must feel near-real-time |
| Match results | 1 hour | Recalculated periodically |
| Geocoded addresses | 24 hours | External API results (Google Maps) |
| User notification preferences | 1 hour | Rarely changes |

---

## 4. CacheManager Usage

```php
use UniHome\Shared\CacheManager;

// Get cached value
$room = CacheManager::get( 'rooms', 'room', $room_id );
if ( $room !== false ) {
    return $room;
}

// Fetch from DB
$room = $this->repository->get_by_id( $room_id );

// Store in cache
CacheManager::set( 'rooms', 'room', $room_id, $room, HOUR_IN_SECONDS );

return $room;
```

### CacheManager Interface

```php
class CacheManager {
    /**
     * Get a cached value.
     * @return mixed|false  Cached value or false if not found.
     */
    public static function get( string $module, string $entity, string|int $id ): mixed;

    /**
     * Set a cached value.
     */
    public static function set( string $module, string $entity, string|int $id, mixed $data, int $ttl ): bool;

    /**
     * Delete a cached value.
     */
    public static function delete( string $module, string $entity, string|int $id ): bool;

    /**
     * Delete all cache entries for a module.
     */
    public static function flush_module( string $module ): void;
}
```

---

## 5. Invalidation Rules

Cache entries must be invalidated when the underlying data changes.

| Event | Invalidate |
|---|---|
| Room created | `rooms_count_active` |
| Room updated | `rooms_room_{id}`, `rooms_search_*` |
| Room status changed | `rooms_room_{id}`, `rooms_count_active`, `rooms_search_*` |
| Verification approved | `rooms_room_{room_id}` (verified badge changes display) |
| New message | `notification_unread_{user_id}` |
| Notification read | `notification_unread_{user_id}` |
| Match computed | `roommate_matches_{user_id}` |
| Recommendation recalculated | `recommendation_user_{user_id}` |

### Invalidation Pattern

```php
// In RoomService::update_room()
public function update_room( int $id, array $data ): array|WP_Error {
    $result = $this->repository->update( $id, $data );

    if ( ! is_wp_error( $result ) ) {
        CacheManager::delete( 'rooms', 'room', $id );
        CacheManager::flush_module( 'rooms_search' ); // All search caches
    }

    return $result;
}
```

---

## 6. Search Result Caching

Search results are cached with a hash of the query parameters:

```php
$cache_key = md5( wp_json_encode( $filters ) );
$cached = CacheManager::get( 'rooms', 'search', $cache_key );

if ( $cached !== false ) {
    return $cached;
}

$results = $this->repository->search( $filters );
CacheManager::set( 'rooms', 'search', $cache_key, $results, 15 * MINUTE_IN_SECONDS );
```

---

## 7. External API Response Caching

Google Maps geocoding responses are cached to reduce API calls:

```php
$cache_key = md5( $address );
$coords = CacheManager::get( 'maps', 'geocode', $cache_key );

if ( $coords === false ) {
    $coords = $this->google_maps_client->geocode( $address );
    CacheManager::set( 'maps', 'geocode', $cache_key, $coords, DAY_IN_SECONDS );
}
```

---

## 8. Production Considerations

- For production, configure a persistent **object cache** (Redis or Memcached) via a WordPress drop-in (`wp-content/object-cache.php`).
- With object cache, transients are stored in memory instead of `wp_options`, significantly improving performance.
- Without object cache, transients with expiration are stored in `wp_options` with an additional `_transient_timeout_` row. This is acceptable for MVP but not optimal at scale.
