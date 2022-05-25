#  DisifyApiTests

## HTTP Rest Api

**Http Rest API** ensures communication between services using HTTP protocol. According to requirements there are available various types of method - Get, Post, Put, Delete etc. Http request consist of url address, query parameters, HTTP method, headers and body. The answer for Http request is response which contains status code, headers and body.
**Http protocol** is widely used in mircoservices architecture as well as in classical communication between client and server. 

## Introduction to automated Api tests for Disify Api

Disify project offers validation and verification email address. It's possible to check if email address is disposable, temporary, has invalid MX records, detect if its mistyped, inactive or non-existent.
Link to disify project - https://disify.com/.
In my exercise project I chose several test cases and automated them using Postman technology.

## Link to Postman Disify Api tests collection

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/7c78cc74ba9234c0e23f?action=collection%2Fimport)

## Why automated API testing is more efficient and accurate than manual testing?

1. [Separated Api Testing] / [Exact bug localization] <br/>
Manual testing perceived as E2E tests checks all layers of application - UI, server layer, database. It has pros and cons. Difficulty in bug localization is the one of the biggest problems. Omitting Frontend makes possible to accurate bug locaalization on Api side. 

2. [Time] <br/>
API testing is much faster and more efficient than UI testing. Manually testing consumes a lot of time and focus, sometimes one wrong click can ruin the test result and the work must be done from the beginning. In this case Api testing can save you a lot of time.
Automated Api tests are repeatable and doesn't rely on human factor.

3. [Api testing can be started at the very beginning] <br/>
Api testing can be started at the initial stage of the software development life cycle, before the user interface is ready. We can achieve two advantages:
    - We can find bugs at a very early stage of the software development cycle, this means that the cost of repairing a bug is low
    - We can find inccorect business assumptions, whose changes doesn't cost too much at the early stage 
    
4. [The ability to participate in projects where working with API becomes necessary to design tests] <br/>
Nowadays one of the most popular architecture is the architecture of microservices, when the application is not monolithic but becomes a superset of independent applications which exchange data with each other using a few methods like Rest Api. In this case, Api testing can be crucial to keep system working. We can tests microservices separately (when tested endpoint doesn't communicate with any other service) as well as integrately (when tested endpoint communicate with other services).

## Why use Postman technology to test API?

1. All saved queries sync in the cloud. In addition to sending queries to the API, Postman has a Tests tab, which allows you to enter the script using JavaScript. Thanks to this, we can test our response received from the API

2. Thanks to the possibility of introducing test scripts, Postman can allow us to automate the verification of websites. The benefit of introducing automation to our queries will be the ability to perform regression tests smoothly.

## Description

So the first way to test API requires writing request. Positive single email test verification get response - results displayed in JSON string, as above with additional information that email is whitelisted (it wasn’t mentioned in the doc-umentation)
```
{
    "format": true,
    "domain": "gmail.com",
    "disposable": false,
    "dns": true,
    "whitelist": true
}
```
Thanks to Json response, you can analyze if it`s the same as in the documentation.

Apart from Response analyze, you can run both functional and performing tests and much more 😊 
Postman has written automated test cases snippets in Java Script, therefore writing test cases is faster and more enjoyable 😊
Modyfing the postman snippets tests you can test:

1. Server response 

pm.test("Successful email verification", function () {
    pm.response.to.have.status(200);
});

2. Response time – necessary in performance testing :)

pm.test("Response time is less than 1s", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});

3. Response body - including checking particular json response fields (for email address testing - format, domain, disposable, dns) 

Example test, which verify format field from json response:
```
pm.test("When email is correct response should contain format property equals true", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.format).to.eql(true);
});
```

In Contrast disposable test case for non-disposable email format differs variable json data disposable to equals false as below: 

```
pm.test("When email is non-disposa-ble response should contain disposable property equals false", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.disposable).to.eql(false);
});
```

Response can differ from the documentation. Inability to recognize the email format (with-out @) can occur an information: 
```
{
    "format": false
} 
```
Server cannot recognize format of the domain that is why response does not contain information about domain, disposable and dns.
But non-existing email domain in response contains information about valid format, fic-tional domain name, non disposable and false dns 
```
{
    "format": true,
    "domain": "il.com",
    "disposable": false,
    "dns": false
}
```
Testing two disposable email addresses and comparing responses.
First disposable email was generated by thichanthit domain (hezywope@thichanthit.com) after sending request, get a response body which doesn`t contain information about dns:
```
{
    "format": true,
    "domain": "thichanthit.com",
    "disposable": true
}
```
Second disposable email was generated by bunlets.com domain (sigoce8002@bunlets.com)
In this case body response contain wrong information about email disposable equals false while this email address actually is disposable! 
Response also contain information about dns, unlike in the case of disposable email test case above.
```
{
    "format": true,
    "domain": "bunlets.com",
    "disposable": false,
    "dns": true
}
```
According to disposable test cases, 
```
pm.test("When email is disposable response should contain disposable property equals true", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.disposable).to.eql(true);
});
```
the response is incomprehensible, because expected is true while in the test result: ex-pected false to deeply equal true
When email is disposable response should contain disposable property equals true | AssertionError: expected false to deeply equal true

Bug occurred also while testing email dns test case was written:
```
pm.test("When email is disposable response should not contain dns property", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.dns).to.eql(undefined);
});
```
the response is one more time incomprehensible, because expected is undefined while in test result is written expected false to deeply equal true
AssertionError: expected true to deeply equal undefined

In the case of disposable emails sourced from different domains dns and disposable verifi-cation failed, the response body is confusing test results are incomprehensible.

Post request – the response body is compatible with indications of documentation, tests have been passed property.

II. Mass emails verification 
Endpoint should include a few email address
Response should have json string from
```
{
    "total": 2,
    "invalid_format": 0,
    "invalid_dns": 0,
    "disposable": 0,
    "unique": 2,
    "valid": 2,
    "session": "e12000d147c27daf1244f3696f4df9b3"
}
```
Testing two email addresses response is compatible witch indications of documentation.

Error occurs while testing one disposable (hezywope@thichanthit.com) and one non-disposable email address, after sending correct request, respond body contains information that valid address emails equals 1 while request contains two valid emails. (Must note that body contains correct information about total addresses number, invalid format, disposable and unique.)
Test case:
```
 pm.test("When checking two valid emails response should contain valid property equals 2", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.valid).to.eql(2);
});
```
Got a result AssertionError: expected 1 to deeply equal 2
Result is confusing.

Error occurs while testing one disposable (dixiwov868@roxoas.com) and one non-disposable email address. Respond body contains information that disposable equals “0” while one of the checked email address is actually disposable.
Test case:
```
pm.test("When checking one disposable email and one non-disposa-ble email response should include disposable equals 1", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.disposable).to.eql(1);
});
```
Got a result When checking one disposable email and one non-disposable email response should include disposable equals 1 | AssertionError: expected 0 to deeply equal 1
Result is confusing.

Post request while verifying two email addresses – the response body is compatible with indications of documentation, tests have been passed property.



## Conclusion:

Api testing consume less time than manual testing, can exact localize a bug, testing Api can start at a very beginning of software development lifecycle, which can save time and money. Moreover nowadays microservices architecture is very popular, and preparing test cases for this kind of project requires good knowledge of Api testing.
Using Postman technology, you can fast write test cases using Postman snippets and what is important you can save them in certain  collection, which enable fast automated regression testing every time you need it.
There are also some cons - Postman test error messages can be unclear and confusing for some cases, but in my opinion after some time tester is getting used to those types of communicates.
