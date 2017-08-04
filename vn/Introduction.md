# Giới thiệu về Laravel Testing

## Test Case
- Test Case là một thuật ngữ phổ biến thường dùng trong các bài Test cụ thể. Nó thường là đơn vị nhỏ nhất của Testing.
- Một Test Case sẽ bao gồm các thông tin như requirements testing (các inputs, điều kiện thực thi), test steps, verification steps, prerequisites, outputs, test environment ...

## Unit Test
- Unit Testing là một phương pháp kiểm thử phần mềm mà ở đó từng **đơn vị riêng lẻ** (Individual Unit) của source code được test để kiểm tra xem chúng có đảm bảo chất lượng để sử dụng không.
- Ở trong Laravel Project, viết Unit Test là quá trình mà ở đó, tất cả các Test Case cho từng **function/method** riêng biệt được viết.
- Unit Test cần được thiết kế để kiểm tra function một cách **độc lập**. Hay nói cách khác: **`A` cần `B` để chạy. Ngay cả khi `B` có lỗi, thì Unit Test của `A` VẪN SẼ PASS nếu không có vấn đề gì với `A`**
- Một Unit Test tốt **KHÔNG NÊN** thực hiện những việc như sau:
    - Trigger để chạy codes từ những functions khác trong project
    - Truy vấn vào cơ sở dữ liệu
    - Sử dụng file system
    - Truy cập mạng
    - ...

## Integration Test
- Integration Testing là công đoạn trong kiểm thử phần mềm mà ở đó, từng đơn vị phần mềm riêng biệt được kết hợp lại, và kiểm tra như một khối thống nhất.
- Nếu như Unit Test giúp chúng ta chắc chắn rằng từng function riêng biệt hoạt động chính xác, thì Integration Test giúp chúng ta chắc chắn rằng các thành phần khác trong của project sẽ hoạt động hoàn hảo khi chúng kết hợp với nhau trong thực tế.

## PHPUnit
- **PHPUnit** là một Testing Framework hướng đến lập trình viên cho PHP. Nó là một phiên bản của kiến trúc xUnit dành cho Unit Testing Frameworks.
- Các bạn có thể tham khảo [hướng dẫn cài đặt](https://phpunit.de/getting-started.html)

### Cài đặt theo từng project
```
$ composer require --dev phpunit/phpunit
```

- PHPUnit được tích hợp sẵn trong Laravel Laravel project. Bạn có thể bắt đầu Test app của mình bằng câu lệnh
```
$ ./vendor/bin/phpunit
```

> Cài đặt và sử dụng theo từng Project được khuyến khích sử dụng, bởi bạn có thể sẽ cần những version khác nhau của PHPUnit trong những project khác nhau.

### Cài đặt Global

- Thông qua download:
```
$ wget https://phar.phpunit.de/phpunit.phar
$ chmod +x phpunit.phar
$ sudo mv phpunit.phar /usr/local/bin/phpunit
```
- Thông qua composer:
```
$ composer global require phpunit/phpunit
```

Hãy chắc chắn rằng bạn có `/home/<user>/.composer/vendor/bin` hoặc `c:\Users<user>\AppData\Roaming\Composer\vendor\bin` được add vào trong biến `PATH` của bạn.

- Sau đó, bạn sẽ có thể chạy bản PHPUnit đã được cài đặt global ở bất cứ nơi đâu:
```
$ phpunit
```

## Code Coverage
- **Code coverage** là một phương pháp đánh giá được dùng để mô tả mức độ mà source code của một chương trình đã được thực thi, khi mà một bộ Test cụ thể chạy. Nói một cách khác, **Code coverage** là một cách để đảm bảo rằng Tests của bạn thực sự đang test Codes của bạn!
- Công thức tính Code coverage:
```
Code Coverage = (Tổng số dòng Code được gọi bởi các bài Tests của bạn) / (Tổng số dòng Code trong thực tế) x 100%
```

Ví dụ: Nếu code coverage của bạn là 90%, điều đó có nghĩa là 90% các dòng codes trong project của bạn đã được gọi ghi chạy Test.
- Code Coverage có thể được tạo ra bằng PHPUnit với extension with **Xdebug** được kích hoạt. Bởi vậy, hãy chắc chắn rằng bạn đã cài đặt và bật Xdebug lên. Tham khảo [Xdebug Installation Guide](https://xdebug.org/docs/install)
- Bạn có thể chạy PHPUnit để tạo coverage report ở định đạng HTML theo câu lệnh sau:
```
phpunit --coverage-html <dir>

// Hoặc tạo report với định dạng Clover XML format.
phpunit --coverage-clover <file>
```
- Code Coverage thực sự là một công cụ rất hữu ích để tìm kiếm những thành phần chưa được tests trong project của bạn. Tuy nhiên, nó không phải là một con số vạn năng để có thể đảm bảo cho chất lượng Tests của bạn.

## Tài liệu tham khảo thêm
- [PHPUnit Manual](https://phpunit.de/manual/current/en/phpunit-book.pdf)
- [Laravel Testing Official Documents](https://laravel.com/docs/master/testing)
- [Laravel Testing Decoded](https://leanpub.com/laravel-testing-decoded)
- Laracast's [Testing Jargon](https://laracasts.com/series/testing-jargon)
- Laracast's [Testing Laravel](https://laracasts.com/series/phpunit-testing-in-laravel)
- Laracast's [Intuitive Integration Testing](https://laracasts.com/series/intuitive-integration-testing)
