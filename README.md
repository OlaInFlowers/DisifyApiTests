#  DisifyApiTests
## Goals and limitations of the project
The main goal of the project is getting familiar with communication using Http Api as well as introduction to testing process automation using Postman.
Http Api, which I've tested is available and described at https://www.disify.com/. **Disify** offers validation and verification email address. It's possible to check if email address is disposable, temporary, has invalid MX records, detect if its mistyped, inactive or non-existent.<br/>
Within the project I chose several ednpoints (the most interesting in my opinion) and wrote various types of tests in Postman.

## HTTP Api

**HTTP Api** ensures communication between two systems using HTTP protocol. One system (server) exposes endpoints, which can be used by another system or systems (clients). Every endpoint description consist of url address, HTTP method, request parameters, request expected content and response format. In addition HTTP request can contain other elements like headers or cookies.<br/>
What can we send in HTTP request? Actually everything :) We can send data in JSON or XML format, files or just simple string.
What does HTTP response consist of? Essentially it contains HTTP status, which indicates whether out request has been processed successfully. Response content can contain various type of data, cookies and headers, as does HTTP request content.<br/>
**HTTP Api** is widely used in mircoservices architecture as well as in classical communication between client and server.

## Disify HTTP Api tests collection

The tests collection, which I created within the project is available at the following link:<br/><br/>
[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/7c78cc74ba9234c0e23f?action=collection%2Fimport)

## Tests description

I divided tests into two folders:
- *Mass Email Verification*
- *Single Email Verification*

Inside every folder I created several test cases, which check HTTP response correctness as well as non-functional aspects like response time.
Generally tests check whether Disify HTTP Api works as intended (at least according to documentation) 😊

Example test, which verifies format field from json response:
```
pm.test("When email is correct response should contain format property equals true", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.format).to.eql(true);
});
```

Another example test, which checks processing performance:
```
pm.test("Response time is less than 1s", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});
```

Of course I tested HTTP status code response as well:
```
pm.test("Successful email verification", function () {
    pm.response.to.have.status(200);
});
```

Most of the tests confirmed proper functioning of the Disify HTTP Api. Although I've found several errors: <br/><br/>
❌ Not all diposable emails are properly identified. Api worked correctly for `hezywope@thichanthit.com`, but `sigoce8002@bunlets.com` and `dixiwov868@roxoas.com` addresses has been verified as non-disposable (both for single and mass email verification).<br/>
Test case:
```
pm.test("When email is disposable response should contain disposable property equals true", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.disposable).to.eql(true);
});
```
Response body:
```
{
    "format": true,
    "domain": "bunlets.com",
    "disposable": false,
    "dns": true
}
```
❌ Mass emails verification for disposable and non-disposable addresses works incorrectly
Error occurs while testing one disposable (hezywope@thichanthit.com) and one non-disposable email address (but both with correct format), after sending correct request, response body contains information that valid address emails equals 1 while request contains two valid emails. (Must note that body contains correct information about total addresses number, invalid format, disposable and unique.).<br/>
Test case:
```
 pm.test("When checking two valid emails response should contain valid property equals 2", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.valid).to.eql(2);
});
```
Response body:
```
{
    "total": 2,
    "invalid_format": 0,
    "invalid_dns": 0,
    "disposable": 1,
    "unique": 2,
    "valid": 1,
    "session": "b29703e9fdd27bea7f43f03c4da16e69"
}
```

## My opinion

### Automated API testing is more efficient and accurate than manual testing

1. **Separated Api Testing / Exact bug localization**<br/>
Manual testing perceived as E2E tests checks all layers of application - UI, server layer, database. It has pros and cons. Difficulty in bug localization is the one of the biggest problems - there is no certainty whether the problem is in UI or on the Api side. Omitting Frontend makes possible to accurate bug locaalization on Api side.

2. **Time**<br/>
Api testing is much faster and more efficient than UI testing. Manually testing consumes a lot of time and focus, sometimes one wrong click can ruin the test result and the work must be done from the beginning. In this case Api testing can save you a lot of time. Automated Api tests are repeatable and doesn't rely on human factor.

3. **Api testing can be started at the very beginning**<br/>
Api testing can be started at the initial stage of the software development life cycle, before the user interface is ready. We can achieve two advantages:
    - We can find bugs at a very early stage of the software development cycle, this means that the cost of repairing a bug is low
    - We can find inccorect business assumptions, whose changes doesn't cost too much at the early stage
   
4. **The ability to participate in projects where working with API becomes necessary to design tests**<br/>
Nowadays one of the most popular architecture is the architecture of microservices, when the application is not monolithic but becomes a superset of independent applications which exchange data with each other using methods like HTTP Api. In this case, Api testing can be crucial to keep system working. We can tests microservices separately (when tested endpoint doesn't communicate with any other service) as well as integrately (when tested endpoint communicate with other services).

### Postman is a great tool for API testing

Postman helps me understand HTTP Api idea and possibilites. I think it's great tool to simple invoke HTTP Api endpoints as well as speed up testing process.
Postman has a lot of advantages, most important of which are (in my opinion):
- Work synchronization in the cloud
- Quite intuitive interface
- Test cases snippets
- Tests management - collection, folder, sub-folders. It makes possible to keep order and simplify executing tests process

Of course there is no perfect tool and Postman has some cons as well.
In my opinion error messages are a little bit confusing. For example we want to check whether some json response field equals true - received error message is like below:
```
AssertionError: expected false to deeply equal true
```
I think that message like ```AssertionError: expected true, but received false``` would be clearer.
Another problem, which I encountered is connected with tests collection, which uses environment variable. When I shared such collection and try to import it from another account I received error - the environment variable is undefined 😕, so I simply copied variable value to all places, where it was used - not too nice, but at least it works.

## Conclusion:

Automated HTTP Api testing has a lot of advantages:
- consumes less time than manual testing
- can more accurately localize a bug
- can start at a very beginning of software development lifecycle, which can save time and money.
HTTP Api is widely used and there is no indication of change in this context.
Postman makes possible to write test cases very quickly thanks to test code snippets and what is important you can save them in certain collection, which enable fast automated regression testing every time you need it. Despite of several inconveniences Postman is a great tool for HTTP Api testing in my opinion 😊
