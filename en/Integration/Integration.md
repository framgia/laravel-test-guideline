# Integration Tests

Purpose of integration tests is to make sure that all components work together as needed.

While making such tests, remember that changes to data storage or interactions with external services might happen.
In order to handle such cases properly, you might still need to create [Test Doubles](../Knowledge.md#test-doubles) for some classes.

Which classes should be doubled depends on your testing approach and might be different depending on app size.

## Guide

Laravel provides 2 ways to create integration tests: making HTTP requests to routes and browser-based testing with
`laravel/dusk` package.

It is required to have HTTP tests, while browser tests don't have any specific requirements.

Integration tests must cover EVERY route of application and complete following purposes:

### HTTP
- Authentication and policy tests. Each case must have separate test assertion.
- Check valid status codes for every type of response.
- Check correct redirect codes and paths on different events.
- Test Error handlers for correct responses.

### Database
- Ensure data is written to database correctly during requests.
- Test migration process.
- Test abnormal data insertions are handled properly.

If your application interacts with external services and servers, you either need to use staging versions of
such services or mock your client classes.

Testing environment can be defined with special configurations and variables, e. g. `testing` database connection with
`DB_TEST_DATABASE` env variable. However it is **strictly recommended** to replace production variables instead in
phpunit.xml or CI configuration.

It is good to run tests twice with `APP_ENV` set both to `testing` and `production`.

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

### REST API / JSON API
- Test validity of JSON responses.
- Check validity of API versions (if necessary).
- Ensure Error handlers provide valid JSON output.

#### Idempotency and safety
For REST APIs one of the most important things is following idempotency rules. An idempotent HTTP method is a HTTP method that can be called many times without different outcomes. It would not matter if the method is called only once, or ten times over. The result should be the same.

Even though a request might be idempotent, it still might make changes to data, however, safe requests do not make any changes. If a seemingly safe method like GET will change a resource, it might be possible that any middleware client proxy systems between you and the server, will cache this response.

Use the following table to check your methods.

| HTTP Method | Idempotent | Safe |
|-------------|:----------:|:----:|
| OPTIONS     |     yes    |  yes |
| GET         |     yes    |  yes |
| HEAD        |     yes    |  yes |
| PUT         |     yes    |  no  |
| POST        |     no     |  no  |
| DELETE      |     yes    |  no  |
| PATCH       |     no     |  no  |

- **Important:** use Integration tests to ensure that not any unnecessary database or external queries made. Ensure that safe methods do not produce data-affecting changes. The best way is to restrict any data changes on test case setup and allow specific changes before running assertions.
