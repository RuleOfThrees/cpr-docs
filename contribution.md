---
layout: default
title: cpr - For Developers
---

## Overview and Filesystem Structure
Here we aim to cover some topics that will make getting into CPR development and debugging easier. This is not intended to be a thorough documentation of each function, class and template within the project. Rather, it aims to ease the process of contributing to CPR by covering some relevant topics.

In this document, we will be referencing files by their filename, relative to the appropriate root folder. Here are some particularly relevant ones, relative to project root:

* header files have the suffix `.h` and are located in [`include/cpr`](https://github.com/libcpr/cpr/tree/master/include/cpr).
* source files have the suffix `.cpp` and are located in [`cpr`](https://github.com/libcpr/cpr/tree/master/cpr).
* most files related to testing are located in [`test`](https://github.com/libcpr/cpr/tree/master/test).

## Building, Testing and Debugging
For a project that relies on template metaprogramming, such as CPR, compiling tests ensures that templates are instantiated during compilation. Testing and debugging features are enabled through setting certain cmake variables, which can be found in full in [`CMakeLists.txt`](https://github.com/libcpr/cpr/blob/master/CMakeLists.txt). We'd like to single out `CPR_BUILD_TESTS` and `CPR_DEBUG_SANITIZER_FLAG_ALL`, which enable the building of tests and the usage of a number of sanitizers. A debug build could be achieved as follows:

{% raw %}
```bash
# Make a build directory inside project root
$ mkdir build && cd build

# Make a Debug build with tests and sanitizers enabled
$ cmake -DCPR_BUILD_TESTS=1 -DCPR_DEBUG_SANITIZER_FLAG_ALL=1 -DCMAKE_BUILD_TYPE=Debug ..

# Build the project. The '-- -j7' part enables parallelization of build tasks.
# You may use a different number depending on your hardware, or omit this bit.
$ cmake --build . -- -j7

# Run all tests
$ cmake --build . --target test

# Run a specific test set
$ ./bin/session_tests

# See more options to run test suites, such as isolating specific tests
$ ./bin/download_tests --help

# Debug a test set
$ gdb ./bin/multiperform_tests

# See all targets for different Makefiles
$ make help
$ cd test && make help
```
{% endraw %}

## Project Structure
Here we will briefly describe different functional parts of CPR, and mention the headers that are most relevant to them. As any software project's, of course, CPR's parts are interconnected, and most classes play a part in most operations. Here, we will constrain ourselves to the most crucial ones. Where relevant headers are suggested, it's recommended to also look at the appropriate source file, if it exists.
### The API Interface
Relevant Headers: [`api.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/api.h), [`cpr.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/cpr.h)

Here we have the templates that are instantiated to create the cpr API methods. The namespace `priv` contains internal functions, particularly relevant for the `Multi`-methods.
### The Session class
Relevant Headers: [`session.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/session.h)

This class implements most of the logic used by `cpr`. Here, the `curl` `easy-` or `multi-` sessions are constructed, executed, the result packed in a `Response` instance and returned. It is _Moveable_, not _Copyable_, and can have shared ownership. Usage of the `Session` class usually entails construction, the setting of various parameters, either through the `Session::SetParam` methods like, for example, `Session::SetUrl`, or the equivalent polymorphic `Session::SetOption` method, and finally calling the appropriate method, like `Session::Get`. The set-up for the `curl` session happens, to a large degree, inside the `Session::PrepareAction` methods (where `Action` is the corresponding HTTP action).

### Response Wrappers
Relevant Headers: [`response.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/response.h), [`async_wrapper.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/async_wrapper.h)

Responses from API/Session actions come packaged in these classes. `Response` is the main container for a response, while `cpr::AsyncWrapper` is a container for a (cancellable or non-cancellable) asynchronous response with an interface compatible with [`std::future`](https://en.cppreference.com/w/cpp/thread/future).

### Parameter Containers
Relevant Headers: [`auth.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/auth.h), [`bearer.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/bearer.h), [`body.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/body.h), [`buffer.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/buffer.h), [`callback.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/callback.h), [`cert_info.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/cert_info.h), [`connect_timeout.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/connect_timeout.h), [`cookies.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/cookies.h), [`cprtypes.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/cprtypes.h), [`file.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/file.h), [`interface.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/interface.h), [`limit_rate.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/limit_rate.h), [`local_port.h`](https://github.com/libcpr/cpr/blob/master/include/cpr/local_port.h), ...

These classes are aimed to be used in order to supply arguments to `CPR` requests in an object-oriented way. To see how they are handled, it usually is a good idea to look at the implementation of the polymorphic `Session::SetOption` method.

### Placeholder
Relevant Headers: `ph.h`

Text goes here.

## Testing and Writing Tests

Test suites for CPR use the [gtest](https://google.github.io/googletest/) framework, and CPR's own [HttpServer](https://github.com/libcpr/cpr/blob/master/test/httpServer.hpp) in order to test behaviours. To see the various URIs that can be used to test different aspects of CPR's functionality, in may be useful to look at the implementation of [`HttpServer::OnRequest`](https://github.com/libcpr/cpr/blob/master/test/httpServer.cpp), and then at the different `HttpServer::OnRequest<URIName>` methods for a more detailed look at each URI's implementation.