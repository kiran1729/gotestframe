# gotestframe

gotestframe is a testing framework for golang projects that supports JSON and golang template based test writing.

The framework’s main features include
* JSON-based specification of test
* Golang templates for controlling and generating the JSON tests
* Using reflection to register and call golang methods in the tests
* Use JSON path specification to extract data from inputs and outputs for later use
* Use JSON for test output checking
* Ability to compose multiple tests together into lists
* HTTP calls encoded in JSON
* Saving and restoring state

We used JSON as the language to write the tests for two reasons. First, our microservices based design already accepted JSON as the input/output format and keeping to JSON as the test language helped make it easy to add the tests. Second, even for any generic golang code, we could convert JSON into arbitrary inputs and outputs of golang functions due to excellent built-in support. The automation framework goes beyond reading a bunch of JSON input from files and sending it to the services. We make extensive use of JSON, templates, interfaces and reflection to build the framework which makes it unique to golang.

The framework organizes tests into lists, tests, and operations. A list is composed of multiple tests and tests consist of a list of operations. The framework executes the list of tests in order and, with each test, it executes the list of ops in order. The framework can also execute multiple ops files in parallel using separate goroutines.

Lists, tests, and operations are files with golang templating around JSON which enabled loops and advanced control that golang templates support. A list of tests or ops can also be tuned by using a separate JSON as a control input that drives the template execution. As an example, the template can specify a loop count variable that is actually then controlled from an external control file. This makes it easy to write generic ops in template files that are then used to generate the actual JSON used to feed the test core.

Templates also support embedding pre-defined functions to generating inputs such as random strings, numbers and random email addresses. As an example, we use templates to iterate over a range of process names to randomly kill processes to generate a “chaos monkey”-style test.

The operations are specified as either HTTP calls or as struct method function calls. Both can be intermingled and the test automation framework automatically invokes the correct operation.

For HTTP calls, the JSON specification has the URL fragments, headers and the body to be sent with the appropriate HTTP method to be called. The automation framework calls the appropriate service by constructing the HTTP request and sending the request. The result returned by the call is also processed (and headers extracted) as needed.

Operations can also be methods called on golang structs that are registered before the tests start running. This is a huge advantage since it enables all unit tests that already exist to be reused as possible function calls from ops. A classic example of this approach is where the golang code would wait in a timed loop for a service state to be in an expected state. The method can be specified as a path on the registered object. We use reflection to walk the registered struct and its member variables and find the actual function and invoke it via reflection. The parameters to the function are constructed dynamically from the input JSON using reflection.

Operations defined in tests are never really fully independent. Many times the results of one operation need to be used in another operation. For instance, a resource creation call to one service in an operation can return the id of that resource and in the next REST call to another service, that id needs to be used in the URL or in the body.

The automation framework we built supports this in an elegant manner. The JSON specification can also specify symbols to be extracted from the output (or even input) into a symbol table by specifying the JSON path of the item to be extracted. Using the path and golang’s support for partial unmarshaling into an interface, symbols are extracted and saved as raw messages irrespective of the type of item being extracted (i.e. numbers, strings, arrays or JSON objects). The symbols can then be used in other places as inputs to other operations after using reflection to construct the required inputs from the function signature. The framework can also save and reload the symbols so they can be shared across tests.

Test writers can specify a JSON path and JSON fragment which can be used to verify the result of an operation automatically through the framework. As an example, a resource creation operation can specify a check for the state returned by the operation to a service to be correct while ignoring the id returned since the id will be unique and cannot be checked automatically.


