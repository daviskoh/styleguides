# AngularJS Style Guide


## Overview
This guide was created for developers at Crowdsurge to provide consistency through good practices.
<br />

## Table of Contents

* [General Guidelines](#general-guidelines)
* [IIFE Scoping](#iife-scoping)
* [Modules](#modules)
* [Controllers](#controllers)
* [Factories](#factories)
* [Data Services](#data-services)
* [Manual Dependency Injection](#manual-dependency-injection)
* [Publish and subscribe events](#publish-and-subscribe-events)
* [Performance](#peformance)
* [Exception Handling](#exception-handling)
* [E2E Testing](#e2e-testing)

## General Guidelines

* Skinny controllers
    * Controllers should contain only the bare minimum code, generally use them just for connecting the `$scope` to any needed service methods and for maintaining the state of the view.
* No business logic in controllers
    * Move that into a service, its easier to test services than it is controllers.
* No $http calls in controllers
* Always use singletons for services
* Only interact with the DOM via directives
* Use `$watch()` sparingly
    * When you have to use it, make sure the expression and the action are simple functions that evaluate quickly.
* Use `$emit`, `$broadcast` `$on` to communicate between components
* One component per file
* One unit test file per component
* Avoid JQuery - its fat and often unnecessary
* Stay DRY :)

## IIFE Scoping
To avoid polluting the global scope with our function declarations that get passed into Angular, ensure build tasks wrap the concatenated files inside an **Immediately-Invoked Function Expression**

Avoid

```javascript
// logger.js
angular
  .module('app')
  .factory('logger', logger);

// logger function is added as a global variable  
function logger () { }

// storage.js
angular
  .module('app')
  .factory('storage', storage);

// storage function is added as a global variable  
function storage () { }
```
Recommended

```javascript
// no globals are left behind 

// logger.js
(function () {
  angular
    .module('app')
    .factory('logger', logger);

  function logger () { }
})();

// storage.js
(function () {
  angular
    .module('app')
    .factory('storage', storage);

  function storage () { }
})();
 ```
**Note:** The rest of the examples in this guide may omit the IIFE syntax but besure to use it in your code.

## Modules
1. Declare modules without a variable using the setter and getter syntax and keep 1 component per file.
2. Using `angular.module('app', []);` sets a module, whereas `angular.module('app');` gets the module. **Only set once and get for all other instances.**
3. Pass functions into module methods rather than assign as a callback
4. Use named functions instead of passing an anonymous function in as a callback.

```javascript
// app.module.js
angular.module('app.controllers', []);
angular.module('app.factories', []);
angular
    .module('app', ['app.controllers','app.factories','ngRoute']);    
```
```javascript
// someController.js
angular
    .module('app.controllers')
    .controller('SomeController' , SomeController);

function SomeController() { }
```
```javascript
// someFactory.js
angular
    .module('app.factories')
    .factory('someFactory' , someFactory);

function someFactory() { }
```

## Controllers
**Controllers = Classes**  
Angular makes it really easy to add an inline controller as an anonymous function, **but just because you can doesn't mean you should**. Inline functions encourage you to shove everything into `$scope`, but defining your controller as a class and using the prototype to add functionality helps to keep you honest.

Avoid

```javascript
angular
    .module('app')
    .controller('MainCtrl', function ($scope) {
        $scope.doStuff = function () {
            //Really long function body
        };
    });
```
Recommended

```javascript
angular
    .module('app')
    .controller('MainCtrl', MainCtrl);

MainCtrl.$inject = ['$scope'];

function MainCtrl($scope) {
    var vm = this;
    vm.title = 'test';
    vm.returnTitle = returnTitle;
    vm.stuff = vm.doStuff();

    function returnTitle() {
        return vm.title;
    }

};

MainCtrl.prototype.doStuff = function () {
    // Really long function body
};
```
**Inheritance**  
// TODO

**controllerAs syntax**  
Controllers are classes, so use the controllerAs syntax at all times.

Avoid

```html
<div ng-controller="MainCtrl">
  {{ someObject }}
</div>
```
Recommended

```html
<div ng-controller="MainCtrl as vm">
  {{ vm.someObject }}
</div>
```
* In the DOM we get a variable per controller, which aids nested controller methods, avoiding any `$parent` calls

* The controllerAs syntax uses `this` inside controllers, which gets bound to `$scope`

Avoid

```javascript
function MainCtrl ($scope) {
  $scope.someObject = {};
  $scope.doSomething = function () {

  };
}
```
Recommended

```javascript
function MainCtrl () {
  var vm = this;
  vm.someObject = {};
  vm.doSomething = function () {

  };
}
```

* Only use `$scope` in controllerAs when necessary; for example, publishing and subscribing events using `$emit`, `$broadcast`, `$on` or `$watch`. Try to limit the use of these, however, and treat `$scope` as a special use case

**controllerAs 'vm'**  
Capture the this context of the Controller using vm, standing for ViewModel

Avoid

```javascript
function MainCtrl () {
  this.doSomething = function () {

  };
}
```

Recommended

```javascript
function MainCtrl (SomeService) {
  var vm = this;
  vm.doSomething = SomeService.doSomething;
}
```

**Note:** You can avoid any jshint warnings by placing the comment below above the line of code.

```javascript
/* jshint validthis: true */
var vm = this;
```

Why?: The `this` keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of `this` avoids encountering this problem.

**controllerAs in Routing**  

Using Angular Route

```javascript
$routeProvider
.when('/',{
  templateUrl: 'foo.html',
  controller: 'MainCtrl',
  controllerAs: 'vm'
});
```
Using UI Router

```javascript
$stateProvider.state('base.main', {
  templateUrl: "/templates/base/main.html",
  url: '/',
  controller: 'MainCtrl as vm'
});
```

**Presentational logic only (MVVM)**  
Presentational logic only inside a controller, avoid Business logic (delegate to Services).

Avoid

```javascript
function MainCtrl () {

  var vm = this;

  $http
    .get('/users')
    .success(function (response) {
      vm.users = response;
    });

  vm.removeUser = function (user, index) {
    $http
      .delete('/user/' + user.id)
      .then(function (response) {
        vm.users.splice(index, 1);
      });
  };

}
```
Recommended

```javascript
function MainCtrl (UserService) {

  var vm = this;

  UserService
    .getUsers()
    .then(function (response) {
      vm.users = response;
    });

  vm.removeUser = function (user, index) {
    UserService
      .removeUser(user)
      .then(function (response) {
        vm.users.splice(index, 1);
      });
  };

}
```
Why?: Controllers should fetch Model data from Services, avoiding any Business logic. Controllers should act as a ViewModel and control the data flowing between the Model and the View presentational layer. Business logic in Controllers makes testing Services impossible.

**Bindable Members at the bottom**  
Place bindable members at the bottom of the controller before the return and logic code.

```javascript
function Sessions() {

    // variable declaration (and maybe assignment)

    var vm = this,
        lastLogin, timeSinceLastLogin;

    // function declaration

    function gotoSession() {}

    function refresh() {
        loginDate = new Date();
    }

    // variable assignment

    vm.gotoSession = gotoSession;
    vm.refresh = refresh;
    vm.sessions = [];
    vm.title = 'Sessions';

    // logic

    timeSinceLastLogin = lastLogin - new Date();

    // return

    return vm;
}
```

**Function Declarations to Hide Implementation Details**  
Use function declarations to hide implementation details. Keep your bindable members at the bottom. When you need to bind a function in a controller, point it to a function declaration that appears above the file.

**Controller Activation Promises**  
Resolve start-up logic for a controller in an activate function.

Why?: Placing start-up logic in a consistent place in the controller makes it easier to locate, more consistent to test, and helps avoid spreading out the activation logic across the controller.

Note: If you need to conditionally cancel the route before you start use the controller, use a route resolve instead.

Avoid

```javascript
function Avengers(dataservice) {
    var vm = this;
    vm.avengers = [];
    vm.title = 'Avengers';

    dataservice.getAvengers().then(function(data) {
        vm.avengers = data;
        return vm.avengers;
    });
}
```
Recommended

```javascript
function Avengers(dataservice) {

    // variable declaration (and maybe assignment)
    var vm = this;

    // function declaration
    function activate() {
        return dataservice.getAvengers().then(function (data) {
            vm.avengers = data;
            return vm.avengers;
        });
    }

    // variable assignment
    vm.avengers = [];
    vm.title = 'Avengers';

    // return
    activate();
}
```
**Route Resolve Promises**  
When a controller depends on a promise to be resolved, resolve those dependencies in the `$routeProvider` before the controller logic is executed. If you need to conditionally cancel a route before the controller is activated, use a route resolver.

Why?: A controller may require data before it loads. That data may come from a promise via a custom factory or `$http`. Using a route resolve allows the promise to resolve before the controller logic executes, so it might take action based on that data from the promise.

Avoid

```javascript
angular
    .module('app')
    .controller('Avengers', Avengers);

function Avengers(movieService) {
    var vm = this;
    // unresolved
    vm.movies;
    // resolved asynchronously
    movieService.getMovies().then(function (response) {
        vm.movies = response.movies;
    });
}
```
Recommended

```javascript
// route-config.js
angular
    .module('app')
    .config(config);

function config($routeProvider) {
    $routeProvider
        .when('/avengers', {
            templateUrl: 'avengers.html',
            controller: 'Avengers',
            controllerAs: 'vm',
            resolve: {
                moviesPrepService: function (movieService) {
                    return movieService.getMovies();
                }
            }
        });
}
```

```javascript
// avengers.js
angular
    .module('app')
    .controller('Avengers', Avengers);

Avengers.$inject['moviesPrepService'];

function Avengers(moviesPrepService) {
    var vm = this;
    vm.movies = moviesPrepService.movies;
};
```

## Factories
Factories should have a single responsibility, that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

**Accessible Members at the bottom**  
Expose the callable members of the service (its interface) at the bottom.

```javascript
function dataService() {

    // variable declaration (and maybe assignment)

    var privateCounter = 0;

    // function declaration

    function privateFunction() {
        privateCounter++;
    }

    function publicFunction() {
        publicIncrement();
    }

    function publicIncrement() {
        privateFunction();
    }

    function publicGetCount() {
        return privateCounter;
    }

    // return
    return {
        start: publicFunction,
        increment: publicIncrement,
        count: publicGetCount
    };
}
```
* This way bindings are mirrored across the host object, primitive values cannot update alone using the revealing module pattern


**Function Declarations to Hide Implementation Details**  
Use function declarations to hide implementation details. Keep your acessible members of the factory at the bottom. Point those to function declarations that appears above the file.

## Data Services
**Separate Data Calls**  
Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

Why?: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

Why?: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

Why?: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as $http. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

Recommended

```javascript
// dataservice factory
angular
    .module('app.core')
    .factory('dataservice', dataservice);

dataservice.$inject = ['$http', 'logger'];

function dataservice($http, logger) {

    function getAvengers() {
        return $http.get('/api/maa')
            .then(getAvengersComplete)
            .catch(getAvengersFailed);

        function getAvengersComplete(response) {
            return response.data.results;
        }

        function getAvengersFailed(error) {
            logger.error('XHR Failed for getAvengers.' + error.data);
        }
    }

    return {
        getAvengers: getAvengers
    };

}
```

Note: The data service is called from consumers, such as a controller, hiding the implementation from the consumers, as shown below.

Recommended

```javascript
// controller calling the dataservice factory
angular
    .module('app.avengers')
    .controller('Avengers', Avengers);

Avengers.$inject = ['dataservice', 'logger'];

function Avengers(dataservice, logger) {
    var vm = this;

    function activate() {
        return getAvengers().then(function () {
            logger.info('Activated Avengers View');
        });
    }

    function getAvengers() {
        return dataservice.getAvengers()
            .then(function (data) {
                vm.avengers = data;
                return vm.avengers;
            });
    }

    vm.avengers = [];

    activate();
}
```

**Return a Promise from Data Calls**  
When calling a data service that returns a promise such as `$http`, return a promise in your calling function too.

Why?: You can chain the promises together and take further action after the data call completes and resolves or rejects the promise.

Recommended

```javascript
function activate() {
    /**
     * Step 1
     * Ask the getAvengers function for the
     * avenger data and wait for the promise
     */
    return getAvengers().then(function () {
        /**
         * Step 4
         * Perform an action on resolve of final promise
         */
        logger.info('Activated Avengers View');
    });
}

function getAvengers() {
    /**
     * Step 2
     * Ask the data service for the data and wait
     * for the promise
     */
    return dataservice.getAvengers()
        .then(function (data) {
            /**
             * Step 3
             * set the data and resolve the promise
             */
            vm.avengers = data;
            return vm.avengers;
        });
}

activate();
```

## Manual Dependency Injection
**UnSafe from Minification**  
Avoid using the shortcut syntax of declaring dependencies without using a minification-safe approach.

Why?: The parameters to the component (e.g. controller, factory, etc) will be converted to mangled variables. For example, common and dataservice may become a or b and not be found by AngularJS.

Avoid

```javascript
/* not minification-safe*/
angular
  .module('app')
  .controller('Dashboard', Dashboard);

function Dashboard(common, dataservice) {
}
```

This code may produce mangled variables when minified and thus cause runtime errors.

Avoid

```javascript
/* not minification-safe*/
angular.module('app').controller('Dashboard', d);function d(a, b) { }
```

**Manually Identify Dependencies**  
Use `$inject` to manually identify your dependencies for AngularJS components.

Why?: This technique mirrors the technique used by ng-annotate, which I recommend for automating the creation of minification safe dependencies. If ng-annotate detects injection has already been made, it will not duplicate it.

Why?: This safeguards your dependencies from being vulnerable to minification issues when parameters may be mangled. For example, common and dataservice may become a or b and not be found by AngularJS.

Why?: Avoid creating inline dependencies as long lists can be difficult to read in the array. Also it can be confusing that the array is a series of strings while the last item is the component's function.

Avoid

```javascript
angular
  .module('app')
  .controller('Dashboard', 
    ['$location', '$routeParams', 'common', 'dataservice', Dashboard]);

function Dashboard($location, $routeParams, common, dataservice) {
}
```
Recommended

```javascript
angular
  .module('app')
  .controller('Dashboard', Dashboard);

Dashboard.$inject = ['$location', '$routeParams', 'common', 'dataservice'];

function Dashboard($location, $routeParams, common, dataservice) {
}
```

Note: When your function is below a return statement the `$inject` may be unreachable (this may happen in a directive). You can solve this by either moving the `$inject` above the return statement or by using the alternate array injection syntax.

```javascript
// inside a directive definition
function outer() {
  return {
      controller: DashboardPanel,
  };

  DashboardPanel.$inject = ['logger']; // Unreachable
  function DashboardPanel(logger) {
  }
}
```
```javascript
// inside a directive definition
function outer() {
  DashboardPanel.$inject = ['logger']; // reachable
  return {
      controller: DashboardPanel,
  };

  function DashboardPanel(logger) {
  }
}
```

**Manually Identify Route Resolver Dependencies**  
Use `$inject` to manually identify your route resolver dependencies for AngularJS components.

Why?: This technique breaks out the anonymous function for the route resolver, making it easier to read.

Why?: An `$inject` statement can easily procede the resolver to handle making any dependencies minification safe.

Recommended

```javascript
function config ($routeProvider) {
  $routeProvider
    .when('/avengers', {
      templateUrl: 'avengers.html',
      controller: 'Avengers',
      controllerAs: 'vm',
      resolve: {
        moviesPrepService: moviePrepService
      }
    });
}

moviePrepService.$inject =  ['movieService'];
function moviePrepService(movieService) {
    return movieService.getMovies();
}
```

## Publish and subscribe events
**$scope**  
Use the `$emit` and `$broadcast` methods to trigger events to direct relationship scopes only.

```javascript
// up the $scope
$scope.$emit('customEvent', data);

// down the $scope
$scope.$broadcast('customEvent', data);
```

**$rootScope**  
Use only `$emit` as an application-wide event bus and remember to unbind listeners

```javascript
// all $rootScope.$on listeners
$rootScope.$emit('customEvent', data);
```

Because the `$rootScope` is never destroyed, `$rootScope.$on` listeners aren't either, unlike $scope.$on listeners and will always persist, so they need destroying when the relevant $scope fires the `$destroy` event.

```javascript
// call the closure
var unbind = $rootScope.$on('customEvent'[, callback]);
scope.$on('$destroy', unbind);
```

For multiple `$rootScope` listeners, decorate `$rootScope` with a new method use an Object literal then loop each one on the `$destroy` event to unbind all automatically.

```javascript
// config.js
$provide.decorator('$rootScope', eventListeners);

eventListeners.$inject = ['$delegate', '$log'];

function eventListeners('$delegate', '$log') {
    var proto = Object.getPrototypeOf($delegate);
    proto.$eventListeners = function (opts) {
        var _scope = this,
            _listeners = [];

        angular.forEach(opts, function (v, k, o) {
            $log.debug('scope', _scope.$id, 'add listener', k);
            _listeners.push(_scope.$on(k, v));
        });

        _scope.$on('$destroy', function () {
            var l;
            while ((l = _listeners.pop())) {
                l();
                $log.debug('scope', _scope.$id, 'remove listener');
            }
            l = null;
            _listeners = null;
        });
    };
    return $delegate;
}


// controller.js
function Ctrl() {
  // variable declaration
  var vm = this;

  // function declaration
  function test() {
    return num+2;
  }

  // variable assignment
  vm.num = 2;

  
    vm.$eventListeners({
        'test.executeSuccess': test
    });

    test();

}
```

## Performance
**One-time binding syntax**  
In newer versions of **Angular (v1.3.0-beta.10+)**, use the one-time binding syntax `{{ ::value }}` where it makes sense

Avoid

```html
<h1>{{ vm.title }}</h1>
```
Recommended

```html
<h1>{{ ::vm.title }}</h1>
```

Why? : Binding once removes the watcher from the scope's $$watchers array after the undefined variable becomes resolved, thus improving performance in each dirty-check

**Consider $scope.$digest**  
Use `$scope.$digest` over `$scope.$apply` where it makes sense. Only child scopes will update

```javascript
$scope.$digest();
```

Why? : `$scope.$apply` will call `$rootScope.$digest`, which causes the entire application $$watchers to dirty-check again. Using `$scope.$digest` will dirty check current and child scopes from the initiated $scope


## Exception Handling

**Decorators**  
Use a decorator, at config time using the `$provide` service, on the `$exceptionHandler` service to perform custom actions when exceptions occur.

Why?: Provides a consistent way to handle uncaught AngularJS exceptions for development-time or run-time.

Recommended

```javascript
angular.module('app')
    .config(errorHandler);

errorHandler.$inject = ['$provide'];

function errorHandler($provide) {
    $provide.decorator('$exceptionHandler', extendExceptionHandler);
}

extendExceptionHandler.$inject = ['$delegate', '$injector', '$log'];

function extendExceptionHandler($delegate, $injector, $log) {
    return function (exception, cause) {
        var errorData = {
                exception: 'herp:' + exception,
                cause: cause
            },
            $rootScope = $injector.get("$rootScope");
        /**
         * Could add the error to a service's collection,
         * add errors to $rootScope (see above code and $injector), log errors to remote web server,
         * or log locally. Or throw hard. It is entirely up to you.
         * throw exception;
         */
        $delegate(exception, cause);
    };
}
```

**Exception Catchers**
Create a factory that exposes an interface to catch and gracefully handle exceptions.

Why?: Provides a consistent way to catch exceptions that may thrown in your code (e.g. during XHR calls or promise failures).

Recommended

```javascript
// somecontroller.js
someService
    .doWork()
    .then(workComplete)               
    .catch(errors.catch("Error!"));

// errors-service.js
angular
    .module('app.common')
    .factory('errors', errors);

exception.$inject = ['logger'];

function exception(logger) {
    var service = {
        catcher: function (message) {
            return function (reason) {
                logger.error(message, reason);
            };
        }
    }
    return service;
}
```
**Route Errors**  
Handle and log all routing errors using `$routeChangeError`.

Why?: Provides a consistent way handle all routing errors.

Why?: Potentially provides a better user experience if a routing error occurs and you route them to a friendly screen with more details or recovery options.

```javascript
/**
 * Route cancellation:
 * On routing error, go to the dashboard.
 * Provide an exit clause if it tries to do it twice.
 */
$rootScope.$on('$routeChangeError',
    function (event, current, previous, rejection) {
        var destination = (current && (current.title || current.name || current.loadedTemplateUrl)) ||
            'unknown target';
        var msg = 'Error routing to ' + destination + '. ' + (rejection.msg || '');
        /**
         * Optionally log using a custom service or $log.
         * (Don't forget to inject custom service)
         */
        $log.warning(msg, [current]);
    }
);
```

## E2E Testing

An end to end test tests the behaviour of an application from start to finish and is un aware of the implementation.

##### Structuring tests

- Name the test file to describe the module and the actions under test, and store in a folder of e2e tests i.e. tests/e2e/cart-updatingQuantity.spec.js

- Use [PageObjects](https://github.com/angular/protractor/blob/master/docs/page-objects.md) to encapsulate element location and create helper functions relevant to the page. i.e. page.emailInput.sendInput('my@email.com')



```js
'use strict';

// A Page Object: pageObjects/some-page.js
module.exports = function () {
    this.emailInput = element(by.model('state.email'));
    this.submitButton = element(by.dataRef('submit-button'));

    this.get = function (host) {
        browser.get((host || 'http://localhost:3333') + '/#/1097/');
    };
};


// Usage: e2e/module-scenarioUnderTest.spec.js
var page, SomePage = require('../pageObjects/some-page.js');

beforeEach(function () {
    page = new SomePage();
    page.get();
});

```


##### Locating elements

Protractor provides several methods for locating an element in a document, detailed in depth in the [Protractor API ReadMe](https://github.com/angular/protractor/blob/master/docs/api.md#elementfinder)

By using a standard locator or a [chain of locators](https://github.com/angular/protractor/blob/master/docs/api.md#elementfinderprototypeelement) it should be possible to find most elements.

Do not use by.id() - elements in modern single page apps should not use the id attribute as it should be unique throughout the document.

In situations were an element can not be found using standard methods, additional locator methods can be defined in the onPrepare() method of the protractor config. The following example defines `by.dataRef()` which finds elements by the data-ref attribute.

```js
onPrepare: function () {
    by.addLocator('dataRef', function (target, value) {
        var selector = '[data-ref="' + value + '"]',
            elements = (target || document).querySelectorAll(selector);

        if (elements.length) {
            return elements;
        }
    });
}
```