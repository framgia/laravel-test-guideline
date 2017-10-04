# Laravel Testing Conventions

## Cấu trúc thư mục
- Tất cả Unit Tests PHẢI được đặt trong thư mục `tests/Unit`
- Tất cả Integration Tests PHẢI được đặt trong thư mục `tests/Feature`
- Nội dung bên trong thư mục `Unit` hay `Feature` PHẢI có cấu trúc giống với cấu trúc bên trong thư mục `app`. Ví dụ như Unit Test cho file `app/Models/User.php` PHẢI được viết bên trong file `tests/Unit/Models/UserTest.php`

## Quy tắc đặt tên
- Toàn bộ test file PHẢI có namespace riêng. Namespace PHẢI được bắt đầu với `Tests\` (hoặc `ASampleProjectTests\`), và tiếp đó là cấu trúc thư mục. Ví dụ, namespace cho file `tests\Unit\Models\UserTest.php` PHẢI là `Tests\Unit\Models\UserTest` (hoặc `ASampleProjectTests\Unit\Models\UserTest`)
- Để đảm bảo tính dễ đọc, nên sử dụng `snake_case` khi viết tên cho các hàm test. Do một hàm test phải bắt đầu bằng `test`, thế nên sau đây là một ví dụ về tên hàm:
```
public function test_it_throws_an_exception_when_email_is_too_long()
{
}
```
- Những hàm không làm nhiệm vụ test có thể tuân theo quy tắc đặt tên của PSR-2, tức dùng `camelCase` để đặt tên.

## Những thành phần cần viết Unit Test
- **Controllers**: với events handling được disable. Toàn bộ các thành phần bên ngoài PHẢI được mock.
- **Requests** (nếu có): Kiểm tra validation
- **Models**: getters, setters, và những chức năng khác
- **Transformers / Presenters** (nếu có): Kiểm tra kết quả output cho những dữ liệu khác nhau
- **Repositories** (nếu có): Kiểm tra từng hàm có tạo ra đúng SQL queries hay không, hay có các lời gọi hàm, đến mocked query builder, đúng hay không
- **Event listeners**
- **Queue jobs**
- **Auth policies**
- Và các Class chuyên biệt khác trong project

## Những thành phần yêu cầu Integration Test
- **Routes**: Kiểm tra đầu vào, đầu ra với toàn bộ hệ thống
- **Route authentication**

## Code Coverage
- Code Coverage cho toàn bộ project nên đạt trên `80%`
