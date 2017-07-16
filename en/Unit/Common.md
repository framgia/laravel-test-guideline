# Common Testing Rules

With most components like events, event listeners, subscribers, queue jobs and validation rule classes, you should follow
the same basic rules:

1. Test `__construct()` method to make sure that properties are assigned.
2. Cover all public methods with necessary tests.
3. Pay the most attention to the main method (e. g. `handle` for listeners or `passes` for rules).
4. Coverage is not necessary for methods, which return text (e. g. `message` for rules).
5. Coverage is optional for basic get/set methods, which don't modify input arguments.

## Mocking DI instances

Laravel uses injections in various cases and your components might use it via method/constructor injection
or directly inside methods via container instance or facades.

For such cases it is very important to create test doubles for every injected instance and make sure that
no additional instances are resolved.


By default, `TestCase` class provided with Laravel automatically instantiates application instance. It is good
to use this approach, but highly recommended to have an additional basic test case with mocked app.

```php
class MockApplicationTestCase extends TestCase
{
    /**
     * Creates the application.
     *
     * @return \Illuminate\Foundation\Application|\Mockery\Mock
     */
    public function createApplication()
    {
        return m::mock(\Illuminate\Foundation\Application::class)->makePartial();
    }
}
```

This approach will help you to control all DI process carefully. Now you can inject test doubles simply.

```php
public function test_it_assignes_event_handler()
{
    $e = m::mock(Dispatcher::class);
    $this->app->bind(Dispatcher::class, $e);
    // Test event assignment with mocked class.
}
```

## Testing Database Queries

Isolation is very important for test cases and it is critical to make sure that SQL queries are built properly.

All database interactions in Laravel are made through `Illuminate\Database\Connection` class, which methods can be easily
mocked with Mockery package.

```php
$connection = m::mock(Connection::class);
$query = new Builder($connection, new SQLiteGrammar(), new Processor());
// DB::table is the most common call and it is simpler to mock the method manually instead of dealing with partial mocks.
$connection->allows()->table()->andReturnUsing(function ($table) use ($query) {
    return $query->from($table);
})

// Test your queries

$connection->shouldReceive('select')->with(
    'select * from "streets" limit 10 offset 0', // query
    [], // bindings
    m::any() // useReadPdo. use true/false if necessary
)->andReturn([
    // 'query result'
]);

$someClass->methodUsingConnection();
```

Sometimes it is handy to inject mocked connection into DI and have it available everywhere.
Direct property modification is used here, because default extension requires mocking of connection methods,
which we don't need.

```php
protected $db;

public function setUp()
{
    $this->afterApplicationCreated(function () {
        $this->db = m::mock(
            Connection::class.'[select,update,insert,delete]',
            [m::mock(\PDO::class)]
        );

        $manager = $this->app['db'];
        $manager->setDefaultConnection('mock');

        $r = new \ReflectionClass($manager);
        $p = $r->getProperty('connections');
        $p->setAccessible(true);
        $list = $p->getValue($manager);
        $list['mock'] = $this->db;
        $p->setValue($manager, $list);
    });

    parent::setUp();
}
```

You should remember following things for database testing:

- You may have named connections, be careful with that.
- Models get connections via `ConnectionResolverInterface` which is assigned in the model itself and might differ.
- Queries are created differently using different grammars. Use the one suitable for your project.

## Examples

* [Laravel event](https://github.com/framgia/laravel-test-examples/tree/master/app/Events) and [tests](https://github.com/framgia/laravel-test-examples/tree/master/tests/Unit/Events)
* [Listeners](https://github.com/framgia/laravel-test-examples/tree/master/app/Listeners) and [tests](https://github.com/framgia/laravel-test-examples/tree/master/tests/Unit/Listeners)
* [Database mock setup](https://github.com/framgia/laravel-test-examples/blob/master/tests/Unit/Http/Controllers/CityControllerTest.php#L24) and [usage](https://github.com/framgia/laravel-test-examples/blob/master/tests/Unit/Http/Controllers/CityControllerTest.php#L76)
