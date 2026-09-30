# UniHome — Student Housing Marketplace

> A WordPress-based marketplace platform connecting students with housing, roommates, and campus services.

---

## Overview

UniHome is a multi-module SaaS application built on WordPress as an application framework. It provides:

- **Room Marketplace** — landlords list rooms, students search and rent
- **Verification System** — rooms verified through a trust workflow
- **Roommate Matching** — compatibility-based roommate suggestions
- **Communication** — messaging between students and landlords
- **Recommendation Engine** — personalized room recommendations
- **Monetization** — premium listings, packages, subscriptions

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| CMS Framework | WordPress | 6.4+ |
| Language | PHP | 8.0+ |
| Database | MySQL | 8.0+ |
| Frontend | Vanilla JS + CSS | — |
| Autoloading | Composer (PSR-4) | 2.x |
| Testing | PHPUnit + WP_UnitTestCase | 9.x |

---

## Quick Start

### Prerequisites

- PHP 8.0+
- MySQL 8.0+
- Composer 2.x
- WordPress 6.4+ installed
- Local server: WAMP / MAMP / Docker (see `docs/environment-setup.md`)

### Setup

```bash
# 1. Clone the repository
git clone <repository-url> unihome
cd unihome

# 2. Install WordPress (if not already installed)
# Follow docs/environment-setup.md for detailed instructions

# 3. Navigate to the plugin directory
cd wp-content/plugins/unihome-plugin

# 4. Install Composer dependencies
composer install

# 5. Activate the plugin via WP Admin
# wp-admin > Plugins > UniHome > Activate

# 6. Run tests
vendor/bin/phpunit
```

### Development Workflow

```
1. Read STATUS.md              → Current phase and tasks
2. Read module documentation   → Your assigned module
3. Check docs/database.md      → Table schemas
4. Read docs/architecture.md   → Layer pattern
5. Implement                   → Code within boundaries
6. Run tests                   → vendor/bin/phpunit
7. Update STATUS.md            → Mark task progress
```

---

## Project Structure

```
unihome/
├── CLAUDE.md                  # AI agent instructions
├── STATUS.md                  # Current project state
├── README.md                  # This file
├── CONTRIBUTING.md            # PR template, commit format (TBD)
│
├── docs/
│   ├── architecture.md        # System architecture
│   ├── database.md            # Table schemas
│   ├── modules.md             # Module boundaries
│   ├── api.md                 # REST API catalog
│   ├── wordpress-development-guide.md
│   ├── environment-setup.md
│   ├── testing.md
│   ├── security.md
│   ├── error-handling.md
│   ├── caching-strategy.md
│   ├── deployment.md
│   ├── roles-and-capabilities.md
│   ├── decision-log.md
│   └── history/               # Phase history (append-only)
│
├── wp-content/
│   ├── themes/unihome-theme/  # Presentation layer
│   └── plugins/unihome-plugin/# Business logic
│       ├── unihome.php        # Plugin entry point
│       ├── composer.json      # Autoloader config
│       ├── includes/
│       │   ├── modules/       # Business modules
│       │   ├── shared/        # Cross-cutting services
│       │   └── database/      # Schemas and migrations
│       ├── api/               # REST route registration
│       ├── admin/             # WP admin pages
│       ├── assets/            # JS/CSS
│       ├── templates/         # Reusable template parts
│       └── tests/             # PHPUnit tests
```

---

## Documentation Index

### For Developers

| Document | Purpose |
|---|---|
| [STATUS.md](STATUS.md) | Current phase, tasks, blockers |
| [docs/architecture.md](docs/architecture.md) | System layers, module dependencies, data flow |
| [docs/database.md](docs/database.md) | All table schemas, migrations, deletion policies |
| [docs/modules.md](docs/modules.md) | Module boundaries, ownership, interfaces |
| [docs/api.md](docs/api.md) | REST API endpoint catalog |
| [docs/environment-setup.md](docs/environment-setup.md) | Local dev environment setup |
| [docs/testing.md](docs/testing.md) | How to write and run tests |
| [docs/security.md](docs/security.md) | Security checklist and patterns |
| [docs/error-handling.md](docs/error-handling.md) | Error conventions and logging |
| [docs/caching-strategy.md](docs/caching-strategy.md) | Cache keys, TTLs, invalidation |
| [docs/deployment.md](docs/deployment.md) | Deploy process and rollback |
| [docs/roles-and-capabilities.md](docs/roles-and-capabilities.md) | Role/capability matrix |
| [docs/decision-log.md](docs/decision-log.md) | Architecture Decision Records |
| [docs/wordpress-development-guide.md](docs/wordpress-development-guide.md) | WordPress conventions |

### Phase History & Specifications

| Document | Phase | Focus |
|---|---|---|
| [docs/history/phase-0-foundation.md](docs/history/phase-0-foundation.md) | Phase 0 | Foundation & Developer Infrastructure |
| [docs/history/phase-1-room-marketplace.md](docs/history/phase-1-room-marketplace.md) | Phase 1 | Room Marketplace & Media Uploads |
| [docs/history/phase-2-verification.md](docs/history/phase-2-verification.md) | Phase 2 | Trust System & Verification State Machine |
| [docs/history/phase-3-roommate.md](docs/history/phase-3-roommate.md) | Phase 3 | Compatibility Algorithm & Roommate Matching |
| [docs/history/phase-4-communication.md](docs/history/phase-4-communication.md) | Phase 4 | In-app Messaging & Behavioral Signals |
| [docs/history/phase-5-recommendation.md](docs/history/phase-5-recommendation.md) | Phase 5 | Multi-signal Recommendation Engine |
| [docs/history/phase-6-monetization.md](docs/history/phase-6-monetization.md) | Phase 6 | Listing Packages & Payment Gateway |
| [docs/history/phase-7-ecosystem-marketplace.md](docs/history/phase-7-ecosystem-marketplace.md) | Phase 7 | Second-hand Goods (Future Scope) |
| [docs/history/phase-8-advertisement.md](docs/history/phase-8-advertisement.md) | Phase 8 | Campus Ad Zones & Impressions (Future Scope) |

### For AI Coding Agents

| Document | Purpose |
|---|---|
| [CLAUDE.md](CLAUDE.md) | **Read first.** Project constraints, patterns, file locations |
| [STATUS.md](STATUS.md) | Current phase context |
| [docs/architecture.md](docs/architecture.md) | Layer boundaries |
| [docs/decision-log.md](docs/decision-log.md) | Decisions already made — do not contradict |

---

## Team

| Role | Count |
|---|---|
| Technical Lead | 1 |
| Developers | 4 |
| AI Coding Agents | Antigravity / Claude / Cursor |

---

## Architecture Principle

> WordPress = Application Foundation  
> UniHome Plugin = Business Application  
> Custom Tables = Business Data  
> Controller → Service → Repository = Module Pattern

See [docs/architecture.md](docs/architecture.md) for full details.

---

## License

TBD
