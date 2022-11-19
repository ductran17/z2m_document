## Table of Content:
- [Prerequisite](#prerequisite)
- [General Test Process](#general-test-process)
- [Jest install & configuration](#jest-install--configuration)
- [How to write test for Z2M](#how-to-write-test-for-z2m)
  - [Test file creation rules](#test-file-creation-rules)
  - [Writting test via example](#writting-test-via-example)
    - [**Step 1: Mocking function**](#step-1-mocking-function)
    - [**Step 2: Test cases setup**](#step-2-test-cases-setup)
    - [**Step 3: simpleDescriptor() test**](#step-3-simpledescriptor-test)
- [Run test](#run-test)
- [Reference](#reference)
# Prerequisite
To write your own test, you need to grasp the basic concepts and features of Jest testing, which can be found via the [official document on Jest website](https://jestjs.io/docs/getting-started). Main sections that readers need to focus are:
- [Getting Started](https://jestjs.io/docs/getting-started)
- [Using Matchers](https://jestjs.io/docs/using-matchers)
- [Testing Asynchronous Code](https://jestjs.io/docs/asynchronous)
- [Setup and Teardown](https://jestjs.io/docs/setup-teardown)
- [Mock Functions](https://jestjs.io/docs/mock-functions)
# General Test Process

You can think of the test as: checking whether the function produces the expected result. The most typical test process is as follows:

- Import the function to be tested
- Give the function an input
- Define the desired output
- Check if the function produces the expected output

# Jest install & configuration
 First, we install Jest:

```js 
npm i jest --save-dev
```

To configure an NPM script for running tests on command line, open up `package.json` and configure as below (already configured in Z2M project):
```js 
  "scripts": {
    "test": "jest test",
    "test-with-coverage": "jest test --coverage",
    "test-watch": "jest test --watch",
  },
```
Install [Jest extension](https://marketplace.visualstudio.com/items?itemName=Orta.vscode-jest) on Vscode, which helps to display all test cases's status and many other features (recommend).

 
# How to write test for Z2M
## Test file creation rules
- All test files must be placed in `test` folder, which can be found in each of `zigbee2mqtt`, `zigbee-herdsman`, `zigbee-herdsman-converter`, `zigbee2mqtt-frontend`.
- All test files must be named after the module that it run tests on, for e.g: `adapter.test.ts`.
## Writting test via example
- We will take test case `simpleDescriptor` in `zigbee-herdsman/test/adapter/z-stack/adapter.test.ts` as an example for writting a test case. 
- This test was created to verify the working of function `simpleDescriptor()` in `zigbee-herdsman/src/adapter/z-stack/adapter/zStackAdapter.ts`

### **Step 1: Mocking function**

[Mock functions](https://jestjs.io/docs/mock-functions) were created with `znp.waitFor()`, `znp.open()`, `znp.Request()`, `znp.close()`, `queue.execute()` 

*(Mocking is to isolate the behaviour of the object you want to test you replace the other objects by mocks that simulate the behaviour of the real objects).*
```js
const mockZnpWaitFor = jest.fn();
const mockZnpOpen = jest.fn();
const mockZnpClose = jest.fn();
const mockQueueExecute = jest.fn().mockImplementation(async (func) => await func());
```
=> ***when these functions are called, mocked versions of them will be executed instead of the real one.***

### **Step 2: Test cases setup**
Setup for all the tests case in test file:
```js
beforeEach(() => {
        jest.useRealTimers();
        jest.useFakeTimers();
        adapter = new ZStackAdapter(networkOptions, 
        ........
    });
```
The wrapped code in beforeEach will be executed before every test cases in the test file.

### **Step 3: simpleDescriptor() test**
This is the complete test case for `simpleDescriptor`:
```js
it('Simple descriptor', async () => {
        basicMocks();
        await adapter.start();
        const result = await adapter.simpleDescriptor(1, 20);
        expect(mockQueueExecute.mock.calls[0][1]).toBe(1);
        expect(result).toStrictEqual({
            deviceID: 7,
            endpointID: 20,
            inputClusters: [8],
            outputClusters: [9],
            profileID: 124,
        });
    });
```
The test case has its own `test\it` test block with following process: 

- test block description: `"Simple descriptor"`.
- `basicMock()` to mock `znp.request` and `znp.waitFor` function (create a different mock version from mocking in previous step)
- execute and return the result of `adapter.simpleDescriptor()`. Inside this function, mock functions in the previous step will be called instead of real version of them.
- `expect` wraps the objective function and combines it with the [Matcher](https://jestjs.io/docs/using-matchers), for e.g:  `toBe`, to check whether the result meets expectations. 
- `mockQueueExecute.mock.calls[0][1]` is to access to the second parameter of the first call to function `queue.Execute()`


# Run test
To run all test files:

```
    npm run test
```


To run all test cases in a test file:

```
    jest $testFileName
```

To run some specific test cases in a test file:
- mark all these test cases with `only`, for e.g: `it.only("Simple Descriptor, () => ...")`
- use the command:

```
    jest $testFileName
```
To run and output the coverage of the test (see coverage report via console log or `/coverage/lcov-report/index.html`):
```
    npm run test-with-coverage
```


To run and output the watch mode of Jest test:
```
    npm run test-watch
```

# Reference

- [https://jestjs.io/docs/getting-started](https://jestjs.io/docs/getting-started)
- [https://medium.com/@kalone.cool/explain-the-implementation-principle-of-the-jest-framework-in-a-simple-way-222b11a55c04](https://medium.com/@kalone.cool/explain-the-implementation-principle-of-the-jest-framework-in-a-simple-way-222b11a55c04)
- [https://softwaretestingnotes.com/](https://softwaretestingnotes.com/)
- [https://dev.to/wscats/explain-the-implementation-principle-of-the-jest-framework-in-a-simple-way-4dio](https://dev.to/wscats/explain-the-implementation-principle-of-the-jest-framework-in-a-simple-way-4dio)
