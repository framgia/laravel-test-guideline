# Testing Validation Rules.

Có 2 cách thực hiện việc test với Validation Rules:

1. Kiểm tra xem những rules cần thiết đã được set hay chưa
2. Kiểm tra nhưng data không hợp lệ thì không thể pass qua được validation

Trường hợp trước thì rất là đơn giản, và không yêu cầu những test cases phức tạp, trường hợp sau thì lại cần rất nhiều data để test. Những test cases mới nên được bổ sung liên tục mỗi khi có sự xuất hiện của dữ liệu không hợp lệ phát sinh trong quá trình user testing hoặc sử dụng trên production.

## Guide

### Testing validation rules

Validation rules được set qua 2 cách thông thường: được sử dụng trong app logic (thông qua hàm `validate()` của controller, hay `Validator` facade ...) hoặc qua Form Requests.

Trường hợp đầu tiên cần phải được test thông qua mocking validator instance, và chạy test trong những class bao gồm logic.

```php
public function test_store_method()
{
    $c = new Controller();
    $v = m::mock(\Illuminate\Contracts\Validation\Factory::class);

    // Substitute Validator instance in DI container.
    $previous = $this->app->make(\Illuminate\Contracts\Validation\Factory::class);
    $this->app->bind(\Illuminate\Contracts\Validation\Factory::class, $v);

    $r = new Request();
    $request->headers->set('content-type', 'application/json');
    $data = [
        'name' => 'Jonh',
        'email' => 'jonh@example.com',
    ];
    $request->setJson(new ParameterBag($data));

    $v->expects('make')->once()
        ->with($data, [
            'name' => 'required',
            'email' => 'required|email',
        ], m::any(), m::any())
        ->andReturnUsing(function (...$args) { return $previous->make(...$args); });

    $c->store($request);

    // Additional assertions.
}
```

Trong trường hợp dùng Form Requests, mọi thứ có thể được hoàn thành dễ dàng hơn:

```php
public function test_it_contains_valid_rules()
{
    $r = new StoreRequest();

    $this->assertEquals([
        'name' => 'required',
        'email' => 'required|email',
    ], $r->rules());
}
```

### Testing abnormal data

Việc đoán trước toàn bộ các abnormal data là hoàn toàn không thể, tuy nhiên, mỗi khi những trường hợp như thế xuất hiện, cần phải bổ sung thêm những test case mới.

1. Abnormal data được phát hiện trong quá trình sử dụng application.
2. Thêm vào những test case cho controller hay các instance khác với abnormal data để mô phỏng lỗi.
3. Thêm vào bản fix và test case với những lỗi được dự đoán từ trước.

### Testing custom validation rules

Nếu trong application của bạn có những custom validations, bạn cần phải thêm vào những test cases cho các rules đó.

Những tests này không yêu cầu kiểm tra user input, mà cần phải kiểm tra validation logic để chắc chắn rằng nó fails hoặc passes trong những trường hợp được dự báo trước.

Thật không may là không có một method nào để extract extensions từ validation factory, bởi thế cần phải đọc ra protected property.

```php
// AppServiceProvider::boot
Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
    return $value == 'foo';
});

// AppServiceProviderTest
public function test_validator_foo_rule()
{
    // Extract extensions from validation factory.
    $v = $this->app['validator'];
    $r = new ReflectionClass($v);
    $p = $r->getProperty('extensions');
    $p->setAccessible(true);
    $extensions = $p->getValue($v);

    // Check if extension had been registered properly.
    $this->assertArrayHasKey('foo', $extensions);
    $rule = $extensions['foo'];

    // Test cases for extension.
    $this->assertTrue($rule('attr', 'foo'));
    $this->assertFalse($rule('attr', 'bar'));
}
```

Class-based rules (Laravel 5.5) có thể được test một cách đơn giản, giống những class bình thường khác với những rules cơ bản cho code coverage.

