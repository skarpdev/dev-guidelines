## Testing

We prefer automated tests. They enable us to do much more thorough testing as we can do thousands of tests rather than the tiny amount of tests you can do manually before going insane.

And remember that test code should be _clear_ not _clever_ (see [Why good developers write bad tests](https://www.youtube.com/watch?v=oO-FMAdjY68)).


### Definitions

There are different types of automated tests. We use the following...

 1. Unit tests
 2. Narrow integration tests
 3. Wide integration tests / api tests
 4. Browser tests


#### Unit tests

A unit test should be insanely fast and its purpose is to show that the code does what we expect it to - e.g. calls other things with expected parameters or follows the expected codepath). If written properly they can also become a low level form of documentation.

A unit test, tests exactly 1 thing (aka a `unit`, which is generally a method/function) in isolation - meaning that we mock calls that leave the method in order to avoid the test also testing them. Let's take an example.

```javascript
function weTestThis(input, helper) {
    const gotThisFromHelper = helper.getNumberFrom(input);
    return gotThisFromHelper + 666;
}
```

For the above method, we want to ensure that:

1. we call the helper correctly
2. that we add 666 to the result from helper
3. that we return that new value

We only want to test `weTestThis`, so we mock `helper`. That allows us to control the returned value from `helper` and means that we do not have to care about what `input` actually looks like.

So our test suite might look something like this:

```javascript
describe('weTestThis', () => {

    let helper = somehowMakeAMock(...);

    it('should call the helper with correct parameters', () => {
        const input = new Input();

        weTestThis(input, helper);

        // verify that helper.getNumberFrom(..) was called once with "input"
        verify(helper.getNumberFrom).wasCalledWith(input).once();
    })


    it('should return response from helper with 666 added', () => {
        const input = new Input();

        // control the value returned from the helper
        setReturn(helper.getNumberFrom, 123);

        const actual = weTestThis(input, helper);

        assert.equals(actual, 123 + 666);
    })
});
```

The above examples use a javascript inspired pseudo-language.


#### Narrow integration tests

Unit tests only ensure that each individual unit behaves as we expect - they do not ensure that the combined units (i.e. the system) works as it should. That is where integration tests come in. These comes in multiple varieties, but here we focus on the **narrow** integration tests - i.e. tests that test multiple components together, but not the entire stack from end-to-end.

**Typically** we use these for testing that our database code does what we expect it to do, meaning that the components we test are _repository classes_ and a _database instance_.

We do not test trivial _get by id_ style queries, but we do test those that use filtering on multiple columns as problems from these queries not returning as expected can be very hard to detect during normal use.

Let us say that we want to test the following:

```javascript
class SomeRepository {
    async function getNewestOnesAsync(color, howMany) {

        //
        // using an ORM
        //
        return db
            .from('Something')
            .where(row => row.color == color)
            .orderByDescending(row => row.createdAt)
            .limit(howMany);

        //
        // using direct SQL
        //
        return db.readAsync('select * from Something where color=$1 order by createdAt desc limit $2', color, howMany);
    }
}
```

What we want to ensure, is that the resulting query to the database works as we intend it to. The example above includes 2 implementations: one using an ORM and one using raw queries. The implementation details do not matter when testing this, as what we are testing is that a given input results the in expected output, when the state of the database is as we define.

The tests might look like this:

```javascript
describe('SomeRepository', () => {

    it('should return the X newest entries with the given color in descending order', async () => {

        // setup the database state
        await dbHelper.ensureTableExistsAndIsEmptyAsync('Something');
        await dbHelper.insertAsync(new Something('id-1', 'blue', '2020-01-01 12:34:56')); // 3rd
        await dbHelper.insertAsync(new Something('id-2', 'red', '2020-06-16 11:23:35'));  // wrong color
        await dbHelper.insertAsync(new Something('id-3', 'blue', '2020-01-01 05:01:02')); // 4th, so not included
        await dbHelper.insertAsync(new Something('id-4', 'blue', '2020-03-01 12:34:56')); // 1st
        await dbHelper.insertAsync(new Something('id-5', 'blue', '2020-03-01 12:34:00')); // 2nd

        // make the call
        const actual = getRepository().getNewestOnesAsync('blue', 3);

        // assert that we got back exactly what we expected in the order we expected them
        assert.equals(actual.count, 3);
        assert.equals(actual[0].id, 'id-4');
        assert.equals(actual[1].id, 'id-5');
        assert.equals(actual[2].id, 'id-1');

    });
});
```


#### Wide integration tests

**Wide** integration tests even more components at the same time. For service/API style backends, we typically call these `api tests` as we use them to test the service from the outside - i.e. we spin up the service and call the API just like any consumer of the service would. This helps us test that all the wiring in the service works (endpoints are found, middleware works as expected, mapping works, dependency injection works etc).

These tests are resource heavy to write and run, so we try to keep them to an absolute minimum. Typically we end up with tests like the following for each endpoint:

- a test without any _authentication_ at all, so we can ensure that the call is blocked and no data leaks out
- a test with invalid (e.g. expired) _authentication_, so we can ensure that the call is blocked and no data leaks out
- a test with valid _authentication_, but lacking the required _authorization_, so we can ensure that the call is blocked and no data leaks out
- a test that calls the endpoint with correct state etc, so we can expect a positive response

On top of that, if the endpoint in question returns a response with an abnormal "shape" in certain cases (e.g. error cases), then we also test that.

These api tests can also work as a example code for anyone who needs to build something that calls the service.

We tend to also include a number of flow tests - e.g. tests that calls the api multiple times during its run in order to simulate some kind of usage flow. An example of this could be an e-commerce system where a flow test might be to:

- create a basket
- put something in the basket
- finalize the basket
- do the payment
- check that an order was created


#### Browser test

A browser test is also a form of wide integration test, but it tends to use a webdriver to simulate a user clicking around in a browser to ensure that web pages work correctly.

