## Testing

We prefer automated tests. They enable us to do much more thorough testing as we can do thousands of tests rather than the tiny amount of tests you can do manually before going insane.


### Definitions

There are different types of automated tests. We use the following...

 1) Unit tests
 2) Functional tests
 3) Integration tests
 4) Browser tests


#### Unit test

A unit test should be insanely fast and its purpose is to show that the code does what we expect it to. If written properly they can also become a low level form of documentation.

A unit test tests 1 thing (generally a method/function) in isolation - meaning that we mock calls that leave the method in order to avoid the test also testing them. Let's take an example.

```javascript
function weTestThis(input, helper) {
    const gotThisFromHelper = helper.getNumberFrom(input);
    return gotThisFromHelper + 666;
}
```

The above function depends on the returned value from `helper`. As we are purely interested in testing `weTestThis`, we mock the `helper` so we do not end up testing that as well.


#### Functional test

Unit tests only ensure that each individual unit behaves as we expect - they do not ensure that the combined units (i.e. the system) works as it should. That is where functional tests come in.

A functional test ensures that an entire part of the system is working as it should, which means that they generally do not involve mocks. It also means that functional tests are much slower than unit tests.

Consider a project where we are building some sort of webservice/api. In the unit tests we have ensured that each handler and it's helpers are working individually, but we need a functional test to show that a call to the service results in the appropriate response. This shows that the wiring etc. is correct.


#### Integration test

Integration tests are similar to functional tests, but either test the system that we are making from the outside or they run tests that show that an integration we have made with an external system is working.

Testing from the outside, means that we start our service and then run tests in a different scope that calls the service (just like any other client). This is slightly different from a functional test where we generally inject a request directly into part of the system/server.


#### Browser test

A browser test is also a form of integration test, but it tends to use a webdriver to simulate a user clicking around in a browser to ensure that web pages work correctly.
