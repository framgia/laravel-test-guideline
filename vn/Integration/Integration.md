# Integration Tests

Mục đích của Integration Test là để chắc chắn rằng tất cả các components làm việc với nhau như ta mong muốn.

Khi thực hiện kiểu test này, hãy nhớ rằng những thay đổi với data storage hay những thao tác với các service bên ngoài là có thể xảy ra.

Để có thể giải quyết một cách đúng đắn những tình huống như vậy, có thể bạn vẫn cần phải tạo các [Test Doubles](../Knowledge.md#test-doubles) cho một vài class.

Việc những Class nào cần đến Test Doubles thì phụ thuộc vào cách tiếp cận của từng team, và có thể khác biệt phụ thuộc vào độ lớn của application.

## Guide

Laravel cung cấp cho chúng ta 2 cách để thực hiện Integration Test: tạo HTTP requests đến routes, và browser-based testing với `laravel\dusk` package.

Việc viết HTTP test là CẦN THIẾT, trong khi browser test thì không có yêu cầu cụ thể.

Integration test PHẢI cover được TOÀN BỘ route của application, và phải tuân thủ các mục đích sau:

**HTTP**
- Authentication và Policy tests. Mỗi test case cần có những test assertion riêng biệt.
- Kiểm tra Status Codes cho từng loại response.
- Kiểm tra Redirect Codes và Paths cho các các sự kiện khác nhau.
- Kiểm tra tính đúng đắn của JSON responses.
- Kiểm tra Error Handlers cho các response.
- Kiểm tra tính đúng đắn của API versions (nếu cần thiết).

**Database**
- Đảm bảo data được ghi vào database một cách đúng đắn trong quá trình xử lý requests.
- Kiểm tra quá trình migration.
- Kiểm tra việc insert dữ liệu không bình thường thì có được xử lý chính xác hay không.

Nếu application của bạn có tương tác với những services hay servers bên ngoài, thì bạn cần sử dụng phiên bản Staging của những services đó, hoặc `mock` class Client của bạn.

Testing environment có thể được định nghĩa bởi những config và biến đặc biệt. Chặng hạn như ở `testing` database connection ta có thể tạo biến môi trường là `DB_TEST_DATABASE`. Tuy nhiên, ta nên làm theo cách là thay đổi giá trị của các biến production ở bên trong file `phpunit.xml` hoặc trong config CI.

Sẽ là tốt hơn nếu phần tests có thể được chạy hai lần, với `APP_ENV` được đặt giá trị lần lượt là `testing` và `production`.

```xml
<php>
    <env name="APP_ENV" value="testing"/>
    <env name="CACHE_DRIVER" value="array"/>
    <env name="SESSION_DRIVER" value="array"/>
    <env name="QUEUE_DRIVER" value="sync"/>
    <env name="DB_HOST" value="test.database.local"/>
    <env name="DB_USERNAME" value="testing"/>
    <env name="DB_PASSWORD" value="secret"/>
</php>
```
