# Testing Middleware Classes

Middleware tests cases nhìn chung không có gì khác biệt nhiều với hầu hết các class khác. Ngoại trừ việc bạn cần chú ý đến `$next` callback.

## Guide

Bạn nên nhớ rằng middleware có thể trả về data mà không cần gọi `$next`, và đó có thể là trường hợp không mong muốn.
Ta có các trường hợp xử lý với middleware như sau:

- Trả về kết quả của `$next` closure: basic *before* middleware
- Trả về response mà không gọi `$next`: abortion middleware.
- Nhận về response từ `$next` và thực hiện thay đổi trên đó: basic *after* middleware.

Dựa trên cách thực hiện của bạn mà những cases trên có thể được dùng cùng nhau. (ví dụ như conditional abortion hoặc before/after execution).

Để mock `$next` callback, bạn có thể đơn giản là tạo một closure với các assertions ở bên trong:

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

Đôi khi middleware cần tương tác với Session, Cache hoặc các components khác. Trong trường hợp đó, bạn cần mock những components này một cách chính xác.

Trong trường hợp đặc biệt cần đến chỉnh sửa session, bạn nên sử dụng `$request->getSession()` ở bên trong method, thay vì dùng Facades hoặc Dependecy Injection.

## Examples

* [Middlewares](https://github.com/framgia/laravel-test-examples/tree/master/app/Http/Middleware)
* [Middleware tests](https://github.com/framgia/laravel-test-examples/tree/master/tests/Unit/Http/Middleware)