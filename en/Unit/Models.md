# Testing Models

Models are ones of mostly used classes in Laravel applications. And at the same time they are the most complicated
from all default components.

Depending on implementation, models might include validation, external services connection (e. g. ElasticSearch indexing),
property getters and various custom methods.

As far as models might be used in any part of application, they must be tested carefully.

## Guide

### Configuration properties

Any model includes some default configurations with public or protected properties. These properties might affect
application behaivor a lot, but they are the most easy to test.
You can simply assert return values of that property getters:

```php
public function test_contains_valid_fillable_properties()
{
    $m = new User();
    $this->assertEquals(['name', 'email'], $m->getFillable());
}
```

Following method tests must be included in tests:

- `$fillable` -> `getFillable()`
- `$guarded` -> `getGuarded()`
- `$table` -> `getTable()`
- `$primaryKey` -> `getKeyName()`
- `$connection` -> `getConnectionName()`: in case multiple connections exist.
- `$hidden` -> `getHidden()`
- `$visible` -> `getVisible()`
- `$casts` -> `getCasts()`: note that method appends incrementing key.
- `$dates` -> `getDates()`: note that method appends `[static::CREATED_AT, static::UPDATED_AT]`.
- `newCollection()`: assert collection is exact type. Use `assertEquals` on `get_class()` result, but not `assertInstanceOf`.

It is fine to put all checks inside one test method, e. g. `test_model_configuration()`

### Relations

Relations are defined as methods returning appropriate relation classes. In some cases relation may be defined
with custom foreign keys or additional query changes. The best way to test relation config is to assert values
on returned `Relation` instance.

In all default cases, any relations is being created with `$model->newQuery()` as builder instance. If not modified,
it can be simply asserted with `assertEquals`

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

Depending on relation type, following test asserts must be included:

- `getQuery()`: assert query has not been modified or modified properly.
- `getForeignKey()`: any `HasOneOrMany` or `BelongsTo` relation, but key type differs (see documentaiton).
- `getQualifiedParentKeyName()`: in case of `HasOneOrMany` relation, there is no `getLocalKey()` method, so this one should be asserted.
- `getOwnerKey()`: `BelongsTo` relation and its extendings.
- `getQualifiedForeignPivotKeyName()`: `BelongsToMany` relation.
- `getQualifiedRelatedPivotKeyName()`: `BelongsToMany` relation.
- `getTable()`: `BelongsToMany` relation. Optional, because included in methods above.
- `getMorphType()`: any polymorphic relations.

### Property values

Many models might include property mutators or getters. Those methods modify output in many cases including `__get` and
`toArray()` results.

There must be a test case, which makes assertion on all properties.
For such case, you can use `unguarded` helper or remove guards manually.

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

In case any getter exists, values might be modified, so tests should cover such additions.
Use `getAttributeValue()` to make sure, getter is called and property, and call to getter itself to test different values.
As far as mutators might also exist, use `setRawAttributes` to set initial values to ignore any unexpected value changes.

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

Same rules apply to mutators. In order to avoid any getters applied, use `getAttributes()` method for assertions.

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

Model includes protected static `$dispatcher` property, which stores application event handler. If your model
uses internal events or observers, make sure that proper listeners are assigned by mocking dispatcher class.

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

> Do not use `Dispather::hasListeners()` method, because events might be assigned from anywhere in the application.
> Purpose of this test is to make sure that events have been assigned inside the model.

Testing of observers should follow the same approach as for testing any usual classes.
While event handlers, attached during boot can be accessed via `getListeners()` method of event dispatcher.
However it is highly recommended not to use such assignments and use observers instead.

### Additional checks.

All additional methods and configurations must be tested if present, including but not limited to:

- Custom model methods.
- Usage of traits.
- Custom `CREATED_AT` and `UPDATED_AT` keys.
- `$with` property if set.
- `$incrementing` property if changed by default.
- `$dateFormat` property if changed by default.

## Examples

* [Example abstract model test helper](https://github.com/framgia/laravel-test-examples/blob/master/tests/ModelTestCase.php)
* [Example model](https://github.com/framgia/laravel-test-examples/blob/master/app/City.php) and [test](https://github.com/framgia/laravel-test-examples/blob/master/tests/Unit/CityTest.php)
