# UniHome — Environment Setup

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §3, §4  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Requirements

| Software | Version | Purpose |
|---|---|---|
| PHP | 8.0+ | Application language |
| MySQL | 8.0+ | Database |
| Composer | 2.x | Autoloading and dependencies |
| WordPress | 6.4+ | Application framework |
| Web Server | Apache 2.4+ or Nginx | HTTP serving |
| Git | 2.x | Version control |

---

## 2. Local Development Options

### Option A: WAMP / MAMP / XAMPP (Recommended for beginners)

1. Install WAMP (Windows), MAMP (macOS), or XAMPP.
2. Ensure PHP 8.0+ is selected in the control panel.
3. Ensure MySQL 8.0+ is running.
4. Create a database: `unihome_dev`.

### Option B: Docker

```yaml
# docker-compose.yml (example)
version: '3.8'
services:
  wordpress:
    image: wordpress:6.4-php8.0-apache
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: unihome_dev
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: root
    volumes:
      - ./wp-content:/var/www/html/wp-content

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: unihome_dev
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

---

## 3. WordPress Installation

1. Download WordPress 6.4+ from https://wordpress.org/download/
2. Extract to your web server root (e.g., `C:\wamp64\www\unihome\` or Docker volume).
3. Create database `unihome_dev` in MySQL.
4. Run the WordPress installer at `http://localhost/unihome/`.
5. Complete setup with admin credentials.

---

## 4. Plugin Setup

```bash
# Navigate to WordPress plugins directory
cd wp-content/plugins/

# Clone the plugin (or symlink from your repo)
git clone <repository-url> unihome-plugin

# Install Composer dependencies
cd unihome-plugin
composer install

# Activate plugin
# Go to wp-admin > Plugins > UniHome > Activate
```

### Symlink approach (if repo root is outside WordPress)

```bash
# Windows (run as admin)
mklink /D "C:\wamp64\www\unihome\wp-content\plugins\unihome-plugin" "C:\projects\unihome\wp-content\plugins\unihome-plugin"

# Linux / macOS
ln -s /path/to/unihome/wp-content/plugins/unihome-plugin /path/to/wordpress/wp-content/plugins/unihome-plugin
```

---

## 5. wp-config.php Configuration

Add these constants to `wp-config.php` (above "That's all, stop editing!"):

```php
// Debug settings (development only)
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );          // Logs to wp-content/debug.log
define( 'WP_DEBUG_DISPLAY', false );     // Don't show errors on screen
define( 'SCRIPT_DEBUG', true );          // Use unminified scripts

// UniHome environment
define( 'UNIHOME_ENV', 'development' );
define( 'UNIHOME_PAYMENT_MODE', 'sandbox' );

// API Keys (get from team lead)
define( 'UNIHOME_MAPS_API_KEY', '' );    // Google Maps API key
define( 'UNIHOME_GEMINI_API_KEY', '' );  // Gemini API key

// Database table prefix (WordPress default, do not change)
// $table_prefix = 'wp_';
```

---

## 6. Xdebug Setup (Optional)

### php.ini additions:

```ini
[xdebug]
zend_extension=xdebug
xdebug.mode=debug,coverage
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

### VS Code launch.json:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/var/www/html": "${workspaceFolder}"
            }
        }
    ]
}
```

---

## 7. Verification Checklist

After setup, verify:

- [ ] WordPress loads at `http://localhost/unihome/` (or configured URL)
- [ ] Admin panel accessible at `/wp-admin/`
- [ ] UniHome plugin appears in Plugins list
- [ ] Plugin activates without errors
- [ ] Health check: `GET /wp-json/unihome/v1/health` returns `200`
- [ ] PHPUnit runs: `cd wp-content/plugins/unihome-plugin && vendor/bin/phpunit`
- [ ] `WP_DEBUG_LOG` is enabled and `wp-content/debug.log` is writable
- [ ] Database contains `wp_unihome_*` tables (after plugin activation)

---

## 8. Common Issues

| Issue | Solution |
|---|---|
| Composer not found | Install from https://getcomposer.org/ |
| PHP version < 8.0 | Update PHP in WAMP/MAMP or Docker image |
| MySQL JSON functions not available | Ensure MySQL 8.0+ (not 5.7) |
| Plugin activation fails | Check `wp-content/debug.log` for errors |
| REST API returns 404 | Flush permalinks: Settings > Permalinks > Save |
| `vendor/` missing | Run `composer install` in plugin directory |
