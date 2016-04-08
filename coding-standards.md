# **Coding Standards and Practices**
What follows collects our coding standards and practices. Where it makes sense, examples have been given to illustrate the particular standard or description of practice.

As much as possible, this document attempts to follow the definitions of the terms **MUST**, **SHOULD** and **MAY** (and their negations) as defined in [RFC2119](https://tools.ietf.org/html/rfc2119).

In many cases in this document, code commenting constructs are used to provide additional in-place explanations of the process or standard being explained. Additionally,
those constructs may be abbreviated in an attempt to reduce distraction from that which is being explained. Neither of these uses of commenting constructs are intended to
provide examples of how such constructs should be used in practice.

## **Table of Contents**
1. [General](#general)
   1. [Header Comment](#header-comment)
   1. [`'use strict';`](#use-strict)
   1. [Single-Quoted Strings](#single-quoted-strings)
   1. [Indentation Character](#indentation-character)
   1. [Equality Operators `===` and `!==`](#equality-operators-and-)
   1. [Truthy and Falsy](#truthy-and-falsy)
      1. [Casting To Boolean With `!!` and `!`](#casting-to-boolean-with-and-)
   1. [Identifier Naming](#identifier-naming)
   1. [Accessing Globals](#accessing-globals)
      1. [Server-Side Node.js Code](#server-side-nodejs-code)
      1. [Client-Side Code](#client-side-code)
   1. [Using `require` In Node](#using-require-in-node)
   1. [Using `parseInt`](#using-parseint)
   1. [Constructing and Raising Errors](#constructing-and-raising-errors)
   1. [Logging Errors](#logging-errors)
   1. [Promises](#promises)
      1. [Promise Structure](#promise-structure)
      1. [The Promise Constructor vs. The Deferred Pattern](#the-promise-constructor-vs-the-deferred-pattern)
         1. [The Explicit Construction Anti-Pattern](#the-explicit-construction-anti-pattern)
      1. [Starting a Promise Chain](#starting-a-promise-chain)
1. [Application Layers](#application-layers)
   1. [Route Layer](#route-layer)
      1. [HTTP Verbs](#http-verbs)
      1. [API Documentation](#api-documentation)
   1. [Controller Layer](#controller-layer)
      1. [Route Parameters and Query Parameters](#route-parameters-and-query-parameters)
   1. [DAL Layer](#dal-layer)
   1. [Adapter Layer](#adapter-layer)
1. [Data Retrieval](#data-retrieval)
   1. [SQL Query Formatting and Execution](#sql-query-formatting-and-execution)
   1. [Pagination and Sorting](#pagination-and-sorting)
      1. [Validating the `sortby` Parameter](#validating-the-sortby-parameter)
   1. [Empty Result vs. Not Found](#empty-result-vs-not-found)
   1. [Transactionalized SQL](#transactionalized-sql)

## **General**
These are the general standards and practices that apply, more or less, across the board for most of our applications.

Check out the [CHANGELOG](/TeraLogics/coding-standards/blob/master/CHANGELOG.md) to see what's new and what has been changed. 

### **Header Comment**
The first line of each file **MUST** be the copyright header comment. See [Copyright Header](https://github.com/TeraLogics/coding-standards/blob/master/copyright-header.md) for examples of appropriate format in
various programming languages. The header **MUST** contain the following components:

```
Copyright (C) <year> TeraLogics. All Rights Reserved.
```

The value of `<year>` should be selected when the file is created. It is not necessary to update the value of year in the future.

### **`'use strict';`**
Every file **SHOULD** have `'use strict';` before the first non-comment line. At a minimum, every outermost function scope **MUST** have `'use strict';` as the first
non-comment line, or **MUST** be inheriting strict mode from the file. This statement places the ECMAScript 5 interpreter into
[strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode). ECMAScript 6+ module code
[is always executed in strict mode](http://www.ecma-international.org/ecma-262/6.0/#sec-strict-mode-code), but the `'use strict';` statement produces no negative effect.

For server-side JavaScript, this example is the boilerplate from which all files should be produced.

```js
/*
 * Copyright (C) 2016 TeraLogics, LLC. All Rights Reserved.
 */

'use strict';

// file contents
```

For client-side JavaScript, the `'use strict';` statement will be at the top of the outermost function scope.

```js
/*
 * Copyright (C) 2016 TeraLogics. All Rights Reserved.
 */

(function () {
	'use strict';

	// file contents
})();
```

### **Single-Quoted Strings**
In JavaScript, either single quotes, double quotes or back ticks (`` ` ``) are valid delimiters for string literals. All code **SHOULD** use single quoted strings.

In situations where the string literal contains a significant plurality of single quotes, double-quoted or back-ticked strings **MAY** be used.

```js
var story = "Steve's father's sister's cousin's favorite uncle's trainer's massage therapist.";
```

All SQL statements **MUST** be written using double-quoted strings. See [SQL Query Formatting and Execution](#sql-query-formatting-and-execution).

```js
var sql = [
	"SELECT name, age",
	"FROM people",
	"WHERE department = 'gym'"
];
```

ECMAScript 6 [Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) require the use of back-ticked strings.

```js
var name = 'Chris',
	state = 'rambunctious';

console.log(`${name} is ${state}!`); // Chris is rambunctious!
```

### **Indentation Character**
All files **MUST** be indented with a single tab (`\t`) character per indentation level unless the format of the file explicitly forbids it. Where possible, the code editor **SHOULD** be
configured to display the indent as four (4) spaces. Hanging and other formatting indentation (often automated by the IDE) using space is permitted.

### **Equality Operators `===` and `!==`**
All equality and inequality comparisons **MUST** be performed with the strict equality operator (`===`) and the strict inequality operator (`!==`), respectively.
These operators will return `true` only if the two operands' values are of the same type _and_ are equivalent. The equality operator (`==`) and inequality operator (`!=`) will perform
type coercion (implicit type casting) to arrive at a common type before performing the comparison. In _almost all_ cases, comparisons are explicitly intended to operate on two values
of the same type. Using the triple-equals operator can help find logic or process errors and will more explicitly state the intention of the statement.

This example shows some of the differing behaviours of the strict and non-strict comparison operators.

```js
1    == '1'        // true
0    == false      // true
null == undefined  // true
1    != true       // false

1    === 1         // true
1    === '1'       // false
0    === false     // false
1    !== true      // true
null === undefined // false
```

### **Truthy and Falsy**
In JavaScript, a **falsy** value is a value that translates to `false` when evaluated in a Boolean context. All values not defined as **falsy** are **truthy** and will translate
to `true` when evaluated in a Boolean context. All of the following values are falsy:

```js
false
0 // zero
"" || '' || `` // empty string
null
undefined
NaN
```

All other values are truthy. It is important to consider falsy values when writing certain kinds of code.

When providing default values for omitted inputs, be aware of the falsy values
present in the data type of the inputs. In the final call in the example, below, even though non-`null` / `undefined` values are passed in, because zero (`0`) and `false`
are both falsy, the defaulting code is unintentionally triggered. In these cases, other methods of detecting unspecified inputs and assigning default values **MUST** be used.

```js
function report(count, flag) {
	count = count || 35;
	flag = flag || true;
	
	console.log(`count=${count} flag=${flag}`);
}

report();         // count=35 flag=true
report(12, true); // count=12 flag=true
report(0, false); // count=35 flag=true
```

When a function is designed to return a Boolean value, care must be taken to take into account how values are returned from Boolean statements. In the following example, several conditions
are composed to produce the return value. Because an empty string (`''`) and zero (`0`) are falsy, the possible return types for this function are `undefined`, `Number`, `String` and `Boolean`!

```js
function check(list, name) {
	return list && list.length && name && list[0] === name;
}

check();                   // undefined
check([]);                 // 0 (zero)
check(['Chris'], '');      // '' (empty string)
check(['Chris'], 'Chris'); // true
```

In order to ensure that the only type that is returned from this function is `Boolean`, all of the conditional components must be cast from truthy or falsy to `Boolean`.

#### **Casting To Boolean With `!!` and `!`**

The type of the return value of the unary logical [not operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_NOT_(!)) `!` will always
be `Boolean`. Applying the `!` operator to any falsy value will return `true`. Similarly, applying the `!` operator _twice_ will effectively cast any truthy or falsy value to `true` or `false`,
respectively. Given that, the above `check(list, name)` function **MUST** be rewritten as follows to ensure that the only possible return type is `Boolean`.

```js
function check(list, name) {
	return !!list && !!list.length && !!name && list[0] === name;
}

check();                   // false
check([]);                 // false
check(['Chris'], '');      // false
check(['Chris'], 'Chris'); // true
```

### **Identifier Naming**
The following conventions **SHOULD** be used when selecting names for identifiers:

* **Data** - Identifiers for inert data should be nouns. If the data is a collection, the noun should be plural.
* **Functions** - Identifiers for functions should be verbs or verb phrases. If the function returns a Boolean value, the function name should start with a prefix like `is`, `has`, `should`, `can`, etc.

```js
var employee = {
		name: 'Chris',
		age: 33
	},
	employees = [employee];

function isOld(employee) {
	return employee.age > 30;
};

function shouldTerminate(employee) {
	return !employee.name || employee.name.length % 3 === 0;
};

var toTerminate = _.find(employees, function (employee) {
	return isOld(employee) || shouldTerminate(employee);
});
```

### **Accessing Globals**
The global space **SHOULD** be used sparingly, but in some cases can greatly simplify code. In those cases, certain conventions can be used to improve readability and
maintainability by making interactions with the global space explicit.

#### **Server-Side Node.js Code**
In server-side Node.js code, all variables in the global space **MUST** be accessed through the `global` object. There are a handful of exceptions to this rule that reduce
code noise for very-commonly-used components. When any of the following are present in the global space, it is not necessary to access them with the `global` prefix:

* `Promise` ([bluebird](http://bluebirdjs.com/docs/api-reference.html))
* `path`
* `TLError`
* `TLUtil`
* `_` ([Underscore.js](underscorejs.org))

In ECMAScript 6, native promise support was added to JavaScript; a global `Promise` object is already defined in that version of the language. We explicitly overwrite the native
promise with **bluebird** in the global space, as indicated above.

In _most_ of our applications, additions to the global space will be made towards the top of the main/index file for the application.

#### **Client-Side Code**
In client-side code, two [JSHint](http://jshint.com/docs/) directives **MUST** be used to indicate, for each file, what globals are consumed and what globals are exported.

In this example, the file indicates that it expects the presence of two globals (`One` and `Two`) and exports two other globals (`Three` and `Four`).

```js
/* globals One, Two */

/* exported Three, Four */
```

### **Using `require` In Node**
Most of our applications define various globals for the absolute paths to the folders containing the files for application layers and libraries. When using `require` to import
the code from files in those locations, [`path.join`](https://nodejs.org/api/path.html#path_path_join_path1_path2) **MUST** be used to provide the necessary path.

```js
var carsDal = require(path.join(global.__dalsdir, 'cars'));
```

### **Using `parseInt`**
When using the [`parseInt(string, radix)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt) function, the `radix` parameter **MUST** be provided.
When the `radix` parameter is not provided, the behavior is determined by the value of the `string` parameter. In _almost all_ cases, the desired behavior is known at the
point when the code is written.

```js
parseInt('42', 10);
```

### **Constructing and Raising Errors**
Throughout the application code, errors _produced_ by our code **MUST** use the `TLError` class. For detailed technical information about the `TLError` class, see the
README in the **tl-error** module repository.

```js
throw new TLError('Customized message about the error.', TLError.ErrorType);
```

When creating a new error to indicate a failure state, in _most_ cases, that error should simply be thrown. Select a descriptive error message and select an error type
that is appropriate for the kind of failure. **Be aware that the text of the error message will be transmitted to the client for most requests.**

In some cases, when interacting with libraries or third-party modules, a failure raised by that library or module needs to be handled. In that situation, the error **SHOULD** be
wrapped in a `TLError` by providing it to the constructor's `innerError` parameter. In _most_ cases, the error type should be `TLError.Internal` to indicate that error occurred
outside of the basic mechanisms of data exchange. Doing so has two benefits: **(1)** a friendly error message can be returned to the client; and, **(2)** the low-level details of
the application are concealed.

```js
return lib.doSomething().then(function (result) {
	// process result
}).catch(function (err) {
	// handle the library error
	throw new TLError('Could not do something.', TLError.Internal, err);
});
```

Most of our applications have terminal error handling fallback mechanisms. In these mechanisms, if an error is received that is not an instance of `TLError`, it will
be automatically wrapped with a generic error message and an error type of `TLError.Internal` as a failsafe to ensure that low-level details are protected. Because of this, it is
not _explicitly_ necessary to wrap errors manually, but doing so allows the customization of the message which will improve the client's experience.

When instances of `TLError` are logged, in most of our applications, the `fullStack` property will be used which provides a stack trace that is aware of the chain of `innerError`s.
The outer error and stack trace is followed by each inner error, indented progressively farther for each level of nesting.

### **Logging Errors**
In some cases, errors need to be logged outside of the standard process for error logging in the course of a client request. These errors may be: **(1)** handled, but not
intended to be passed all the way out to the client response; or, **(2)** caught during a process unrelated to an interaction with a client. In these cases, the `TLUtil.logError`
function **MUST** be used. For detailed technical information about the `logError` function or other functions in `TLUtil`, see the README
in the **tl-util** module repository.

The purpose of the `logError` function is to normalize the output for logged errors. Here are some examples of how it might be used.

```js
.catch(function (err) {
	TLUtil.logError(err); // logs error information at the ERROR level
	TLUtil.logError(err, console.notice); // logs error information at the NOTICE level	
});

.catch(TLUtil.logError(console.debug)); // provides a function to `catch` that will log the error at the DEBUG level

.catch(TLUtil.logError); // logs the error at the ERROR level
```

### **Promises**
All of our applications use Promises to control the flow of asynchronous operations. A full overview of and guide for using Promises is outside the scope of this document; however,
there are a few guidelines to follow when using them. 

#### **Promise Structure**
One of the issues that Promises are intended to solve is Callback Hell. In Callback Hell, code marches progressively rightward as each asynchronous action's handler executes another
asynchronous action requiring and handler and so on. It is perfectly possibly to transition nodeback-style code to Promise code while preserving Callback Hell.

In the first example, nodebacks are simply replaced with promises, one-for-one. In the second example, the promises returned by `secondTask` and `thirdTask` are returned from the
`then` function. Promises **MUST NOT** be structured in this way.

```js
firstTask().then(function () {
	secondTask().then(function () {
		thirdTask();	
	});
});
```

If a _value_ is returned from a `then` function, that value becomes the outer promise's resolution value. If a _promise_ is returned from a `then` function, its
entire chain is inserted into the outer promise's chain at the location of the `then`. In this way, the next `then` will actually handle the result of the promise returned from
the previous `then`, but on the outer level. Promises **MUST** be structured in this way.

```js
firstTask().then(function () {
	return secondTask();
}).then(function () { 
	return thirdTask();
});
```

In this example the actual chain of execution will be `p1 -> p2 -> p3 -> p4 -> p5`. The `p2 -> p3 -> p4` chain is returned from `p1`'s `then` function and so is executed between `p1` and `p5`.

```js
p1().then(function () {
	return p2().then(function () {
		return p3();
	}).then(function () {
		return p4();
	});
}).then(function () {
	return p5();
})
```

Following the guideline above, this promise chain should actually be written as the following:

```js
p1().then(function () {
	return p2();
}).then(function () {
	return p3();
}).then(function () {
	return p4();
}).then(function () {
	return p5();
})
```

An exception to that kind of structure is when a subordinate promise chain should be executed conditionally. In this example, there are two possible chains:

1. `p1 -> p2 -> p3 -> p5`
2. `p1 -> p4 -> p5`

The control-of-flow statements in `p1`'s `then` function switch between the two internal promise chains.

```js
p1().then(function () {
	if (something) {
		return p2().then(function () {
			return p3();
		});
	} else {
		return p4();	
	}
}).then(function () {
	return p5();
})
```

#### **The Promise Constructor vs. The Deferred Pattern**
When interacting with asynchronous code outside of your control, it may be necessary to translate some sort of callback-based asynchrony to promises in order to integrate
it with the rest of an application's code. In this case, there are two main possibilities:

* Using [`promisify`](http://bluebirdjs.com/docs/api/promise.promisify.html) or [`promisifyAll`](http://bluebirdjs.com/docs/api/promise.promisifyall.html). In many cases, these two
  utilities will be sufficient translators.
* Manual translation using the Promise constructor.

In this example, a callback-based library call is translated using a wrapper function that exposes its functionality to the application code as a promise.

```js
function getWeather() {
	return new Promise(function (resolve, reject) {
		weatherlib.getWeather(function (err, weather) {
			if (err) {
				return reject(err);			
			}
			
			resolve(err);
		});	
	});
}
```

In code examples, you may see a `deferred` object being used for this kind of translation. This pattern is deprecated and **MUST NOT** be used. The Promise constructor **MUST** be
used in its place.

##### **The Explicit Construction Anti-Pattern**
In this example, the deprecated deferred pattern is used to interact with an inner promise.

```js
function getCars() {
	var deferred = Promise.defer();
	
	carsDal.getCars().then(function (result) {
		deferred.resolve(result);	
	}).catch(function (err) {
		deferred.reject(err);
	});
	
	return deferred.promise;
}
```

This kind of code is an anti-pattern because there is no need to wrap the call to `carsDal.getCars` as it already returns a promise. The result of that promise can simply be
returned from the wrapping `getCars` function as in this example.

```js
function getCars() {
	return carsDal.getCars();
}
```

These two examples are functionally equivalent, though the second example is far less noisy without the unnecessary layers of indirection. As a consequence, the second example
is easier to read and maintain, and represents the structure to which this kind of code **MUST** conform.

#### **Starting a Promise Chain**
In _most_ cases, it is the responsibility of the **adapter** layer to begin the promise chain. More than that, it is critical for other code interacting with functions on the
**adapter** layer _and_ the **DAL** layer that they always return a promise. If a library function that returns a promise cannot be used as the point of origin, there are several
options in the **bluebird** library for creating that initial promise. Most of the functions listed here are more flexible than the single use case described. See the linked
documentation for more in-depth details.

* [`Promise.resolve(Promise<any>|any value)`](http://bluebirdjs.com/docs/api/promise.resolve.html) and [`Promise.reject(Promise<any>|any value)`](http://bluebirdjs.com/docs/api/promise.reject.html)    
  These two functions are generally used to use a simple value to create an automatically-resolved or -rejected promise, respectively.
      
    ```js
    return Promise.resolve('Chris'); // returns a promise resolved with the value 'Chris'
    ```
        
* [`Promise.try(function() fn)`](http://bluebirdjs.com/docs/api/promise.try.html)    
  `Promise.try` will take a function and use it to begin a promise chain. Any values or promises returned from this function will continue the chain. Any exceptions raised
  will be turned into rejections on the returned promise.    
  
    ```js
    function vibrate(seconds) {
    	return Promise.try(function () {
    		if (seconds > 10) {
    			throw new TLError('These are not good vibrations.', TLError.InvalidArgument);
    		}
    		
    		return viblib.vibrate(seconds);
    	})
    }
    ```
        
* [`new Promise(function(function resolve, function reject) resolver)`](http://bluebirdjs.com/docs/api/new-promise.html)    
  The Promise constructor can also be used to begin a promise chain. See [**The Promise Constructor vs. The Deferred Pattern**](#the-promise-constructor-vs-the-deferred-pattern)
  for more information
  
There are other ways that **bluebird** can be used to begin a promise chain, but these will be the most common.

## **Application Layers**
Application code is divided into layers. Each layer is responsible for a certain portion of the handling of a request.

| Layer | Description |
|------:|-------------|
| [**Route**](#route-layer) | The **route** layer (or "resources" layer) is responsible for defining the text of each path and connecting that path to its handler function in the **controller** layer.
| [**Controller**](#controller-layer) | The **controller** layer is responsible for handling the request passed to it by the **route** layer. The request is translated into a tuple that passes the data on to the **DAL** layer.
| [**DAL**](#dal-layer) | The **Data Abstraction Layer** (...layer) is responsible for validating the contents of the tuple it is provided by the **controller** layer. It then coordinates calls to whatever functions in the **adapter** layer are required to handled the request.
| [**Adapter**](#adapter-layer) | The **adapter** layer (or "model" layer, in UV) is the lowest layer, interfacing directly with the data store.

### **Route** Layer
Functionality is organized into files at this layer by the first path component. If a route's path is `/cars/15`, it would be defined in a file named **cars.js**.
That file **MUST** export a function that takes, at a minimum, a reference to the Express application. Different projects may also take other parameters.
Look at existing routes to see what is available.

```js
module.exports = function (app) {
	// routes...
};
```

For each route, first define the path, then define the supported HTTP verbs. The verb definition will include, at least, the function in the **controller** layer
that will handle that path/verb combination.

```js
var carsCtrl = require(path.join(global.__ctrldir, 'cars'));

module.exports = function (app) {
	app.route('/cars')
		.get(carsCtrl.search);

	app.route('/cars/:cid')
		.get(carsCtrl.get)
		.delete(carsCtrl.delete);
};
```

In some cases, multiple Express [middlewares](http://expressjs.com/guide/using-middleware.html) (see that link for more information) may be composed to chain multiple
layers of processing together such as in the following example where access to a route is gated through several permission-checking middlewares:

```js
module.exports = function (app) {
	app.route('/feeds/:fid')
		.delete(
			middleware.loadFeedPermissions(['modify']),
			middleware.hasFeedPermission(['modify'], {
				idParam: 'fid'
			}),
			feedsCtrl.removeFeed
		);
};
```

#### **HTTP Verbs**
Each route **MUST** be defined under the appropriate HTTP verb. This table describes the meaning and use of the HTTP verbs that our REST APIs use.

| Verb | Operation | Details |
|------|-----------|---------|
| `GET` | Get, Fetch, Search | `GET` is used to retrieve data. |
| `PUT` | Add, Insert | `PUT` is used to create a resource. |
| `POST` | Update | `POST` is used to update a resource. |
| `PATCH` | Toggle, Action | `PATCH` is used to modify a single property of a resource or execute an action. |
| `DELETE` | Remove, Delete | `DELETE` is used to delete a resource. |

#### **API Documentation**
The **route** layer is also the layer in which the API documentation is stored. The documentation for each path/verb combination **SHOULD** be placed
above the verb definition. Any common/shared documentation items/constructs **MAY** be placed above the first call to `app.route` or outside
of the `module.exports` line entirely.

```js
/* common documentation items */

module.exports = function (app) {
	app.route('/cars/:cid')
		/* documentation for GET /cars/:cid */
		.get(carsCtrl.get)
		/* documentation for DELETE /cars/:cid */
		.delete(carsCtrl.delete);
};
```

### **Controller** Layer
The organization of the **controller** layer follows that of the **route** layer. The **cars.js**, above, will have a corresponding **cars.js** in this layer.
Each function in this layer is an Express [middleware](http://expressjs.com/guide/using-middleware.html) (see that link for more information) that is used
to handle the request after it is routed by the **route** layer. The `carsCtrl.search` function, referenced above, might be defined like this:

```js
var carsDal = require(path.join(global.__dalsdir, 'cars'));

exports.search = function (req, res, next) {
	carsDal.search({               // call the DAL layer
		make: req.query.make,
		model: req.query.model,
		year: req.query.year
	}).then(function (results) {
		res.json(results);         // send the results as JSON
		// ...or...
		res.render('viewname', {   // render a view with the results
			cars: results
		});
		return next();
	}).catch(next);
};
```

It is this layer's responsibility to: **(1)** perform any implementation-specific data-type validations on incoming data from the request, and then transform values confirmed
be type-compatible; **(2)** pass the normalized data as a tuple to the **DAL** layer; **(3)** pass the output of the **DAL** layer back to the client via the response in a manner
appropriate to the route; **(4)** catch any errors raised by the **DAL** layer and pass them on to be handled; and, **(5)** complete the promise chain.

After passing data to the response, `next` **MUST** be called to ensure that the middleware chain will continue.

```js
res.json(results);
return next();
```

In the catch function, `next` **MUST** be called, passing the error as the first parameter, so that it will be properly handled by the error-handling middleware down the line.
Don't wrap the `next` function in an anonymous function if the only statement within it would be `next(err)`; however, if wrapping/producing a new error provides additional
beneficial information, a more complex catch block may be constructed. See [Constructing and Raising Errors](#constructing-and-raising-errors).

```js
catch(next)
```

#### **Route Parameters and Query Parameters**
As the direct handler for our routing system, as defined in the **route** layer, the **controller** layer is tightly coupled to _how_ we implement client communication.
In a REST service, Route parameter (`/route/:param`) and query parameter (`/route?query=param`) values will always be strings. It is the responsibility of this layer to obfuscate
or shield the lower layers from this implementation detail.

At this layer, the data type compatibility of these kinds of parameters **MUST** be evaluated, and if the `String` value is compatible with the target type, it **MUST** be
converted to that type before it is assigned to the data tuple. In this example, the controller's `get` function is handling a request for the route `/cars/:cid`. The full request
URL was `/cars/15?extended=true`; it includes a car ID that is intended to be `Number` and an `extended` flag that is intended to be `Boolean`.

```js
var carsDal = require(path.join(global.__dalsdir, 'cars'));

exports.get = function (req, res, next) {
	var obj = {
		cid: req.params.cid,
		extended: req.query.extended
	};

	validation.parameters(function () {
		if (!obj.cid || !validation.int(obj.cid)) {
			return 'Invalid cid; it must be an integer.';
		} else {
			obj.cid = parseInt(obj.cid, 10);
		}
		
		if (obj.extended && obj.extended !== 'true' && obj.extended !== 'false') { // validation.boolean is too permissive for this
			return 'Invalid extended; it must be a Boolean value.';
		} else {
			obj.extended = obj.extended === 'true';
		}
	}).then(function () {
		return carsDal.get(obj);
	}).then(function (results) {
		res.json(results);
		return next();
	}).catch(next);
};
```

In _most_ cases, the **tl-validation** module can be used to determine type compatibility; in other cases, such as the case above, its functions may be too permissive for
the specific check being performed. See the README on the **tl-validation** module repository for more information.

### **DAL** Layer
The organization of the **Data Abstraction Layer** follows that of the **controller** layer. The **cars.js**, above, will have a corresponding **cars.js** in this layer.
Each function of this layer receives the tuple from the **controller** layer. The `carsDal.search` function, referenced above, might be defined like this:

```js
var util = require('util'),
	carsAdapter = require(path.join(global.__adaptersdir, 'cars'));

exports.search = function (obj) {
	return validation.parameters(function () {  // start the promise chain and validate
		if (util.isNullOrUndefined(obj.make) && util.isNullOrUndefined(obj.model) && util.isNullOrUndefined(obj.year)) {
			return 'One of make, model or year is required.';
		}

		if (obj.year && !_.isNumber(obj.year)) {
			return 'Invalid year; it must be an integer.';
		}
		
		// ... validate that `make` and `model` are strings ...
	}).then(function () {
		return carsAdapter.search(obj);  // call the adapter layer and return the result
	};

	// no error handling on this layer, unless wrapping/producing a new error provides additional beneficial information
};
```

It is this layer's responsibility to: **(1)** perform strict type validation the information coming in via the tuple; **(2)** pass the validated and type-correct data as
a tuple to the **adapter** layer; and, **(3)** return the result of calling the **adapter** layer.

All functions exported from the **DAL** layer **MUST** return a promise of some kind. The **controller** layer relies on this interface contract for continuation and for
error handling.

```js
exports.search = function (obj) {
	// MUST return a promise
};
```

If `validation.parameters` is not available, **bluebird**'s `try` function can be used in its place.

```js
return Promise.try(function () {
	// do stuff
}).then(/* whatever */);
```

If a single value _must_ be returned outside of the scope of the validation, it can be returned with **bluebird**'s `resolve` / `reject` functions to ensure that it
is wrapped in a promise. While anything can be provided as a rejection reason, as in the first example, above, in most cases, the rejection value **SHOULD** be
at least an instance of `Error` and, in most cases, an instance of `TLError`. See [Starting a Promise Chain](#starting-a-promise-chain) for more information.

```js
return Promise.reject('Something bad.');
// ...or...
return Promise.reject(new TLError('Something else bad.', TLError.PaymentRequired));
```

In this layer, in _almost all cases_, validation **SHOULD** be type strict. By the time data is passed to a DAL function, it should already conform to the type
specified by that function's data structure contract. The easiest way to perform type checks is by using the `is_____(value)` functions from
[Underscore.js](http://underscorejs.org/#isEqual). One gotcha to be aware of is that while all `Array`s are `Object`s, not all `Object`s are `Array`s.

There **SHOULD** be no error handling on this layer unless it is specifically necessary for the implementation of the route.

### **Adapter** Layer
The organization of the **adapter** layer follows that of the **Data Abstraction Layer**. The **cars.js**, above, will have a corresponding **cars.js** in this layer.
Each function of this layer receives the tuple from the **DAL** layer. The `carsAdapter.search` function, referenced above, will be implemented
differently, depending on what data store is being used.

It is this layer's responsibility to: **(1)** perform any final checks to ensure that the incoming parameters can be passed to the data store; **(2)** formulate
a query appropriate to the data store that will fetch the desired data; and, **(3)** obfuscate any data-store-specific structures from the other layers by returning
only the data requested. See [Constructing and Raising Errors](#constructing-and-raising-errors).

```js
exports.search = function (obj) {
	// MUST return a promise
};
```
As with the **DAL** layer, all functions exported from the **adapter** layer **MUST** return a promise of some kind. The **DAL** layer relies on this interface contract
for continuation and for error handling.

The **adapter** layer **MUST NOT** be concerned with the restriction of output based on business rules. Its job is to provide unopinionated access to the information
of the resource being accessed. It is the job of higher layers (generally the **DAL** layer -- sometimes the **controller** layer) to perform selective omission or
transformation of that data such that it fits the business requirements of the higher-level action being performed.

## **Data Retrieval**
These are standards and practices related to data retrieval. For most of our applications, the data store is PostgreSQL.

### **SQL Query Formatting and Execution**
Maintaining the string literals that represent SQL queries can be difficult without a standard for formatting. SQL queries **SHOULD** be constructed and formatted as follows:

```js
var sql = [
	"SELECT",
	"  e.employee_id AS id,",
	"  e.name,",
	"  e.position,",
	"  CASE",
	"    WHEN e.rank = 0 THEN 'Fearless Leader'",
	"    WHEN e.rank = 1 THEN 'Manager'",
	"    ELSE 'Underling'",
	"  END AS rankname,",
	"  d.department_name as department",
	"FROM employee e",
	"INNER JOIN department d",
	"  USING(department_id)"
];
```

Double quotes **MUST** be used for strings containing SQL queries. See [Single-Quoted Strings](#single-quoted-strings). Structural formatting indentation within the string **MUST** be
done using two (2) spaces per indentation level. Trailing spaces **MUST NOT** be present on lines.

To compile the query to be sent to the data store, it is joined using the space character. Additional arrays can be used to compartmentalize the query or to dynamically build
the where clause. Our applications have lots of examples of query builds of varying complexities; look around for good examples of how best to handle different cases.

```js
pg.singleQuery(/* connection string */, sql.join(' '), params)
```

### **Pagination and Sorting**
When a large amount of data is to be returned to the client, pagination and sorting **SHOULD** be implemented. Four additional parameters **MUST** be accepted to control
the pagination and sorting behavior.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sortby` | String | _varies_ | Indicates by which property the results should be sorted. A default value **SHOULD** be selected on a case-by-case basis.<br/>This parameter **MUST** accept the same property names as are returned in the output data structures.<br/>This parameter **MUST** be validated in the **adapter** layer to ensure only valid values are accepted. See [Validating the `sortby` Parameter](#validating-the-sortby-parameter). |
| `sortdirection` | String | `'ASC'` | Indicates the direction in which to sort by the selected property.<br/>This parameter **SHOULD** use the values `'ASC'` and `'DESC'`, normalizing the input for case. |
| `limit` | Number | `100` | The maximum number of records to retrieve. |
| `offset` | Number | `0` | The number of records to skip. |

The output of a paginated response **MUST** conform to the following data structure:

```js
{
	total: Number,
	filteredTotal: Number,
	data: Object[]
}
```

| Property | Description |
|----------|-------------|
| `data` | The array of response data objects conforming to the constraints described by the request. |
| `filteredTotal` | The total number of records matching the constraints described by the request. |
| `total` | The total number of records. |

This data structure can easily be created using [`Promise.props`](http://bluebirdjs.com/docs/api/promise.props.html).

```js
return Promise.props({
	data: Promise<Object[]>,
	filteredTotal: Promise<Number>,
	total: Promise<Number>
});
```

The resulting promise will be resolved with an object whose property values are the resolution values of the promises assigned to those properties.

#### **Validating the `sortby` Parameter**
The easiest way to validate the `sortby` parameter is to maintain a map in the **adapter** layer that will both validate and translate incoming values, producing their data-store-level
equivalent names (which may or may not be different).

```js
const SEARCH_VALID_SORT = {
	id: 'recording_id',
	nid: 'nid',
	feedname: 'feed_name',
	starttime: 'start_time',
	endtime: 'end_time'
};
```

The incoming value can easily be validated using this map.

The `sortby` parameter should be validated and translated on the **adapter** layer as that layer is the custodian of: the data-store-specific sorting implementation; and, the mapping
between the external-contract resource data structure and its back-end storage details.

```js
exports.search = function (obj) {	
	if (obj.sortby && !SEARCH_VALID_SORT[obj.sortby.toLowerCase()]) {
		// validate the sortby parameter using the map and provide an informative error message for the client
		return Promise.reject(new TLError('Invalid sortby; it must be one of: ' + _.keys(SEARCH_VALID_SORT).join(', '), TLError.InvalidArgument));
	}
	
	// other code
	
	if (obj.sortby) {
		// map the input to the correct value for the data store
		var sort = "ORDER BY " + SEARCH_VALID_SORT[obj.sortby];
		
		// ...
	}
	
	// other code
};
```

### **Empty Result vs. Not Found**
In the following examples, the **adapter** layer will be communicating with a PostgreSQL database.

In this example, the application has received a request that is expected to return zero or more records. At the **adapter** layer, it is not necessary to perform any
conditional logic based on the number of rows returned. The `rows` array will always contain zero or more records. The resulting array, empty or not, should be returned
_all the way up the stack_.

```js
// Controller layer
exports.search = function (req, res, next) {
	carsDal.search({
		// get parameters from the request
	}).then(function (cars) {
		// return results to the client

		return next();
	}).catch(next);
};

// DAL layer
exports.search = function (obj) {
	// validation omitted

	return carsAdapter.search(obj);
};

// Adapter layer
exports.search = function (obj) {
	return pg.singleQuery(/* connection string */, /* query */).then(function (results) {
		return results.rows;
	});
};
```

**The final result of a request that may result in zero or more records MUST always be `200 OK`, regardless of how many records are to be returned.**

In this example, the application has received a request for a single record uniquely identified by some value. At the **adapter** layer, it is not necessary to perform any
conditional logic based on the presence or absence of that record. The value of `rows[0]` will be `undefined` if no record matches the criteria. The empty / `undefined` list
should be returned from the **adapter** layer and passed up through the **DAL** layer. In the **controller** layer, the absence of a record matching the criteria should finally
be translated into a not-found error.

```js
// Controller layer
exports.get = function (req, res, next) {
	carsDal.get({
		id: req.params.id
	}).then(function (car) {
		if (!car) {
			throw new TLError('Could not find car.', TLError.NotFound);
		}

		// return results to the client

		return next();
	}).catch(next);
};

// DAL layer
exports.get = function (obj) {
	return validation.parameters(function () {
		if (!obj.id || !validation.int(obj.id)) {
			return 'Invalid id; it must be an integer.';
		} else {
			obj.id = parseInt(obj.id, 10);
		}
	}).then(function () {
		return carsAdapter.get(obj);
	});
};

// Adapter layer
exports.get = function (obj) {
	return pg.singleQuery(/* connection string */, /* query */).then(function (results) {
		return results.rows[0];
	});
};
```

**The final result of a request for a single record by a unique identifier MUST be `404 Not Found` when the record is not found or `200 OK` when the record is found.**

### **Transactionalized SQL**
When multiple queries need to be executed in sequence and the success of the batch relies on the individual success of each query, those queries should be executed within a transaction.
There is a helper function in the **tl-postgres** library to aid in writing transactionalized SQL code.

```js
exports.doThings = function () {
	return pg.transaction(/* connection string */, function (client) {
		return pg.query(client, firstSql, firstParams).then(function () {
			return pg.query(client, secondSql, secondParams);		
		});
	});
};
```

Before calling the handler function provided to `pg.transaction`, a connection is established and a transaction is initiated. The handler function is then provided with
a client representing the open connection. That client can then be used by calls to `pg.query` to execute queries on that connection within the transaction. The handler function
**MUST** return a promise. If the promise returned from the handler function is resolved, then the transaction is committed and the outer promise chain is resolved with the inner
resolution value. If the promise returned from the handler function is rejected, then the transaction is rolled back, and the outer promise chain is rejected with the error that
caused the inner rejection.
