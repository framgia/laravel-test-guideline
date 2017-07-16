# Knowledge about Unit Testing

## Classify

Trước khi tạo bất cứ Test Cases nào, chúng ta nên xác định rõ giá trị đầu vào của **từng function/method** cần được test. Các Test Cases phải được thiết kế để có thể cover được hết các sự kết hợp của các giá trị inputs cùng các điều kiện.

Nhìn chung, chúng ta thường chia test case ra làm 3 loại dựa trên dữ liệu inputs cho Unit Test.

- **Normal**: Inputs thuộc vào dải dữ liệu bình thường (accepted). Một lượng lớn codes có thể được cover bằng cách chỉ cần chạy **normal** test cases.
- **Boundary**: Inputs bằng hoặc xấp xỉ giá trị maximum hay minimum. Chúng được sử dụng để phát hiện lỗi tại cận, thay vì tìm kiếm lỗi tại những vị trí ở giữa trong dải input.
- **Abnormal**: Inputs là không hợp lệ hay không được kỳ vọng, dùng để kiểm tra khả năng handle lỗi.

Ví dụ: *Giả sử như chúng ta có một function để kiểm tra địa chỉ email nhập vào từ user. Độ dài tối đa của email là 50 ký tự.*

```php
function validate($email) {
    if (filter_var($email, FILTER_VALIDATE_EMAIL) && strlen($email) <= 50) {
        return true;
    }
    return false;
}

```

Chúng ta nên viết các Test Cases như sau:

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
