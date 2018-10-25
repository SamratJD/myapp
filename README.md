# RSVP API Automation

## Index

| **SECTIONS**             | **SUB-SECTIONS**                    |
| ----------------     | ------------------------------- |
| [Getting Started](#getting-started)   | [Prerequisites](#prerequisites) : [Framework Configuration](#framework-configuration) : [Folder Structure](#folder-structure)|
| [Syntax Guide](#syntax-guide)      | [url](#url) : [path](#path) : [request](#request) : [method](#method) : [status](#status) : [param](#param) : [header](#header) : [def](#def) : [assert](#assert) : [response](#response) : [read()](#read) : [match](#match) : [match-contains](#match-contains) : [match each](#match-each)|
| [Running the tests](#running-the-tests) | [Run all scenarios](#run-all-scenarios) : [Environment switch](#environment-switch) : [Run based on tags](#run-based-on-tags) |
| [Test Reports](#test-reports)      | [Cucumber-report](#cucumber-report) : [Cluecumber-report](#cluecumber-report) |

## Getting Started
These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.


### Prerequisites
* [Apache Maven](https://maven.apache.org/download.cgi) - The build tool used
* [Eclipse IDE](https://www.eclipse.org/ide/) - The IDE used for script development
* [Java 8](https://www.oracle.com/technetwork/java/javase/overview/index.html) (at least version 1.8.0_112 or greater)
* [Cucumber Eclipse Plugin](https://marketplace.eclipse.org/content/cucumber-eclipse-plugin) - To use the Cucumber editor and **feature** file runner

### Framework Configuration

Once all the mentioned prerequisites have been installed, then download the framework from the GitHub repository path: `https://github.aig.net/aadhikar/GRITQEA-RSVP-AGILEnet-PlanFees.git` and import it in Eclipse IDE as **Existing Maven Project**.

### Folder Structure

```



```

## Syntax Guide
### `url`
A URL remains constant until you use the url keyword again, so this is a good place to set-up the 'non-changing' parts of your REST URL-s.

```cucumber
Given : https://uat.cloud.api.aig.net/valic/valic-plan-fees-api/v1/
```

### `path`

REST-style path parameters. Can be expressions that will be evaluated. Comma delimited values are supported which can be more convenient, and takes care of URL-encoding and appending '/' where needed.

```cucumber
Given path /planfees/breakpointassettypes
```

### `request`

* In-line JSON:

```cucumber
Given request { groupNumber: '01015', planNumber: '001' }
```

* From a file in the same package. Use the classpath: prefix to load from the classpath instead.

```cucumber
Given request read('my-json.json')
```

* Defining the request is mandatory if you are using an HTTP method that expects a body such as post. If you really need to have an empty body, you can use an empty string as shown below, and you can force the right Content-Type header by using the header keyword.

```cucumber
Given request ''
And header Content-Type = 'text/html'
```
* Sending a file as the entire binary request body is easy (note that multipart is different):

```cucumber
Given path 'upload'
And request read('my-image.jpg')
When method put
Then status 200
```
### `method`

The HTTP verb - `get`, `post`, `put`, `delete`, `patch`, `options`, `head`, `connect`, `trace`.

Lower-case is fine.
```cucumber
When method post
```

It is worth internalizing that during test-execution, it is upon the `method` keyword that the actual HTTP request is issued. Which suggests that the step should be in the `When` form, for example: `When method post`. And steps that follow should logically be in the `Then` form. Also make sure that you complete the set up of things like `url`, `param`, `header`, `configure` etc. *before* you fire the `method`.

```cucumber
# set headers or params (if any) BEFORE the method step
Given header Accept = 'application/json'
When method get
# the step that immediately follows the above would typically be:
Then status 200
```
### `status`
This is a shortcut to assert the HTTP response code.
```cucumber
Then status 200
```
And this assertion will cause the test to fail if the HTTP response code is something else.

### `param` 
Setting query-string parameters:

```cucumber
Given param userName = 'JSmith'
```

The above would result in a URL like: `http://myhost/mypath?userName=JSmith`. Note that the `?` will be automatically inserted.

Multi-value params are also supported:
```cucumber
* param myParam = 'foo', 'bar'
```
You can also use JSON to set multiple query-parameters in one-line using `params` and this is especially useful for dynamic data-driven testing.

### `header`

It is to set HTTP headers, Karate does not provide any special keywords for things like 
the `Accept` header. You simply do 
something like this:

```cucumber
Given path 'some/path'
And request { some: 'data' }
And header Accept = 'application/json'
When method post
Then status 200
```
### `def`
**Set a named variable**
```cucumber
# assigning a string value:
Given def myVar = 'world'

# using a variable
Then print myVar

# assigning a number (you can use '*' instead of Given / When / Then)
* def myNum = 5
```
Note that `def` will over-write any variable that was using the same name earlier.

### `assert`
**Assert if an expression evaluates to `true`**

Once defined, you can refer to a variable by name. Expressions are evaluated using the embedded JavaScript engine. The assert keyword can be used to assert that an expression returns a boolean value.

```cucumber
Given def color = 'red '
And def num = 5
Then assert color + num == 'red 5'
```
Everything to the right of the `assert` keyword will be evaluated as a single expression.

Something worth mentioning here is that you would hardly need to use `assert` in your test scripts. Instead you would typically use the `match` keyword, that is designed for performing powerful assertions against JSON and XML response payloads.

### `response`
After every HTTP call this variable is set with the response body, and is available until the next HTTP request over-writes it. You can easily assign the whole `response` (or just parts of it using Json-Path or XPath) to a variable, and use it in later steps.

The response is automatically available as a JSON, XML or String object depending on what the response contents are.

As a short-cut, when running JsonPath expressions - `$` represents the `response`.  This has the advantage that you can use pure JsonPath and be more concise.  For example:

```cucumber
# the three lines below are equivalent
Then match response $ == { name: 'Billie' }
Then match response == { name: 'Billie' }
Then match $ == { name: 'Billie' }

# the three lines below are equivalent
Then match response.name == 'Billie'
Then match response $.name == 'Billie'
Then match $.name == 'Billie'
```
### `read()`
Karate makes re-use of payload data, utility-functions and even other test-scripts as easy as possible. Teams typically define complicated JSON (or XML) payloads in a file and then re-use this in multiple scripts. Keywords such as `set` and `remove` allow you to to 'tweak' payload-data to fit the scenario under test. You can imagine how this greatly simplifies setting up tests for boundary conditions. And such re-use makes it easier to re-factor tests when needed, which is great for maintainability.

> Note that the `set` (multiple) keyword can build complex, nested JSON (or XML) from scratch in a data-driven manner, and you may not even need to read from files for many situations. Test data can be within the main flow itself, which makes scripts highly readable.

Reading files is achieved using the `read` keyword. By default, the file is expected to be in the same folder (package) and side-by-side with the `*.feature` file. But you can prefix the name with `classpath:` in which case the 'root' folder would be `src/test/java` (assuming you are using the recommended folder structure).

Prefer `classpath:` when a file is expected to be heavily re-used all across your project.  And yes, relative paths will work.

```cucumber
# json
* def someJson = read('some-json.json')
* def moreJson = read('classpath:more-json.json')

# xml
* def someXml = read('../common/my-xml.xml')

# import yaml (will be converted to json)
* def jsonFromYaml = read('some-data.yaml')

# string
* def someString = read('classpath:messages.txt')

# javascript (will be evaluated)
* def someValue = read('some-js-code.js')

# if the js file evaluates to a function, it can be re-used later using the 'call' keyword
* def someFunction = read('classpath:some-reusable-code.js')
* def someCallResult = call someFunction
```
### `match`
**Payload Assertions / Smart Comparison**

The `match` operation is smart because white-space does not matter, and the order of keys (or data elements) does not matter. Karate is even able to [ignore fields you choose](#ignore-or-validate) - which is very useful when you want to handle server-side dynamically generated fields such as UUID-s, time-stamps, security-tokens and the like.

The match syntax involves a double-equals sign '==' to represent a comparison (and not an assignment '=').

```cucumber
Then match response.planFeesResponse.compareReport.totalCount == 15
```
### `match contains`
**JSON Keys**

In some cases where the response JSON is wildly dynamic, you may want to only check for the existence of some keys. And `match` (name) `contains` is how you can do so:

```cucumber
* def foo = { bar: 1, baz: 'hello', ban: 'world' }

* match foo contains { bar: 1 }
* match foo contains { baz: 'hello' }
* match foo contains { bar:1, baz: 'hello' }
# this will fail
# * match foo == { bar:1, baz: 'hello' }
```
### `match each`
The `match` keyword can be made to iterate over all elements in a JSON array using the `each` modifier. Here's how it works:
```cucumber
* def data = { foo: [{ bar: 1, baz: 'a' }, { bar: 2, baz: 'b' }, { bar: 3, baz: 'c' }]}

* match each data.foo == { bar: '#number', baz: '#string' }

# and you can use 'contains' the way you'd expect
* match each data.foo contains { bar: '#number' }
* match each data.foo contains { bar: '#? _ != 4' }

# some more examples of validation macros
* match each data.foo contains { baz: "#? _ != 'z'" }
* def isAbc = function(x) { return x == 'a' || x == 'b' || x == 'c' }
* match each data.foo contains { baz: '#? isAbc(_)' }
``` 

## Running the tests
* Normally in dev mode, you will use your IDE to run a *.feature file directly or via the companion 'runner' JUnit Java class.
* When you have a 'runner' class in place, it would be possible to run it from the command-line as well.
* To execute the test cases we are using maven commands as given below:

### Run all scenarios
`mvn clean test  -Dtest=ExecuteTest -Dmaven.test.failure.ignore=true`

### Environment switch

1. Run in non-prod environment:

`mvn clean test -DargLine="-Dkarate.env=non-prod" -Dtest=ExecuteTest -Dmaven.test.failure.ignore=true
`

2. Run in dev environment:

`mvn clean test -DargLine="-Dkarate.env=dev" -Dtest=ExecuteTest -Dmaven.test.failure.ignore=true
`

### Run based on tags

1. Run **smoke** scenarios

`mvn clean test -Dcucumber.options="--tags @smoke" -Dtest=ExecuteTest -Dmaven.test.failure.ignore=true`

2. Run **regression** scenarios

`mvn clean test -Dcucumber.options="--tags @regression" -Dtest=ExecuteTest -Dmaven.test.failure.ignore=true`

## Test Reports
### Cucumber-report
This report gets generated in the `target\cucumber-html-reports` folder. Below is the screenshot of the report:

### Cluecumber-report
This report gets generated in the `target\generated-report` folder. Below is the screenshot of the report:
