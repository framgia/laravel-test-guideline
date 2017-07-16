# Knowledge about Unit Testing

## Classify

Before creating of any test cases, we should determine input values of **particular function/method** which need to be test. The test cases should be designed to cover all combinations of input values and preconditions. Basically, we usually divide types of test case by 3 types of input dataset for unit testing.

- **Normal**: Inputs is normal range values (accepted). A large amount of code can be covered just by running normal test cases.
- **Boundary**: Inputs is at or just beyond its maximum or minimum limits. It is used to identify errors at boundaries rather than finding those exist in center of input domain.
- **Abnormal**: Inputs is invalid (illegal or not expected) to test error handling and recovery.

For example: *Suppose we have a function which validate email address entered by user. The maximum length of an email address is 50 characters.*

```php
function validate($email) {
    if (filter_var($email, FILTER_VALIDATE_EMAIL) && strlen($email) <= 50) {
        return true;
    }
    return false;
}

```

we should write the test cases as below:

<details>
    <summary>Normal cases</summary>

```php
public function test_valid_email_format_and_length()
{
    // Email with length 18 (less than: maximum - 1)
    $email = 'sample@framgia.com';
    $this->assertEquals(true, validate($email));
}
```

</details>

<details>
    <summary>Boundary cases</summary>

```php
public function test_valid_email_format_and_length_max_minus()
{
    // Email with length 49 (maximum - 1)
    $email = 'samplesamplesamplesamplesamplesamples@framgia.com';
    $this->assertEquals(true, validate($email));
}

public function test_valid_email_format_and_length_max()
{
    // Email with length 50 (equal maximum)
    $email = 'samplesamplesamplesamplesamplesamplesa@framgia.com';
    $this->assertEquals(true, validate($email));
}

public function test_valid_email_format_and_length_max_plus()
{
    // Email with length 51 (maximum + 1)
    $email = 'samplesamplesamplesamplesamplesamplesam@framgia.com';
    $this->assertEquals(false, validate($email));
}
```

</details>

<details>
    <summary>Abnormal cases</summary>

```php
public function test_invalid_email_format()
{
    // Invalid email format with normal length (between 0 ~ 50)
    $email = 'framgia.com';
    $this->assertEquals(false, validate($email));
}

public function test_valid_email_format_and_length_exceeded()
{
    // Email with length 54
    $email = 'samplesamplesamplesamplesamplesamplesample@framgia.com';
    $this->assertEquals(false, validate($email));
}
```

</details>

## Test Doubles
Một trong những yêu cầu cơ bản của Unit Test đó là tính **cô lập** (**isolation**). Nhìn chung thì tính cô lập là rất khó (nếu không muốn nói là không thể) bởi luôn luôn có rất nhiều dependencies trong cả project.

Vì thế, khái niệm về `Test Doubles` ra đời. Một `Test Double` cho phép chúng ta loại bỏ dependency nguyên bản, từ đó giúp cô lập unit.

Dưới đây là một vài loại `Test Doubles`

***Một vài phần trong các định nghĩa sau được lấy từ bài viết [Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html) trên blog của Martin Fowler***

### Dummies
- Dummy là objects được truyền vào nhưng mà không hề được sử dụng. Chúng thường chỉ được dùng để hoàn thành danh sách parameter.

### Fake
- Fake objects thực ra có mang những triển khai logic, thế nhưng thường sử dụng những "lối tắt", khiến chúng không thích hợp để triển khai trên production (Ví dụ như in memory database)

### Stubs
- Stubs đưa ra những câu trả lời có sẵn cho các lời gọi hàm được thực hiện trong quá trình test, và thường sẽ không trả về bất cứ cái gì ngoài những thứ mà chúng đã được lập trình trong bài test.

### Mocks
- Mocks là objects đã được lập trình trước với các expectations, tạo ra một đặc tả cho lời gọi mà chúng dự kiến sẽ nhận được.

### Stubs vs Mocks
- Stub giúp chúng ta chạy test. Mock là object thực hiện test.
- Một Fake mà bạn dùng để kiểm tra lời gọi hàm thì là Mock. Nếu không nó là Stub.
- Stub không bao giờ có thể làm cho test fail. Mock thì có thể.

## Examples
- Dưới đây là các PHP Mocking Frameworks mà bạn có thể sử dụng để dễ dàng tạo ra Mocks phục vụ việc test:
    - Mockery: Được khuyến khích sử dụng. Đã được tích hợp ngay trong Laravel Project. Tham khảo tài liệu ở [đây](http://docs.mockery.io/)
    - Prophecy: Một phần của PHPSpec project, nhưng có thể được sử dụng độc lập bên ngoài PHPSpec. Xem thêm tại [đây](https://github.com/phpspec/prophecy)
- Ví dụ về việc tạo Stubs và Mocks với **Mockery**

// TODO
