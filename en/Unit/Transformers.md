# Testing Transformers and Presenters

Almost every project includes serializations and presenters in one or another way.

The default way for Laravel is to use mutators/getters on model classes, however in more complex applications,
a variety of ways may be applied.

This guide is applicable for general cases of transformers or presenters.

## Guide

Having appropriate transformers is critical for API providing applications and highly recommended for others.

Transformer tests must complete following purposes:

1. Ensure data consistency (object structure must not change between commit unexpected).
2. Ensure property mutations are done correctly.
3. Ensure data types match expected output.

For many API client applications appearance of new properties or absence of some can be critical.
It means that any change to such structure must be under control.

Ensure that your transformers always have consistent data

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

If your application implements feature tests for API, it is suitable to replace this check with `assertJsonStructure`
method in feature test case.

For JSON outputs it is necessary to check data types, espcecially if your API clients might be written in strict type
languages.

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
