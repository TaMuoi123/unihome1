# UniHome — Security Guide

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §7  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Authentication

UniHome uses WordPress built-in authentication:

| Mechanism | Context |
|---|---|
| **Cookie auth + REST nonce** | Frontend JS calling REST API |
| **Application Passwords** | External API clients (future) |

### REST API Authentication

```javascript
// Frontend JS — include nonce in all requests
fetch( unihomeAPI.root + 'rooms', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-WP-Nonce': unihomeAPI.nonce,
    },
    body: JSON.stringify( data ),
} );
```

WordPress automatically verifies the nonce for cookie-authenticated REST requests. No manual `wp_verify_nonce()` is needed in Controllers — `WP_REST_Server` handles this.

---

## 2. Authorization (Capability Checks)

Every REST endpoint must have a `permission_callback`:

```php
register_rest_route( 'unihome/v1', '/rooms', [
    'methods'             => WP_REST_Server::CREATABLE,
    'callback'            => [ $controller, 'create_room' ],
    'permission_callback' => function() {
        return current_user_can( 'unihome_create_room' );
    },
] );
```

**Rules:**
- Never use `'permission_callback' => '__return_true'` for authenticated endpoints.
- `__return_true` is only acceptable for public read endpoints (room listing, health check).
- Ownership checks happen in the Service layer (capability grants the action; Service verifies the resource belongs to the user).

See `docs/roles-and-capabilities.md` for the full capability matrix.

---

## 3. Input Sanitization

All user input is sanitized using WordPress functions:

| Data Type | Function |
|---|---|
| Plain text | `sanitize_text_field()` |
| Email | `sanitize_email()` |
| URL | `esc_url_raw()` |
| Integer | `absint()` or `intval()` |
| Float | `floatval()` |
| HTML content | `wp_kses_post()` |
| File name | `sanitize_file_name()` |
| Textarea | `sanitize_textarea_field()` |
| Slug | `sanitize_title()` |

**Rule:** Input is sanitized in the **Controller** before passing to the Service. The Service receives clean data.

```php
// In Controller
$title   = sanitize_text_field( $request->get_param( 'title' ) );
$price   = floatval( $request->get_param( 'price' ) );
$email   = sanitize_email( $request->get_param( 'email' ) );
$address = sanitize_textarea_field( $request->get_param( 'address' ) );
```

---

## 4. SQL Injection Prevention

**All database queries use `$wpdb->prepare()`:**

```php
// ✅ Correct
$room = $wpdb->get_row(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}unihome_rooms WHERE id = %d AND status = %s",
        $room_id,
        'active'
    )
);

// ❌ NEVER DO THIS
$room = $wpdb->get_row(
    "SELECT * FROM {$wpdb->prefix}unihome_rooms WHERE id = {$room_id}"
);
```

**Rules:**
- All `$wpdb` queries in Repository classes use `$wpdb->prepare()`.
- No string concatenation with user input in SQL.
- Code review checklist includes SQL injection check.

---

## 5. Cross-Site Scripting (XSS) Prevention

### Output Escaping

All output is escaped at the template level:

```php
// In templates
<h1><?php echo esc_html( $room->title ); ?></h1>
<a href="<?php echo esc_url( $room->link ); ?>">View</a>
<div><?php echo wp_kses_post( $room->description ); ?></div>
<input value="<?php echo esc_attr( $room->address ); ?>">
```

| Context | Function |
|---|---|
| HTML body | `esc_html()` |
| HTML attribute | `esc_attr()` |
| URL | `esc_url()` |
| JavaScript | `esc_js()` or `wp_json_encode()` |
| Rich HTML | `wp_kses_post()` (allows safe HTML tags) |

**Rule:** Never use `echo $variable` without escaping.

---

## 6. Cross-Site Request Forgery (CSRF) Prevention

### REST API

Automatic via WordPress REST nonce (`X-WP-Nonce` header). No additional CSRF protection needed.

### Admin Forms

```php
// In form template
wp_nonce_field( 'unihome_admin_action', 'unihome_nonce' );

// In handler
if ( ! wp_verify_nonce( $_POST['unihome_nonce'], 'unihome_admin_action' ) ) {
    wp_die( 'Security check failed' );
}
```

---

## 7. File Upload Security

```php
// Allowed file types (in activation)
add_filter( 'upload_mimes', function( $mimes ) {
    // Only allow image types for room photos
    return [
        'jpg|jpeg' => 'image/jpeg',
        'png'      => 'image/png',
        'webp'     => 'image/webp',
        'pdf'      => 'application/pdf', // For verification documents
    ];
} );
```

**Rules:**
- File type validated server-side (never trust client MIME type).
- File size limits enforced.
- Uploaded files stored in `wp-content/uploads/` via WordPress media library.
- Direct file path access is not exposed to users.

---

## 8. Secrets Management

| Secret | Storage | Access |
|---|---|---|
| API keys (Maps, Gemini) | `wp-config.php` constants | `Config::get('maps_api_key')` |
| Payment credentials | `wp-config.php` constants | `Config::get('payment_key')` |
| Database credentials | `wp-config.php` (WP default) | `$wpdb` |

**Rules:**
- **Never** commit secrets to Git.
- **Never** store secrets in `wp_options` (visible in admin).
- `wp-config.php` is not tracked by Git.
- In production, use server environment variables when possible.

---

## 9. OWASP Top 10 — UniHome Mapping

| # | OWASP Risk | UniHome Mitigation |
|---|---|---|
| 1 | Broken Access Control | Capability checks on every endpoint; ownership in Service |
| 2 | Cryptographic Failures | Secrets in `wp-config.php`; WordPress password hashing |
| 3 | Injection (SQL) | `$wpdb->prepare()` everywhere |
| 4 | Insecure Design | 3-layer architecture; code review checklist |
| 5 | Security Misconfiguration | `WP_DEBUG` off in production; hardened `wp-config.php` |
| 6 | Vulnerable Components | Composer audit; WordPress auto-updates for security |
| 7 | Auth Failures | WordPress auth; rate limiting (future) |
| 8 | Data Integrity | Audit log; FK constraints; typed columns |
| 9 | Logging & Monitoring | `Logger::error()` + `Logger::audit()` |
| 10 | Server-Side Request Forgery | No user-controlled URLs in server-side requests |

---

## 10. Security Audit Checklist (Per Phase)

- [ ] All REST endpoints have `permission_callback`
- [ ] No `__return_true` on authenticated endpoints
- [ ] All user input sanitized in Controllers
- [ ] All SQL uses `$wpdb->prepare()`
- [ ] All output escaped in templates
- [ ] All state changes audit-logged
- [ ] File uploads validated server-side
- [ ] No secrets in committed code
- [ ] No direct `$_POST` / `$_GET` access
- [ ] CSRF protection on all admin forms
