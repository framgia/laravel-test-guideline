# Testing Controllers

Controllers implement the main request processing algorithm. They must be paid the most attention during testing process.

Unit tests written for controllers must be totally isolated and every external call from inside controller method must be mocked.

Depending on your application logic, controllers might implement validation in method logic or via form requests.
If first, controller tests must include cases with boundary and abnormal request input. Otherwise those cases should be
tested in appropriate form request test cases.
However, this affects only user input. Data provided from outside controller method must be mocked properly with all kinds of data.

## Guide

## Examples
