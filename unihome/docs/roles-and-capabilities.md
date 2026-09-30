# UniHome — Roles and Capabilities

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §4  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Custom Roles

| Role | Slug | Description |
|---|---|---|
| **Student** | `student` | Searches rooms, creates roommate profile, sends messages, views recommendations |
| **Landlord** | `landlord` | Lists rooms, uploads images, submits verifications, receives messages |
| **Verifier** | `verifier` | Reviews and approves/rejects verification submissions |
| **Admin** | `administrator` | Full platform management (WordPress built-in role, extended) |

---

## 2. Capability Matrix

### Room Module

| Capability | Student | Landlord | Verifier | Admin |
|---|---|---|---|---|
| `unihome_view_rooms` | ✅ | ✅ | ✅ | ✅ |
| `unihome_create_room` | ❌ | ✅ | ❌ | ✅ |
| `unihome_edit_room` | ❌ | ✅ (own) | ❌ | ✅ |
| `unihome_delete_room` | ❌ | ✅ (own) | ❌ | ✅ |
| `unihome_manage_rooms` | ❌ | ❌ | ❌ | ✅ |

### Verification Module

| Capability | Student | Landlord | Verifier | Admin |
|---|---|---|---|---|
| `unihome_submit_verification` | ❌ | ✅ | ❌ | ✅ |
| `unihome_verify_rooms` | ❌ | ❌ | ✅ | ✅ |
| `unihome_manage_verifications` | ❌ | ❌ | ❌ | ✅ |

### Roommate Module

| Capability | Student | Landlord | Verifier | Admin |
|---|---|---|---|---|
| `unihome_create_profile` | ✅ | ❌ | ❌ | ✅ |
| `unihome_view_matches` | ✅ | ❌ | ❌ | ✅ |
| `unihome_manage_matches` | ❌ | ❌ | ❌ | ✅ |

### Chat Module

| Capability | Student | Landlord | Verifier | Admin |
|---|---|---|---|---|
| `unihome_send_message` | ✅ | ✅ | ❌ | ✅ |
| `unihome_view_conversations` | ✅ | ✅ | ❌ | ✅ |
| `unihome_manage_messages` | ❌ | ❌ | ❌ | ✅ |

### Payment Module

| Capability | Student | Landlord | Verifier | Admin |
|---|---|---|---|---|
| `unihome_purchase_package` | ❌ | ✅ | ❌ | ✅ |
| `unihome_view_payments` | ❌ | ✅ (own) | ❌ | ✅ |
| `unihome_manage_payments` | ❌ | ❌ | ❌ | ✅ |

### Notification (all authenticated users)

| Capability | Student | Landlord | Verifier | Admin |
|---|---|---|---|---|
| `unihome_view_notifications` | ✅ | ✅ | ✅ | ✅ |

### Admin

| Capability | Student | Landlord | Verifier | Admin |
|---|---|---|---|---|
| `unihome_view_analytics` | ❌ | ❌ | ❌ | ✅ |
| `unihome_manage_settings` | ❌ | ❌ | ❌ | ✅ |

---

## 3. Role Registration

```php
// In class-unihome-activator.php
public static function create_roles(): void {
    add_role( 'student', 'Student', [
        'read'                       => true,
        'unihome_view_rooms'         => true,
        'unihome_create_profile'     => true,
        'unihome_view_matches'       => true,
        'unihome_send_message'       => true,
        'unihome_view_conversations' => true,
        'unihome_view_notifications' => true,
    ] );

    add_role( 'landlord', 'Landlord', [
        'read'                        => true,
        'unihome_view_rooms'          => true,
        'unihome_create_room'         => true,
        'unihome_edit_room'           => true,
        'unihome_delete_room'         => true,
        'unihome_submit_verification' => true,
        'unihome_send_message'        => true,
        'unihome_view_conversations'  => true,
        'unihome_view_notifications'  => true,
        'unihome_purchase_package'    => true,
        'unihome_view_payments'       => true,
    ] );

    add_role( 'verifier', 'Verifier', [
        'read'                       => true,
        'unihome_view_rooms'         => true,
        'unihome_verify_rooms'       => true,
        'unihome_view_notifications' => true,
    ] );

    // Extend administrator
    $admin = get_role( 'administrator' );
    if ( $admin ) {
        $admin->add_cap( 'unihome_view_rooms' );
        $admin->add_cap( 'unihome_create_room' );
        $admin->add_cap( 'unihome_edit_room' );
        $admin->add_cap( 'unihome_delete_room' );
        $admin->add_cap( 'unihome_manage_rooms' );
        $admin->add_cap( 'unihome_submit_verification' );
        $admin->add_cap( 'unihome_verify_rooms' );
        $admin->add_cap( 'unihome_manage_verifications' );
        $admin->add_cap( 'unihome_create_profile' );
        $admin->add_cap( 'unihome_view_matches' );
        $admin->add_cap( 'unihome_manage_matches' );
        $admin->add_cap( 'unihome_send_message' );
        $admin->add_cap( 'unihome_view_conversations' );
        $admin->add_cap( 'unihome_manage_messages' );
        $admin->add_cap( 'unihome_view_notifications' );
        $admin->add_cap( 'unihome_purchase_package' );
        $admin->add_cap( 'unihome_view_payments' );
        $admin->add_cap( 'unihome_manage_payments' );
        $admin->add_cap( 'unihome_view_analytics' );
        $admin->add_cap( 'unihome_manage_settings' );
    }
}
```

---

## 4. Ownership Rules

- `✅ (own)` means the capability only applies to resources owned by the user.
- Ownership is checked in the Service layer: `if ( $room->owner_id !== get_current_user_id() ) return new WP_Error(...)`.
- Admin (`administrator`) bypasses ownership checks via `unihome_manage_*` capabilities.

---

## 5. Role Cleanup

On plugin deactivation, custom roles are **not** removed (WordPress convention — users would lose their role assignment). Roles are only removed on plugin **uninstall** (if the plugin provides an uninstall hook).
