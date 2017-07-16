# Knowledge about Unit Testing

## Classify

Before creating of any test cases, we should determine input values of **particular function/method** which need to be test. The test cases should be designed to cover all combinations of input values and preconditions. Basically, we usually divide types of test case by 3 types of input dataset for unit testing.

- **Normal**: Inputs are normal range values (accepted). A large amount of codes can be covered just by running **normal** test cases.
- **Boundary**: Inputs are at or just beyond its maximum or minimum limits. They are used to identify errors at boundaries rather than finding those exist in center of input domain.
- **Abnormal**: Inputs are invalid (illegal or not expected) to test error handling and recovery.

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
One of the fundamental requirements of Unit Test is **isolation**. In general, isolation is hard (if not impposible) as there are always dependencies across the whole project.

Therefore, the concept of `Test Doubles` was born. A `Test Double` allows us to break the original dependency, helping isolate the unit.

Here are some types of `Test Doubles`

***Some parts of following definitions are taken from Martin Fowler's Blog [Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)***

### Dummies
- Dummy objects are passed around but never actually used. Usually they are just used to fill parameter lists.

### Fake
- Fake objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an in memory database is a good example).

### Stubs
- Stubs provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test.

### Mocks
- Mocks are objects pre-programmed with expectations which form a specification of the calls they are expected to receive.

### Stubs vs Mocks
- Stub helps us to run test. Otherwise, Mock is an object which runs the test.
- A Fake which you verify calls against is a Mock. Otherwise, it's a Stub.
- Stub can never fail the test. Otherwise, Mock can.

## Examples
- Here are some PHP Mocking Frameworks that you can use to easily create Mocks for testing:
    - Mockery: It is highly recommended. It has been already integrated with Laravel Project. Document [here](http://docs.mockery.io/)
    - Prophecy: A part of PHPSpec project, but can be used outside PHPSpec. Check it [here](https://github.com/phpspec/prophecy)
- Examples of creating Stubs and Mocks using **Mockery**

// TODO
