# UniHome — WordPress Development Guide

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §4  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## Foundational Principle

> We are not building a WordPress website.  
> We are building an application on top of WordPress.

WordPress is the **application framework**. It provides infrastructure we consume. We do not use WordPress as a content management system for business data.

See `docs/decision-log.md` ADR-002 for the full rationale.

---

## 1. What WordPress Provides

| Capability | WordPress API | UniHome Usage |
|---|---|---|
| **Authentication** | `wp_login`, `wp_logout`, `is_user_logged_in()` | Login/logout/session for all user types |
| **User Roles** | `add_role()`, `current_user_can()` | Custom roles: `student`, `landlord`, `verifier`, `admin` |
| **Admin Panel** | `add_menu_page()`, `add_submenu_page()` | Custom admin pages for management |
| **Media** | `wp_handle_upload()`, attachment system | Room photos, verification documents |
| **Plugin Lifecycle** | `register_activation_hook()`, `register_deactivation_hook()` | DB migration on activation, cleanup on deactivation |
| **Security** | `wp_nonce_field()`, `wp_verify_nonce()`, `sanitize_text_field()` | CSRF protection, input sanitization |
| **Database** | `$wpdb->prepare()`, `$wpdb->insert()`, `$wpdb->get_results()` | All database queries |
| **REST API** | `register_rest_route()`, `WP_REST_Request`, `WP_REST_Response` | All HTTP endpoints |
| **Cron** | `wp_schedule_event()`, `wp_cron` | Scheduled tasks (with system cron in production) |
| **Transients** | `set_transient()`, `get_transient()`, `delete_transient()` | Caching |
| **Email** | `wp_mail()` | Email delivery (via NotificationService only) |
| **Options** | `get_option()`, `update_option()` | Plugin config, feature flags |

---

## 2. What We Do NOT Use WordPress For

| Forbidden | Reason |
|---|---|
| **WooCommerce** | Payment logic is custom |
| **Custom Post Types for business data** | Rooms, verifications, profiles use custom tables (ADR-001) |
| **Page builders** (Elementor, WPBakery, Gutenberg blocks) | Application UI is code-controlled |
| **Third-party plugins for business logic** | No "booking plugin" or "matching plugin" |
| **WordPress theme customizer** | Theme is code-controlled |
| **`admin-ajax.php`** | REST API only for new endpoints |
| **`get_posts()` / `WP_Query` for business data** | Use Repository classes |

---

## 3. Hook Conventions

### Action Hooks

UniHome registers actions in `class-unihome-loader.php`:

```php
// Registration pattern
add_action( 'rest_api_init', [ $rest_router, 'register_routes' ] );
add_action( 'init', [ $this, 'register_roles' ] );
add_action( 'admin_menu', [ $admin_pages, 'register_menus' ] );
add_action( 'wp_enqueue_scripts', [ $this, 'enqueue_frontend_assets' ] );
add_action( 'admin_enqueue_scripts', [ $this, 'enqueue_admin_assets' ] );
```

### Filter Hooks

```php
// Only use filters for WordPress-level customization
add_filter( 'upload_mimes', [ $this, 'add_custom_mimes' ] );
add_filter( 'wp_mail_content_type', [ $this, 'set_html_mail' ] );
```

### Custom Hooks (UniHome-specific)

For cross-module communication that doesn't fit the Service call pattern:

```php
// Firing a custom action
do_action( 'unihome_room_status_changed', $room_id, $old_status, $new_status );

// Listening to a custom action
add_action( 'unihome_room_status_changed', [ $notification_service, 'on_room_status_changed' ], 10, 3 );
```

**Naming convention:** All custom hooks use the `unihome_` prefix.

---

## 4. REST API Pattern

All endpoints use `register_rest_route`. See `docs/api.md` for the full endpoint catalog.

```php
register_rest_route( 'unihome/v1', '/rooms', [
    'methods'             => WP_REST_Server::CREATABLE,
    'callback'            => [ $controller, 'create_room' ],
    'permission_callback' => [ $controller, 'can_create_room' ],
] );
```

**Rules:**
- Namespace: `unihome/v1`
- Every route has a `permission_callback` (never `__return_true` for authenticated endpoints).
- Input from `WP_REST_Request::get_param()`, never from `$_POST` or `$_GET`.
- Responses via `WP_REST_Response`, never `wp_send_json_*`.
- Nonce verification is automatic for cookie-authenticated REST requests.

---

## 5. Database Access

All queries go through `$wpdb` in Repository classes:

```php
// Always use prepare() for user input
$room = $wpdb->get_row(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}unihome_rooms WHERE id = %d",
        $room_id
    )
);

// Insert
$wpdb->insert(
    $wpdb->prefix . 'unihome_rooms',
    [
        'owner_id' => $owner_id,
        'title'    => $title,
        'price'    => $price,
        'status'   => 'draft',
    ],
    [ '%d', '%s', '%f', '%s' ]
);

// Update
$wpdb->update(
    $wpdb->prefix . 'unihome_rooms',
    [ 'status' => $new_status ],
    [ 'id' => $room_id ],
    [ '%s' ],
    [ '%d' ]
);
```

**Never hardcode table names.** Always use `$wpdb->prefix . 'unihome_rooms'`.

---

## 6. User Roles and Capabilities

See `docs/roles-and-capabilities.md` for the full matrix.

```php
// Registration (in class-unihome-activator.php)
add_role( 'student', 'Student', [
    'read'                   => true,
    'unihome_view_rooms'     => true,
    'unihome_create_profile' => true,
    'unihome_send_message'   => true,
] );

// Checking (in Controller permission_callback)
public function can_create_room( WP_REST_Request $request ): bool {
    return current_user_can( 'unihome_create_room' );
}
```

**Capability naming:** `unihome_{action}_{entity}` (e.g., `unihome_create_room`, `unihome_verify_rooms`).

---

## 7. Media Upload

Room photos and verification documents use WordPress media library:

```php
// In Controller
$attachment_id = media_handle_upload( 'file', 0 );
if ( is_wp_error( $attachment_id ) ) {
    return new WP_REST_Response( [ 'message' => 'Upload failed' ], 400 );
}

// Link to room via junction table
$this->room_service->add_image( $room_id, $attachment_id, $sort_order );
```

**Storage:** Local `wp-content/uploads/` for MVP. Cloud migration (S3/GCS) is a future infrastructure task.

---

## 8. WP-Cron

WordPress cron is **not real cron** — it fires on page loads.

### Development / Staging
- `wp_cron` is acceptable.

### Production
```
# System crontab
* * * * * wget -q -O - https://example.com/wp-cron.php?doing_wp_cron > /dev/null 2>&1
```

```php
// wp-config.php
define( 'DISABLE_WP_CRON', true );
```

### Registering Scheduled Tasks

```php
// In activation hook
if ( ! wp_next_scheduled( 'unihome_expire_listings' ) ) {
    wp_schedule_event( time(), 'hourly', 'unihome_expire_listings' );
}

// Handler
add_action( 'unihome_expire_listings', [ $room_service, 'expire_old_listings' ] );
```

---

## 9. Asset Enqueuing

```php
// Frontend assets
add_action( 'wp_enqueue_scripts', function() {
    wp_enqueue_style(
        'unihome-main',
        plugins_url( 'assets/css/main.css', UNIHOME_PLUGIN_FILE ),
        [],
        UNIHOME_VERSION
    );

    wp_enqueue_script(
        'unihome-search',
        plugins_url( 'assets/js/search.js', UNIHOME_PLUGIN_FILE ),
        [],
        UNIHOME_VERSION,
        true // in footer
    );

    // Pass data to JS
    wp_localize_script( 'unihome-search', 'unihomeAPI', [
        'root'  => esc_url_raw( rest_url( 'unihome/v1/' ) ),
        'nonce' => wp_create_nonce( 'wp_rest' ),
    ] );
} );
```

---

## 10. WordPress Configuration (wp-config.php)

```php
// Debug (development only)
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );

// UniHome environment
define( 'UNIHOME_ENV', 'development' );         // development | staging | production
define( 'UNIHOME_PAYMENT_MODE', 'sandbox' );    // sandbox | live
define( 'UNIHOME_MAPS_API_KEY', '...' );
define( 'UNIHOME_GEMINI_API_KEY', '...' );

// Production cron
define( 'DISABLE_WP_CRON', true );              // Only in production/staging
```
