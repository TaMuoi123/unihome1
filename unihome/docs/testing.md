# UniHome — Testing Strategy

> **Source:** DOCUMENTATION_PROPOSAL.md v0.3.0 §7  
> **Maintained By:** Technical Lead  
> **Last Updated:** 2026-09-30

---

## 1. Test Types

| Type | Tool | Location | Tests What |
|---|---|---|---|
| **Unit tests** | PHPUnit | `tests/modules/{mod}/test-{class}.php` | Service classes — business logic in isolation |
| **Integration tests** | WP_UnitTestCase | `tests/modules/{mod}/test-{class}.php` | Repository classes — SQL against test DB |
| **Shared tests** | PHPUnit | `tests/shared/test-{class}.php` | Validator, NotificationService, EventTracker |

---

## 2. Setup

### phpunit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit
    bootstrap="tests/bootstrap.php"
    colors="true"
    beStrictAboutTestsThatDoNotTestAnything="true"
>
    <testsuites>
        <testsuite name="UniHome">
            <directory>tests/</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

### tests/bootstrap.php

```php
<?php
/**
 * PHPUnit bootstrap for UniHome plugin tests.
 * Loads the WordPress test suite and the plugin.
 */

// Path to WordPress test suite
$_tests_dir = getenv( 'WP_TESTS_DIR' ) ?: '/tmp/wordpress-tests-lib';

// Load the WordPress test suite
require_once $_tests_dir . '/includes/functions.php';

// Load the plugin
tests_add_filter( 'muplugins_loaded', function() {
    require dirname( __DIR__ ) . '/unihome.php';
} );

require $_tests_dir . '/includes/bootstrap.php';
```

### Running Tests

```bash
# From plugin root
cd wp-content/plugins/unihome-plugin

# Run all tests
vendor/bin/phpunit

# Run specific test file
vendor/bin/phpunit tests/modules/rooms/test-room-service.php

# Run specific test method
vendor/bin/phpunit --filter test_create_room_with_valid_data

# Run with coverage (requires Xdebug)
vendor/bin/phpunit --coverage-text
```

---

## 3. Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Test file | `test-{class-name}.php` | `test-room-service.php` |
| Test class | `Test_{Class_Name}` | `Test_Room_Service` |
| Test method | `test_{scenario}` | `test_create_room_with_valid_data` |
| Test extends | `WP_UnitTestCase` | (for integration tests with DB) |

---

## 4. Test Structure

```php
<?php
namespace UniHome\Tests\Modules\Rooms;

use WP_UnitTestCase;
use UniHome\Modules\Rooms\RoomService;
use UniHome\Modules\Rooms\RoomRepository;

class Test_Room_Service extends WP_UnitTestCase {

    private RoomService $service;

    public function setUp(): void {
        parent::setUp();
        $this->service = new RoomService( new RoomRepository() );
    }

    public function tearDown(): void {
        parent::tearDown();
        // Clean up test data if needed
    }

    /**
     * @test
     */
    public function test_create_room_with_valid_data(): void {
        $user_id = $this->factory->user->create( [ 'role' => 'landlord' ] );
        wp_set_current_user( $user_id );

        $data = [
            'title'     => 'Test Room',
            'price'     => 5000000,
            'room_type' => 'single',
            'address'   => '123 Test Street',
        ];

        $result = $this->service->create_room( $data );

        $this->assertIsArray( $result );
        $this->assertEquals( 'Test Room', $result['title'] );
        $this->assertEquals( 'draft', $result['status'] );
    }

    /**
     * @test
     */
    public function test_create_room_without_title_returns_error(): void {
        $data = [ 'price' => 5000000 ];

        $result = $this->service->create_room( $data );

        $this->assertWPError( $result );
        $this->assertEquals( 'missing_title', $result->get_error_code() );
    }

    /**
     * @test
     */
    public function test_status_transition_draft_to_active(): void {
        // ... test valid state transition
    }

    /**
     * @test
     */
    public function test_invalid_status_transition_returns_error(): void {
        // ... test invalid state transition
    }
}
```

---

## 5. Rules

1. **Every Service class public method must have at least one test.**
2. **AI-generated code must include tests** or it is rejected in review.
3. **Tests must not depend on external services.** Mock Google Maps, Gemini API, payment gateways.
4. **Tests must be independent.** Each test sets up its own data and cleans up after.
5. **Use `$this->factory->user->create()`** for test users (WordPress test factory).
6. **Use `$this->assertWPError()`** for checking `WP_Error` returns.
7. **Run all tests before submitting a PR.** No PRs with failing tests.

---

## 6. What to Test

| Layer | Test Focus | Priority |
|---|---|---|
| Service | Business rules, state transitions, validation logic | **High** |
| Repository | SQL correctness, query filters, edge cases | **Medium** |
| Shared (Validator) | Input validation rules | **High** |
| Shared (NotificationService) | Event routing, preference checking | **Medium** |
| Controller | Not directly tested (tested via integration/manual) | Low |

---

## 7. Mocking External Dependencies

```php
// Mock the Gemini API client
$mock_client = $this->createMock( GeminiClient::class );
$mock_client->method( 'get_recommendation' )
            ->willReturn( [ 'score' => 85.5 ] );

$service = new RecommendationService( $mock_client );
```

---

## 8. CI Integration (Future)

When CI is configured:
- Tests run on every PR to `develop` and `main`.
- PRs with failing tests cannot be merged.
- Coverage reports generated on `main` branch pushes.
