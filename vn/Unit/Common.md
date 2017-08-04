# Common Testing Rules

Với những thành phần như Events, Event Listeners, Subscribers, Queue Jobs và Validation Rule Classes, bạn nên tuân theo một số quy tắc cơ bản sau:

1. Test `__construct()` method để chắc chắn rằng các properties đều được assigned.
2. Cover tất cả các public methods với những tests cần thiết.
3. Thật chú ý vào **main method** của class (e. g. `handle` đối với Listeners hoặc `passes` với rules).
4. Coverage là không cần thiết đối với những methods mà trả về text (ví dụ như `message` đối với rules).
5. Coverage là không bắt buộc đối với những hàm get/set cơ bản, hàm mà không chỉnh sửa tham số input.

## Mocking DI instances

Laravel sử dụng injection trong rất nhiều trường hợp và components của bạn có thể sử dụng nó thông qua method/constructor injection hoặc trực tiếp trong methods thông qua container instance hoặc Facades.

Trong những trường hợp như vậy, sẽ là rất quan trọng để tạo Test Doubles cho từng injected instance và chắc chắn rằng không có instances nào khác được resolved.

Mặc định thì `TestCase` class được cung cấp với một application instance được khởi tạo sẵn bởi Laravel. Mọi thứ là tốt nếu bạn sử dụng cách tiếp cận này, tuy nhiên nên có thêm một class test case cơ bản với instance application được mock.

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

Cách tiếp cận này sẽ giúp bạn kiểm soát tất cả các quá trình Dependecies Injection một cách cẩn thận. Và giờ bạn có thể inject Test Doubles một cách đơn giản.

```php
public function test_it_assignes_event_handler()
{
    $e = m::mock(Dispatcher::class);
    $this->app->bind(Dispatcher::class, $e);
    // Test event assignment with mocked class.
}
```

## Testing Database Queries

Tính cô lập là rất quan trọng cho test cases, và việc chắc chắn rằng SQL queries được sinh ra chính xác là cực kỳ cần thiết.

Tất cả các thao tác với Database trong Laravel đều được thực hiện thông qua class `Illuminate\Database\Connection`, và các methods của nó thì dễ dàng được mock bằng cách sử dụng Mockery package.

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

Đôi khi, sẽ là rất tiện lợi khi ta inject một Connection đã được mock vào trong DI để có thể sử dụng nó ở mọi nơi.
Ở đây ta trực tiếp thay đổi property, bởi sự mở rộng mặc định thì yêu cầu mock connection methods, thứ mà chúng ta không cần.

```php
public function setUp()
{
    $connection = m::mock(Connection::class);
    // Replace SQLiteGrammar and Processor if necessary.
    $query = new Builder($connection, new SQLiteGrammar(), new Processor());
    $connection->allows()->table()->andReturnUsing(function ($table) use ($query) {
        return $query->from($table);
    })

    // Replace current default connection (if necessary)
    $this->afterApplicationCreated(function () use ($connection) {
        $manager = $app['db'];
        $name = $manager->getDefaultConnection();
        $manager->purge($name);

        $r = new ReflectionClass($manager);
        $p = $r->getProperty('connections');
        $p->setAccessible(true);
        $p->setValue($manager, [
            $name => $connection,
        ])
    });

    // Assign a separate "mock" connection
    $this->afterApplicationCreated(function () use ($connection) {
        $manager = $app['db'];
        $manager->setDefaultConnection('mock');

        $r = new ReflectionClass($manager);
        $p = $r->getProperty('connections');
        $p->setAccessible(true);
        $list = $p->getValue($manager);
        $list['mock'] = $connection;
        $p->setValue($manager, $list);
    });


    parent::setUp();
}
```

Bạn cần phải nhớ tuân theo những thứ sau khi thực hiện database testing
- Bạn có thể có connection có tên (named connection), hãy cẩn thận với nó.
- Models lấy ra connections thông qua `ConnectionResolverInterface`, thứ được tự gán vào trong model và có thể khác biệt.
- Queries được tạo ra khác nhau với những cú pháp khác nhau (db khác nhau). Hãy sử dụng cái thích hợp cho project của bạn.

