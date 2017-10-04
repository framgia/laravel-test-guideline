# Laravel Testing Conventions

## Folder Structure
- All Unit Tests MUST be placed inside the `tests/Unit` folder
- All Integration Test MUST be placed inside the `tests/Feature` folder
- Contents of the `Unit` and `Feature` folders MUST have the same structure with the main `app` folder. For example, unit tests for file `app/Models/User.php` MUST be written in `tests/Unit/Models/UserTest.php`

## Naming Convention
- Each test file MUST have a specific namespace. The namespace MUST start with `Tests\` (or `ASampleProjectTests\`), then followed by the folder structure. For example, namespace for `tests/Unit/Models/UserTest.php` file MUST be `Tests\Unit\Models\UserTest` (or `ASampleProjectTests\Unit\Models\UserTest`)
- For better readability, use `snake_case` for test function's name. A test function has to start with `test`, so here is an example:
```
public function test_it_throws_an_exception_when_email_is_too_long()
{
}
```
- Non-test function can stick with PSR-2 rule, that uses `camelCase` for function name.

## Unit Test Required Components
- **Controllers**: with disabled events handling. All external components MUST be mocked.
- **Requests** (if present): test validation
- **Models**: getters, setters, additional features
- **Transformers / Presenters** (if present): test output results for different source sets
- **Repositories** (if present): test each method for creating correct SQL queries OR correct calls to mocked query builder
- **Event listeners**
- **Queue jobs**
- **Auth policies**
- And any additional project-specific classes

## Integration Test Required Components
- **Routes**: test input/output with integration in a whole system
- **Route authentication**

## Code Coverage
- Code Coverage for the entire project SHOULD be greater than `80%`
