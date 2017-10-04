# Testing Models

Models là một trong những class được sử dụng nhiều nhất trong Laravel Application. Cùng với đó, chúng là thành phần phức tạp nhất trong số các default components.

Tuỳ vào cách bạn thực hiện, mà models có thể bao gồm validation, external services connection (e. g. ElasticSearch indexing), hay property getters và nhiều custom methods khác.

Do models được sử dụng ở khắp nơi trong application, nên chúng phải được test một cách cẩn thận.

## Guide

### Configuration properties

Mọi Model đều bao gồm một vài các thiết lập cơ bản với các public hoặc protected properties. Những properties này có thể ảnh hưởng rất nhiều đến cách thức hoạt động của application, tuy nhiên chúng là là những thành phần dễ test nhất.

Bạn có thể assert giá trị trả về của property getters:

```php
public function test_contains_valid_fillable_properties()
{
    $m = new User();
    $this->assertEquals(['name', 'email'], $m->getFillable());
}
```

Những method tests sau cần phải được sử dụng trong quá trình tests:

- `$fillable` -> `getFillable()`
- `$guarded` -> `getGuarded()`
- `$table` -> `getTable()`
- `$primaryKey` -> `getKeyName()`
- `$connection` -> `getConnectionName()`: Trong trường hợp bạn sử dụng multiple connections.
- `$hidden` -> `getHidden()`
- `$visible` -> `getVisible()`
- `$casts` -> `getCasts()`: note that method appends incrementing key.
- `$dates` -> `getDates()`: note that method appends `[static::CREATED_AT, static::UPDATED_AT]`.
- `newCollection()`: kiểm tra collection có đúng kiểu dữ liệu hay không. Sử dụng `assertEquals` với kết quả trả về từ hàm `get_class()`, chứ không dùng `assertInstanceOf`.

Chúng ta có thể đặt hết tất cả những kiểm tra trên bên trong một test method, chẳng hạn như `test_model_configuration()`

### Relations

Relations được định nghĩa là các methods trả về class relation tương ứng. Trong nhiều trường hợp, relation có thể được định nghĩa với custom foreign keys, hoặc với những câu query thêm vào. Cách tốt nhất để test relation config là assert instance `Relation` được trả về.

Trong tất cả các trường hợp bình thường, các relations được tạo ra với `$model->newQuery()` như là builder instance. Nếu không được chỉnh sửa gì, nó có thể được kiểm tra một cách đơn giản với `assertEquals`

```php
public function test_user_relation()
{
    $m = new Post();
    $relation = $m->user();

    $this->assertInstanceOf(BelongsTo::class, $relation);
    $this->assertEquals('user_id', $relation->getForeignKey());
    $this->assertEquals('id', $relation->getOwnerKey());
    //...
    $this->assertEquals($m->newQuery(), $relation->getQuery());
}
```

Tuỳ theo loại quan hệ, mà các test assertion sau cần phải được sử dụng:

- `getQuery()`: kiểm tra query chưa được chỉnh sửa, hoặc đã được chỉnh sửa một cách chính xác.
- `getForeignKey()`: với những quan hệ `HasOneOrMany` hoặc `BelongsTo`, nhưng loại key thì khác nhau (see documentaiton).
- `getQualifiedParentKeyName()`: với quan hệ `HasOneOrMany`, do không có `getLocalKey()` method, nên method này cần được kiểm tra.
- `getOwnerKey()`: với quan hệ `BelongsTo` và những quan hệ mở rộng từ nó.
- `getQualifiedForeignPivotKeyName()`: với quan hệ `BelongsToMany`.
- `getQualifiedRelatedPivotKeyName()`: với quan hệ `BelongsToMany`.
- `getTable()`: với quan hệ `BelongsToMany`. Không bắt buộc, bởi nó được bao gồm trong những method ở trên.
- `getMorphType()`: với quan hệ polymorphic.

### Property values

Rất nhiều models có thể bao gồm property mutators hay getters. Những methods này thực hiện tay đổi output trong rất nhiều trường hợp, bao gồm cả kết quả từ `__get` và `toArray()`.

Cần phải có những test cases thực hiện assertion trên tất cả các properties.

Trong trường hợp này, bạn cần sử dụng `unguarded` helper hoặc tự loại bỏ guards.

```php
public function test_properties_have_valid_values()
{
    User::unguard();

    $initial = [
        'name' => 'Jonh Doe',
        'email' => 'jonh@example.com',
    ];

    $m = new User($initial);

    $this->assertEquals($initial, $m->setHidden([]), $m->attributesToArray());
}
```

Trong trường hợp có những getter, thì giá trị trả ra có thể bị thay đổi, bởi vậy tests cần cover các trường hợp này.

Sử dụng `getAttributeValue()` để chắc chắn rằng getter đã được gọi. Sau đó gọi đến getter để test giá trị trả ra là khác nhau.

Trong trường hợp có cả mutators, sử dụng `setRawAttributes` để set giá trị ban đầu mà không gặp phải những sự thay đổi không lường trước được trên giá trị đó.

```php
public function test_status_getter()
{
    $m = new User();
    $m->setRawAttributes([
        'status' => 1,
    ]);

    // Test if getter is working.
    $this->assertEquals('enabled', $m->getAttributeValue('status'));

    // Test getter logic with different values.
    $this->assertEquals('disabled', $m->getStatusAttribute(2));
    $this->assertEquals('pending', $m->getStatusAttribute(3));

    // Abnormal case
    $this->expectException(\InvalidArgumentException::class);
    $m->getStatusAttribute(4);
}
```

Những quy tắc như vậy cũng được apply cho Mutators. Để tránh các getters được thực hiện, sử dụng method `getAttributes()` cho các assertions.

```php
public function test_status_getter()
{
    $m = new User();

    // Test if mutator is working.
    $m->setAttribute('status', 'enabled');
    $this->assertEquals(1, $m->getAttributes()['status']);

    // Test mutator logic with different values.
    $this->assertEquals(2, $m->setStatusAttribute('disabled');
    $this->assertEquals(3, $m->setStatusAttribute('pending');

    // Abnormal case
    $this->expectException(\InvalidArgumentException::class);
    $m->setStatusAttribute('invalid_status');
}
```

### Events

Model bao gồm protected static `$dispatcher` property, thứ có chứa application event handler. Nếu model của bạn sử dụng internal events hay observers, hãy chắc chắn rằng listeners tương ứng được assign thông qua class dispather được mock.

```php
public function test_listeners_attached()
{
    $d = m::mock(\Illuminate\Contracts\Events\Dispatcher::class);
    User::setEventDispatcher($d);
    $name = User::class;

    // Assert that created event has been assigned. Include additional checks if needed.
    $d->shouldReceive('listen')->once()->with("eloquent.created: {$name}", m::any());

    User::boot();
}
```

> Không sử dụng `Dispather::hasListeners()` method, bởi event có thể được assign từ bất kỳ đâu trong application.
> Mục đích của test này là để đảm bảo rằng events được assign từ bên trong Model.

Testing với observers nên tuân theo những cách thức tiếp cận chung giống với testing cho các class thông thường khác.

Event handlers được attach trong quá trình boot có thể được truy cập thông qua `getListeners()` method của event dispatcher.
Tuy nhiên hoàn toàn không nên sử dụng assignments kiểu như vậy, mà thay vào đó nên dùng observers.

### Additional checks.

Tất cả các method hay các configuration khác đều phải được test. Chúng bao gồm (không phải là toàn bộ):

- Custom model methods.
- Sử dụng Traits.
- Custom `CREATED_AT` và `UPDATED_AT` keys.
- `$with` property nếu có.
- `$incrementing` property nếu được thay đổi so với mặc định.
- `$dateFormat` property nếu được thay đổi so với mặc định.

## Examples

* [Example abstract model test helper](https://github.com/framgia/laravel-test-examples/blob/master/tests/ModelTestCase.php)
* [Example model](https://github.com/framgia/laravel-test-examples/blob/master/app/City.php) and [test](https://github.com/framgia/laravel-test-examples/blob/master/tests/Unit/CityTest.php)
