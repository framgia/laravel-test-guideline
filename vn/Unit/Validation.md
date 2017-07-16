# Testing Validation Rules.

There are 2 approaches for testing validation rules:
1. Check if necessary rules has been set
2. Check if invalid data cannot pass validation

While first is very simple and does not require any complicated test cases, the second might need a lot of data
to test. It is highly recommended to provide new test cases for every appearance of invalid data during user testing
or production use.

## Guide

### Testing validation rules

Validation rules are set in 2 common ways: various use inside app logic (via controller `validate()` method, `Validator`
facade, etc.) or via form requests.

First case must be tested via mocking validator instance and running test on the class which contains logic.

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

In case of form requests it can be done much simpler:

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

It is impossible to predict all cases of abnormal data, however when such cases appear, tests must be extended.

1. Abnormal data is detected during application use.
2. Implement test case for controller or other instance with abnormal data to emulate failure.
3. Implement fix and provide test case with expected interruption.

### Testing custom validation rules

If your application contains custom validations, you must include test cases for such rules in a separate test case.
These tests are required not to check user input, but to test validation logic itself and be sure that it fails or passes
in predicted cases.

Unfortunately, there is no available method to extract extensions from validation factory, so protected property should be read.

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

Class-based rules (Laravel 5.5) can be simply tested as regular classes with basic rules for code coverage.
