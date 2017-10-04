# Testing Transformers and Presenters

Hầu hết các project đều bao gồm serializations và presenters, theo cách này hay cách khác.

Cách thức mặc định mà Laravel cung cấp đó là sử dụng mutators/getters trong Model class, tuy nhiên trong những application phức tạp, ta nên sử dụng những cách khác.

Hướng dẫn dưới đây có thể được áp dụng cho những trường hợp chung dùng transformers hay presenters.

## Guide

Sử dụng transformers một cách thích đáng là rất cần thiết cho những application cung cấp API, và cũng rất khuyến khích sử dụng trong những application khác nữa.

Transformer test phải tuân thủ các mục tiêu sau:

1. Đảm bảo tính nhất quán của dữ liệu (object structure must not change between commit unexpected).
2. Đảm bảo proterty mutations được thực hiện chính xác.
3. Đảm bảo kiểu data khớp với đầu ra mong muốn.

Trong rất nhiều các API client applications, việc xuất hiện properties mới, hay mất một vài properties có thể sẽ dẫn đến hậu quả nghiêm trọng.
Do đó mọi thay đổi đến cấu trúc object phải luôn được kiểm soát.

Hãy chắc chắn rằng transformers của bạn luôn có dữ liệu nhất quán.

```php
public function test_output_contains_valid_structure()
{
    $u = new User([
        'name' => 'Jonh',
        'email' => 'jonh@example.com',
        'password' => '123456',
        'created_at' => Carbon::now(),
        'updated_at' => Carbon::now(),
    ])

    $u->setRelation('posts', [
        new Post([
            'title' => 'Test post',
            'created_at' => Carbon::now(),
            'updated_at' => Carbon::now(),
        ])
    ])

    $t = (new UserTransformer())->transform($user);

    $this->assertEquals([
        'name', 'email', 'created_at', 'updated_at', 'posts'
    ], array_keys($t));

    $this->assertTrue(is_array($t['posts']));

    // Additional checks.
}
```

Nếu application của bạn có implement Feature Test cho API, bạn có thể thay việc check này bằng cách sử dụng method `assertJsonStructure` bên trong Feature test case.

Với JSON output, cần phải check data type, đặc biệt nếu API Clients của bạn được viết bằng những ngôn ngữ dùng strict-type.

```php
public function test_data_types()
{
    $u = new User([
        'id' => '1',
        'name' => 'Jonh',
        'email' => 'jonh@example.com',
        'created_at' => Carbon::now(),
    ])

    $t = (new UserTransformer())->transform($user);

    $this->assertInternalType('int', $t['id']);
    $this->assertInternalType('string', $t['name']);
    $this->assertInternalType('string', $t['email']);
    $this->assertInternalType('array', $t['created_at']);
    $this->assertInternalType('string', $t['created_at']['date']);
    $this->assertInternalType('int', $t['created_at']['timezone_type']);
    $this->assertInternalType('string', $t['created_at']['timezone']);
}
```
