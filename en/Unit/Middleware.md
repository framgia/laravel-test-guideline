# Testing Middleware Classes

Middleware test cases are not different from most other classes. Except you need to pay more attention to `$next` callback.

## Guide

You should remember that middleware might return data without calling `$next` and that might be an unexpected case.
There are X cases of middleware execution:

- Returns `$next` closure result: basic *before* middleware.
- Returns response without calling `$next`: abortion middleware.
- Receives response from `$next` and modifies it: basic *after* middleware.

Depending on your implementation, these cases might be mixed (e. g. conditional abortion or before/after execution).

In order to mock `$next` callback, you can simply create a closure with assertions within test case:

```php
public function test_middleware_appends_header()
{
    //...
    $next = function ($request) {
        $this->assertEquals('Appended_header_data', $request->headers->get('header-name'));
        return response('ok');
    }

    $middleware->handle($request, $next);
    //...
}
```

Sometimes middleware might interact with session, cache or other components. In this case you must mock those components
properly.

In the special case of session modifications, it is highly recommended to use `$request->getSession()` in method body
instead of facades or dependency injection.

## Examples

* [Middlewares](https://github.com/framgia/laravel-test-examples/tree/master/app/Http/Middleware)
* [Middleware tests](https://github.com/framgia/laravel-test-examples/tree/master/tests/Unit/Http/Middleware)