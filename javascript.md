# 1 JavaScript Style Guide


## 1.1 Default Settings

### 1.1.1 Indentation
Indentation should be achieved only with 4-space Soft Tabs (4 Spaces), or spaces for further alignment.

### 1.1.2 EOF
Set your editor to add a newline at end-of-file [(How do I do that?)](http://robots.thoughtbot.com/no-newline-at-end-of-file)

### 1.1.3 Line length
Set your editor or code formatter to wrap at 100 character line length

## 1.2 General Conventions

### Functions

- DO use a **space after** function keyword 

```js
// === no ===
var name = function() {};
someMethod(123, function() {
	return something;
});

// === yes ===
var name = function () {};	
someMethod(123, function () {
	return something;
});
```

### Object Literals

- DO use object literals when possible

```js
// === no ===
var myArray = new Array();
var myObj = new Object();

// === yes ===
var myArray = [];
var myObj = {};
```

### Commas

- DON'T use leading commas

```js
// === no ===
var cat = "woof"
	, dog = 2
	, fish = {};

// === yes ===
var cat = "woof",
	dog = 2,
	fish = {};		
```

### Variable Declaration

- DO declare vars at **top** of scope

```js
// === no === 
function (arg) {
	if (arg === 2) {
		var something = 1;
		return something;
    }
	var other = "other stuff";
}

// === yes ===
function (arg) {
	var something, other;
	
	if (arg === 2) {
		something = 1;
		return something;
    }
	other = "other stuff";
}
```

### Equality

- DO use === & !== instead of ==  & !=

```js
// === no ===
if (arg == 1 && something != "some") { // do stuff }
	
// === yes ===
if (arg === 1 && something !== "some") { // do stuff }
```

### Parentheses

- DONT mix parentheses usage in a statement

```js
// === no ===
if (arg === 1) { 
	return false; 
}
else return true;
	
// === yes ===
if (arg === 1) { 
	return false; 
} else {
	return true;	
}

if (arg === 1) return false;
```

### Naming Convention

- DO use camelCase everywhere.

```js
// === no ===
var fav_food = "cake";
function eat_some_cake() {}

// === yes ===
var favFood = "cake";
function eatSomeCake() {}
```

### Function / Method Documentation

- DO document all public methods with **jsDoc** syntax

```js
// === no ===
// returns a url bro
function url() {
    var p, cleaned = [], parts = _slice.call(arguments);
    while ((p = parts.shift())) {
        if (p) cleaned.push(encodePart(p));
    }
    return _apiRoot.concat(cleaned).join('/');
}


// === yes ===
/**
 * @name url
 * @description Build a valid API URL based on current configuration
 *
 * @param {...string} [pathPart] Endpoint parts without URI (eg. 'listings','store','1234')
 * @returns {String} Absolute API Endpoint URL
 */
function url() {
    var p, cleaned = [], parts = _slice.call(arguments);
    while ((p = parts.shift())) {
        if (p) cleaned.push(encodePart(p));
    }
    return _apiRoot.concat(cleaned).join('/');
}
```

### Hint Comments

- DO initial hint comments (TODO, FIXME, OPTIMIZE etc) comments

```js
// TODO: this needs to be refactored when something - DM
function url() {
	// FIXME: who are google, change this! - DM
	return "www.google.com";
}
```

### Promises

*Examples below use the [$q](https://docs.angularjs.org/api/ng/service/$q) AngularJS service*

- DO chain promises in order then -> catch -> finally and separate method calls to individual lines

```js
var deferred = $q.defer(),
	promise = deferred.promise;

// === no ===

promise.catch(function() {}).then(function(){}).finally(function (){})

// === yes ===

promise
	.then(function () {
		// stuff
	})
	.catch(function () {
		// more stuff
	})
	.finally(function () {
		// final stuff
	});
```	

- DO return the promise from the chain when inside of a function *rather then* creating another deferred promise & returning its promise

```js
// === no ===
function makePromise() {
    var d = $q.defer();

    someOtherPromiseFunc()
        .then(function (stuff) {
            var data = doThingsWithStuff(stuff);
            d.resolve(data);
        })
        .catch(function (err) {
            d.reject(err);
        });

    return d.promise;
}


// === yes ===
function makePromise() {
    return someOtherPromiseFunc()
        .then(function (stuff) {
            return doThingsWithStuff(stuff);
        });
}
```



##1.3 Testing

*Note: this section is still a work in progress.*

###1.3.1 Unit Tests

Unit tests should test the functionality of a single unit. In the case of Angular, a directive or a service could be considered a unit.

- Name the test file the same as the the unit under test, but with a .spec.js extension and store in the same folder as the unit i.e. services/cart-service.spec.js

- Test only the functionality of that unit - the logic of a function, given x and y  input the output should be z.

- Mock any external dependencies - any code that is not inside the unit should be mocked, stubbed or faked.

- Do not use changeable data like Date() or Math.random(), if needed they should be stubbed.

- Use jasmine 2.0 or above - jasmine 2.0 includes much improved [async testing](http://jasmine.github.io/2.0/introduction.html#section-Asynchronous_Support) support

- Describe and It blocks should generally use the following language

```js
describe('MyUnit', function () {
	describe('Add2 Method', function () {
	    it('should return 4 when 2 is passed as parameter', function () {});
	});
});
```

- Use xit() for pending tests - xit() tests do not need a body and will report a pending state when run.

## 1.4 Beautifying

If you are using Sublime Text, install the [JavaScript Beautify](https://github.com/enginespot/js-beautify-sublime) plugin and update your user package settings:

```js
{
    "indent_size": 4,
    "indent_char": " ",
    "indent_level": 0,
    "indent_with_tabs": false,
    "preserve_newlines": true,
    "max_preserve_newlines": 10,
    "jslint_happy": true,
    "brace_style": "collapse",
    "keep_array_indentation": false,
    "keep_function_indentation": false,
    "space_before_conditional": true,
    "break_chained_methods": false,
    "eval_code": false,
    "unescape_strings": false,
    "wrap_line_length": 100,

    // jsbeautify options
    "format_on_save": false,
    "use_original_indentation": false
}
```

Beautifier can also be run as a grunt task [grunt-jsbeautifier](https://www.npmjs.org/package/grunt-jsbeautifier) with the same configuration options
