# UniHome — Error Handling Conventions

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §7  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Error Flow

```
Repository  →  returns null/false or WP_Error
     │
     ▼
Service     →  returns WP_Error on failure, data on success
     │
     ▼
Controller  →  translates WP_Error to HTTP status + JSON response
```

---

## 2. Layer-Specific Rules

### Repository

- Returns `null` or `false` for not-found.
- Returns `WP_Error` for database errors (failed queries, constraint violations).
- **Never** throws exceptions.

```php
public function get_by_id( int $id ): ?array {
    global $wpdb;
    $row = $wpdb->get_row(
        $wpdb->prepare( "SELECT * FROM {$wpdb->prefix}unihome_rooms WHERE id = %d", $id ),
        ARRAY_A
    );
    return $row ?: null;
}

public function insert( array $data ): int|WP_Error {
    global $wpdb;
    $result = $wpdb->insert( $wpdb->prefix . 'unihome_rooms', $data, [...] );
    if ( false === $result ) {
        Logger::error( 'rooms', 'Insert failed: ' . $wpdb->last_error );
        return new WP_Error( 'db_insert_failed', 'Failed to create room', [ 'status' => 500 ] );
    }
    return $wpdb->insert_id;
}
```

### Service

- Returns data on success.
- Returns `WP_Error` on failure with a descriptive error code.
- **Never** throws exceptions (WordPress convention).
- Error codes follow format: `{action}_{reason}` (e.g., `create_room_missing_title`).

```php
public function create_room( array $data ): array|WP_Error {
    if ( empty( $data['title'] ) ) {
        return new WP_Error( 'missing_title', 'Room title is required', [ 'status' => 400 ] );
    }

    if ( $data['price'] <= 0 ) {
        return new WP_Error( 'invalid_price', 'Price must be positive', [ 'status' => 400 ] );
    }

    $room_id = $this->repository->insert( $data );
    if ( is_wp_error( $room_id ) ) {
        return $room_id; // Propagate the repository error
    }

    Logger::audit( 'room', $room_id, 'created', null, $data );
    return $this->repository->get_by_id( $room_id );
}
```

### Controller

- Translates `WP_Error` into HTTP responses.
- Uses the `status` key from `WP_Error` error data for HTTP status code.

```php
public function create_room( WP_REST_Request $request ): WP_REST_Response {
    $data = $this->sanitize_input( $request );
    $result = $this->room_service->create_room( $data );

    if ( is_wp_error( $result ) ) {
        $status = $result->get_error_data()['status'] ?? 500;
        return new WP_REST_Response(
            [
                'message' => $result->get_error_message(),
                'code'    => $result->get_error_code(),
            ],
            $status
        );
    }

    return new WP_REST_Response( [ 'data' => $result ], 201 );
}
```

---

## 3. WP_Error Conventions

### Error Code Format

```
{module}_{action}_{reason}
```

Examples:
- `room_create_missing_title`
- `verification_approve_invalid_status`
- `match_accept_already_accepted`
- `payment_process_gateway_error`

### Error Data

Always include `status` for HTTP mapping:

```php
new WP_Error(
    'room_not_found',                    // code
    'Room with ID 123 does not exist',   // message
    [ 'status' => 404 ]                  // data with HTTP status
);
```

---

## 4. Logging

### Logger Class

All logging goes through `shared/class-logger.php`:

```php
// Error logging
Logger::error( 'rooms', 'Failed to create room: ' . $wpdb->last_error );
// Output: [2026-10-01 10:30:00] [UniHome:rooms] ERROR: Failed to create room: ...

// Warning
Logger::warning( 'verification', 'Verifier queue is empty' );
// Output: [2026-10-01 10:30:00] [UniHome:verification] WARNING: Verifier queue is empty

// Info
Logger::info( 'chat', 'Conversation created between users 1 and 2' );
// Output: [2026-10-01 10:30:00] [UniHome:chat] INFO: Conversation created...

// Audit (business-critical state changes)
Logger::audit( 'verification', $id, 'status_changed', 'pending_review', 'approved' );
// Writes to wp_unihome_audit_log table
```

### Log Destination

- Development: `wp-content/debug.log` (via `error_log()`)
- Production: Same file, but monitored via log aggregation (future)

### Module Prefix

All log entries include `[UniHome:{module}]` for filtering:

```
[UniHome:rooms] ERROR: Insert failed: Duplicate entry
[UniHome:verification] INFO: Verification #42 approved by user #5
[UniHome:payment] ERROR: Gateway timeout after 30s
```

---

## 5. Audit Logging

State changes that **must** generate audit entries:

| Module | Events |
|---|---|
| Room | Status change, creation, update, deletion (soft) |
| Verification | Status change (every transition), verifier assignment |
| Payment | Payment created, status change |
| Match | Status change (suggested → accepted/declined) |
| User | Role change, account disable |

### Audit Log Entry

```php
Logger::audit(
    'room',              // entity_type
    $room_id,            // entity_id
    'status_changed',    // action
    $old_status,         // old_value (JSON-encoded if complex)
    $new_status          // new_value (JSON-encoded if complex)
);
```

This writes to `wp_unihome_audit_log` with the current user ID and IP address.

---

## 6. Cron Job Error Handling

Failed cron jobs are:
1. Logged via `Logger::error()`.
2. Surfaced as a WordPress admin notice on the UniHome admin page.

```php
// In cron handler
public function expire_old_listings(): void {
    try {
        $expired = $this->room_service->expire_listings_older_than( 90 );
        Logger::info( 'rooms', "Expired {$expired} listings" );
    } catch ( \Throwable $e ) {
        Logger::error( 'rooms', 'Listing expiration cron failed: ' . $e->getMessage() );
        update_option( 'unihome_cron_last_error', [
            'task'    => 'expire_listings',
            'message' => $e->getMessage(),
            'time'    => current_time( 'mysql' ),
        ] );
    }
}
```

---

## 7. Forbidden Error Patterns

| Pattern | Why It's Forbidden |
|---|---|
| `throw new \Exception(...)` in Service | Services return `WP_Error`, not exceptions |
| `die()` / `exit()` in Service or Repository | Kills the entire request; untestable |
| `wp_die()` in Service or Repository | Only valid in admin form handlers |
| Silent failure (no logging) | All errors must be logged |
| Generic error messages | Errors must be specific and actionable |
