# Knowledge about Testing

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
    $email = "sample@framgia.com";
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
    $email = "samplesamplesamplesamplesamplesamples@framgia.com";
    $this->assertEquals(true, validate($email));
}

public function test_valid_email_format_and_length_max()
{
    // Email with length 50 (equal maximum)
    $email = "samplesamplesamplesamplesamplesamplesa@framgia.com";
    $this->assertEquals(true, validate($email));
}

public function test_valid_email_format_and_length_max_plus()
{
    // Email with length 51 (maximum + 1)
    $email = "samplesamplesamplesamplesamplesamplesam@framgia.com";
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
    $email = "framgia.com";
    $this->assertEquals(false, validate($email));
}

public function test_valid_email_format_and_length_exceeded()
{
    // Email with length 54
    $email = "samplesamplesamplesamplesamplesamplesample@framgia.com";
    $this->assertEquals(false, validate($email));
}
```

</details>

## Mocks

## Stubs

## Dummies

## Factories

