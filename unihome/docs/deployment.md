# UniHome — Deployment Guide

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Environment Overview

| Environment | Purpose | URL | Deploy Method |
|---|---|---|---|
| **Development** | Local dev machines | `localhost` | Manual |
| **Staging** | Pre-production testing | TBD | Git pull + script |
| **Production** | Live application | TBD | Git pull + script |

---

## 2. Environment Differences

| Setting | Development | Staging | Production |
|---|---|---|---|
| `WP_DEBUG` | `true` | `true` | `false` |
| `WP_DEBUG_LOG` | `true` | `true` | `true` |
| `WP_DEBUG_DISPLAY` | `false` | `false` | `false` |
| `SCRIPT_DEBUG` | `true` | `false` | `false` |
| `DISABLE_WP_CRON` | `false` | `true` | `true` |
| `UNIHOME_ENV` | `development` | `staging` | `production` |
| `UNIHOME_PAYMENT_MODE` | `sandbox` | `sandbox` | `live` |
| System cron | Not required | Required | Required |
| Object cache | Not required | Recommended | Required |
| SSL | Not required | Required | Required |

---

## 3. Deploy Checklist

### Pre-Deploy

- [ ] All tests pass locally (`vendor/bin/phpunit`)
- [ ] Code reviewed and merged to `main` (for production) or `develop` (for staging)
- [ ] `STATUS.md` updated with completed tasks
- [ ] No secrets in committed code
- [ ] Database migrations tested on a clean install

### Deploy Steps

```bash
# 1. Pull latest code
cd /path/to/wordpress/wp-content/plugins/unihome-plugin
git pull origin main

# 2. Install/update Composer dependencies
composer install --no-dev --optimize-autoloader

# 3. Run database migrations (automatic on next page load via activation check)
# Or manually trigger:
wp eval 'do_action("unihome_run_migrations");'

# 4. Flush caches
wp cache flush
wp transient delete --all

# 5. Flush rewrite rules (if new REST routes added)
wp rewrite flush

# 6. Verify health check
curl -s https://example.com/wp-json/unihome/v1/health
```

### Post-Deploy

- [ ] Health check returns `200`
- [ ] Spot-check critical flows (room listing, search, login)
- [ ] Check `wp-content/debug.log` for new errors
- [ ] Monitor for 5 minutes for error spikes

---

## 4. Database Migration on Deploy

Migrations run automatically when the plugin detects a version change:

```php
// In unihome.php or class-unihome-loader.php
$current_version = get_option( 'unihome_db_version', '0' );
if ( version_compare( $current_version, UNIHOME_DB_VERSION, '<' ) ) {
    UniHome_Activator::run_migrations();
    update_option( 'unihome_db_version', UNIHOME_DB_VERSION );
}
```

**Rules:**
- Migrations are forward-only. Never modify a committed migration.
- FK constraints applied in a separate pass after `dbDelta()`.
- Test migrations on staging before production.
- Back up the database before running migrations in production.

---

## 5. Rollback Procedure

### Code Rollback

```bash
# Revert to previous commit
cd /path/to/wordpress/wp-content/plugins/unihome-plugin
git log --oneline -5           # Find the commit to revert to
git checkout <previous-commit>

# Re-install dependencies
composer install --no-dev --optimize-autoloader

# Flush caches
wp cache flush
```

### Database Rollback

- Migrations are forward-only. There is no automatic rollback.
- If a migration breaks production, **restore from backup**.
- This is why database backups before deploy are mandatory.

---

## 6. Production Cron Setup

WordPress pseudo-cron (`wp_cron`) is disabled in production. A real system cron triggers it:

```cron
# /etc/crontab or crontab -e
* * * * * www-data wget -q -O /dev/null https://example.com/wp-cron.php?doing_wp_cron
```

```php
// wp-config.php
define( 'DISABLE_WP_CRON', true );
```

---

## 7. Production Security Hardening

```php
// wp-config.php additions for production
define( 'DISALLOW_FILE_EDIT', true );    // Disable plugin/theme editor
define( 'DISALLOW_FILE_MODS', true );    // Disable plugin installs from admin
define( 'WP_AUTO_UPDATE_CORE', true );   // Auto-update WordPress core (security)
```

### .htaccess (Apache)

```apache
# Block access to sensitive files
<Files wp-config.php>
    Order deny,allow
    Deny from all
</Files>

<Files debug.log>
    Order deny,allow
    Deny from all
</Files>
```

---

## 8. Monitoring (Future)

| What to Monitor | How |
|---|---|
| Error rate | Parse `debug.log` for `[UniHome:*] ERROR` |
| Cron failures | Check `unihome_cron_last_error` option |
| Response time | Server access logs or APM tool |
| Database size | `wp_unihome_events` and `wp_unihome_audit_log` growth |
| Cache hit rate | Object cache stats (Redis INFO) |
