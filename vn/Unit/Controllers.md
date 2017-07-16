# Testing Controllers

Controllers chứa những giải thuật chính cho vấn đề xử lý request. Chúng cần phải được chú ý nhiều trong quá trình testing.

Unit Tests viết cho controllers phải được cô lập hoàn toàn, và bất cứ lời gọi ra bên ngoài nào từ bên trong controller method phải được mock.

Tuỳ vào logic trong application của bạn, mà controllers có thể thực hiện việc Validation ở phần logic bên trong method hay thông qua Form Request.

Nếu là trường hợp trước, Controller Tests PHẢI bao gồm các cases liên quan đến **[boundary](../Knowledge.md#Classify)** và **[abnormal](../Knowledge.md#Classify)** request input. Ở trường hợp còn lại, những cases trên phải được test trong Form Request test cases tương ứng.

Tuy nhiên, điều này chỉ áp dụng với input được thực hiện bởi user. Data được cung cấp bởi những hàm bên ngoài controller method phải được mock chính xác với tất cả các loại data.

## Guide

## Examples

* [Resource web controller](https://github.com/framgia/laravel-test-examples/app/Http/Controllers/CityController.php) and [tests](https://github.com/framgia/laravel-test-examples/blob/master/tests/Unit/Http/Controllers/CityControllerTest.php)
