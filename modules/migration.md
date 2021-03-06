<!--
{
"name" : "migration",
"version" : "0.1",
"title" : "Migrating from Previous Versions",
"description" : "Minor version releases in AngularJS introduce several breaking changes that may require changes to your application's source code; for instance from 1.0 to 1.2 and from 1.2 to 1.3.",
"canonicalSource" : "https://docs.angularjs.org/guide/migration",
"homepage" : "https://docs.angularjs.org/guide",
"freshnessDate" : 2015-06-02,
"license" : "CC BY 3.0"
}
-->

Minor version releases in AngularJS introduce several breaking changes that may require changes to your
application's source code; for instance from 1.0 to 1.2 and from 1.2 to 1.3.

Although we try to avoid breaking changes, there are some cases where it is unavoidable.

* AngularJS has undergone thorough security reviews to make applications safer by default,
which drives many of these changes.
* Several new features, especially animations, would not be possible without a few changes.
* Finally, some outstanding bugs were best fixed by changing an existing API.


<!-- @section -->

# Migrating from 1.3 to 1.4

Angular 1.4 fixes major animation issues and introduces a new API for `ngCookies`. Further, there
are changes to `ngMessages`, `$compile`, `ngRepeat`, `ngOptions `and some fixes to core filters:
`limitTo` and `filter`.

The reason for the ngAnimate refactor was to fix timing issues and to expose new APIs to allow
for developers to construct more versatile animations.  We now have access to `$animateCss`
and the many timing-oriented bugs were fixed which results in smoother animations.
If animation is something of interest, then please read over the breaking changes below for animations when
`ngAnimate` is used.

`ngMessages` has been upgraded to allow for dynamic message resolution. This handy feature allows for developers
to render error messages with ngMessages that are listed with a directive such as ngRepeat. A great usecase for this
involves pulling error message data from a server and then displaying that data via the mechanics of ngMessages. Be
sure to read the breaking change involved with `ngMessagesInclude` to upgrade your template code.

Other changes, such as the ordering of elements with ngRepeat and ngOptions, may also affect the behavior of your
application. And be sure to also read up on the changes to `$cookies`. The migration jump from 1.3 to 1.4 should be
relatively straightforward otherwise.





<!-- @section -->

## Animation (`ngAnimate`)

Animations in 1.4 have been refactored internally, but the API has stayed much the same. There are, however,
some breaking changes that need to be addressed when upgrading to 1.4.

Due to [c8700f04](https://github.com/angular/angular.js/commit/c8700f04fb6fb5dc21ac24de8665c0476d6db5ef),
JavaScript and CSS animations can no longer be run in
parallel. With earlier versions of ngAnimate, both CSS and JS animations
would be run together when multiple animations were detected. This
feature has been removed, however, the same effect, with even more
possibilities, can be achieved by injecting `$animateCss` into a
JavaScript-defined animation and creating custom CSS-based animations
from there.

By using `$animateCss` inside of a JavaScript animation in Angular 1.4, we can trigger custom CSS-based animations
directly from our JavaScript code.


```js
ngModule.animation('.slide-animation', ['$animateCss', function($animateCss) {
  return {
    enter: function(element, doneFn) {
      // this will trigger a `.ng-enter` and `.ng-enter-active` CSS animation
      var animation = $animateCss(element, {
        event: 'enter'
        // any other CSS-related properties
        //   addClass: 'some-class',
        //   removeClass: 'some-other-class',
        //   from: {},
        //   to: {}
      });

      // make sure to read the ngAnimate docs to understand how this works
      animation.start().done(doneFn);
    }
  }
}]);
```

Click here to learn how to use $animateCss in your animation code

Due to [c8700f04](https://github.com/angular/angular.js/commit/c8700f04fb6fb5dc21ac24de8665c0476d6db5ef),
animation-related callbacks are now fired on `$animate.on` instead of directly being on the element.


```js
// < 1.4
element.on('$animate:before', function(e, data) {
  if (data.event === 'enter') { ... }
});
element.off('$animate:before', fn);

// 1.4+
$animate.on(element, 'enter', function(data) {
  //...
});
$animate.off(element, 'enter', fn);
```

Due to [c8700f04](https://github.com/angular/angular.js/commit/c8700f04fb6fb5dc21ac24de8665c0476d6db5ef),
the function params for `$animate.enabled()` when an element is used are now flipped. This fix allows
the function to act as a getter when a single element param is provided.


```js
// < 1.4
$animate.enabled(false, element);

// 1.4+
$animate.enabled(element, false);
```

Due to [c8700f04](https://github.com/angular/angular.js/commit/c8700f04fb6fb5dc21ac24de8665c0476d6db5ef),
in addition to disabling the children of the element, `$animate.enabled(element, false)` will now also
disable animations on the element itself.

Due to [c8700f04](https://github.com/angular/angular.js/commit/c8700f04fb6fb5dc21ac24de8665c0476d6db5ef),
there is no need to call `$scope.$apply` or `$scope.$digest` inside of a animation promise callback anymore
since the promise is resolved within a digest automatically. (Not to worry, any extra digests will not be
run unless the promise is used.)


```js
// < 1.4
$animate.enter(element).then(function() {
  $scope.$apply(function() {
    $scope.explode = true;
  });
});

// 1.4+
$animate.enter(element).then(function() {
  $scope.explode = true;
});
```

Due to [c8700f04](https://github.com/angular/angular.js/commit/c8700f04fb6fb5dc21ac24de8665c0476d6db5ef),
when an enter, leave or move animation is triggered then it will always end any pending or active parent
class based animations (animations triggered via ngClass) in order to ensure that any CSS styles are resolved in time.





<!-- @section -->

## Forms (`ngMessages`, `ngOptions`)

### ngMessages
The ngMessages module has also been subject to an internal refactor to allow it to be more flexible
and compatible with dynamic message data. The `ngMessage` directive now supports a new attribute
called `ng-message-exp` which will evaluate an expression and will keep track of that expression
as it changes in order to re-evaluate the listed messages.

[Click here to learn more about dynamic ng-messages](https://docs.angularjs.org/api/ngMessages#dynamic-messaging)

There is only one breaking change. Please consider the following when including remote
message templates via `ng-messages-include`:

Due to [c9a4421f](https://github.com/angular/angular.js/commit/c9a4421fc3c97448527eadef1f42eb2f487ec2e0),
the `ngMessagesInclude` attribute has now been removed and cannot be used in the same element containing
the `ngMessages` directive. Instead, `ngMessagesInclude` is to be used on its own element inline with
other inline messages situated as children within the `ngMessages` container directive.


```html
<!-- AngularJS 1.3.x -->
<div ng-messages="model.$error" ng-messages-include="remote.html">
  <div ng-message="required">Your message is required</div>
</div>

<!-- AngularJS 1.4.x -->
<div ng-messages="model.$error">
  <div ng-message="required">Your message is required</div>
  <div ng-messages-include="remote.html"></div>
</div>
```

Depending on where the `ngMessagesInclude` directive is placed it will be prioritized inline with the other messages
before and after it.

### ngOptions

The `ngOptions` directive has also been refactored and as a result some long-standing bugs
have been fixed. The breaking changes are comparatively minor and should not affect most applications.

Due to [7fda214c](https://github.com/angular/angular.js/commit/7fda214c4f65a6a06b25cf5d5aff013a364e9cef),
when `ngOptions` renders the option values within the DOM, the resulting HTML code is different.
Normally this should not affect your application at all, however, if your code relies on inspecting
the value property of `<option>` elements (that `ngOptions` generates) then be sure to [read the details]
(https://github.com/angular/angular.js/commit/7fda214c4f65a6a06b25cf5d5aff013a364e9cef).

Due to [7fda214c](https://github.com/angular/angular.js/commit/7fda214c4f65a6a06b25cf5d5aff013a364e9cef),
when iterating over an object's properties using the `(key, value) in obj` syntax
the order of the elements used to be sorted alphabetically. This was an artificial
attempt to create a deterministic ordering since browsers don't guarantee the order.
But in practice this is not what people want and so this change iterates over properties
in the order they are returned by Object.keys(obj), which is almost always the order
in which the properties were defined.



<!-- @section -->

## Templating (`ngRepeat`, `$compile`)

### ngRepeat

Due to [c260e738](https://github.com/angular/angular.js/commit/c260e7386391877625eda086480de73e8a0ba921),
previously, the order of items when using ngRepeat to iterate over object properties was guaranteed to be consistent
by sorting the keys into alphabetic order.

Now, the order of the items is browser dependent based on the order returned
from iterating over the object using the `for key in obj` syntax.

It seems that browsers generally follow the strategy of providing
keys in the order in which they were defined, although there are exceptions
when keys are deleted and reinstated. See
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete#Cross-browser_issues

The best approach is to convert Objects into Arrays by a filter such as
https://github.com/petebacondarwin/angular-toArrayFilter
or some other mechanism, and then sort them manually in the order you need.


### $compile

Due to [6a38dbfd](https://github.com/angular/angular.js/commit/6a38dbfd3c34c8f9efff503d17eb3cbeb666d422),
previously, '&' expressions would always set up a function in the isolate scope. Now, if the binding
is marked as optional and the attribute is not specified, no function will be added to the isolate scope.




<!-- @section -->

## Cookies (`ngCookies`)

Due to [38fbe3ee](https://github.com/angular/angular.js/commit/38fbe3ee8370fc449b82d80df07b5c2ed2cd5fbe),
`$cookies` will no longer expose properties that represent the current browser cookie
values. `$cookies` no longer polls the browser for changes to the cookies and ***no longer copies
cookie values onto the `$cookies` object***.

This was changed because the polling is expensive and caused issues with the `$cookies` properties
not synchronizing correctly with the actual browser cookie values (The reason the polling
was originally added was to allow communication between different tabs,
but there are better ways to do this today, for example `localStorage`.)

The new API on `$cookies` is as follows:

 * `get`
 * `put`
 * `getObject`
 * `putObject`
 * `getAll`
 * `remove`

You must explictly use the methods above in order to access cookie data. This also means that
you can no longer watch the properties on `$cookies` to detect changes
that occur on the browsers cookies.

This feature is generally only needed if a 3rd party library was programmatically
changing the cookies at runtime. If you rely on this then you must either write code that
can react to the 3rd party library making the changes to cookies or implement your own polling
mechanism.

**DEPRECATION NOTICE**

`$cookieStore` is now deprecated as all the useful logic
has been moved to `$cookies`, to which `$cookieStore` now simply
delegates calls.





<!-- @section -->

## Server Requests (`$http`)

Due to [5da1256](https://github.com/angular/angular.js/commit/5da1256fc2812d5b28fb0af0de81256054856369),
`transformRequest` functions can no longer modify request headers.

Before this commit `transformRequest` could modify request headers, ex.:

```javascript
function requestTransform(data, headers) {
    headers = angular.extend(headers(), {
      'X-MY_HEADER': 'abcd'
    });
  }
  return angular.toJson(data);
}
```

This behavior was unintended and undocumented, so the change should affect very few applications. If one
needs to dynamically add / remove headers it should be done in a header function, for example:

```javascript
$http.get(url, {
  headers: {
    'X-MY_HEADER': function(config) {
      return 'abcd'; //you've got access to a request config object to specify header value dynamically
    }
  }
})
```





<!-- @section -->

## Filters (`filter`, `limitTo`)

### `filter` filter
Due to [cea8e751](https://github.com/angular/angular.js/commit/cea8e75144e6910b806b63a6ec2a6d118316fddd),
the `filter` filter will throw an error when used with a non-array. Beforehand it would silently
return an empty array.

If necessary, this can be worked around by converting an object to an array,
using a filter such as https://github.com/petebacondarwin/angular-toArrayFilter.

### `limitTo` filter
Due to [a3c3bf33](https://github.com/angular/angular.js/commit/a3c3bf3332e5685dc319c46faef882cb6ac246e1),
the limitTo filter has changed behavior when the provided limit value is invalid.
Now, instead of returning empty object/array, it returns unchanged input.






<!-- @section -->

# Migrating from 1.2 to 1.3


<!-- @section -->

## Controllers

Due to [3f2232b5](https://github.com/angular/angular.js/commit/3f2232b5a181512fac23775b1df4a6ebda67d018),
`$controller` will no longer look for controllers on `window`.
The old behavior of looking on `window` for controllers was originally intended
for use in examples, demos, and toy apps. We found that allowing global controller
functions encouraged poor practices, so we resolved to disable this behavior by
default.

To migrate, register your controllers with modules rather than exposing them
as globals:

Before:

```javascript
function MyController() {
  // ...
}
```

After:

```javascript
angular.module('myApp', []).controller('MyController', [function() {
  // ...
}]);
```

Although it's not recommended, you can re-enable the old behavior like this:

```javascript
angular.module('myModule').config(['$controllerProvider', function($controllerProvider) {
  // this option might be handy for migrating old apps, but please don't use it
  // in new ones!
  $controllerProvider.allowGlobals();
}]);
```


<!-- @section -->

## Angular Expression Parsing (`$parse` + `$interpolate`)

- due to [77ada4c8](https://github.com/angular/angular.js/commit/77ada4c82d6b8fc6d977c26f3cdb48c2f5fbe5a5),

You can no longer invoke .bind, .call or .apply on a function in angular expressions.
This is to disallow changing the behaviour of existing functions
in an unforeseen fashion.

  - due to [6081f207](https://github.com/angular/angular.js/commit/6081f20769e64a800ee8075c168412b21f026d99),

The (deprecated) __proto__ property does not work inside angular expressions
anymore.


- due to [48fa3aad](https://github.com/angular/angular.js/commit/48fa3aadd546036c7e69f71046f659ab1de244c6),

This prevents the use of __{define,lookup}{Getter,Setter}__ inside angular
expressions. If you really need them for some reason, please wrap/bind them to make them
less dangerous, then make them available through the scope object.


- due to [528be29d](https://github.com/angular/angular.js/commit/528be29d1662122a34e204dd607e1c0bd9c16bbc),

This prevents the use of `Object` inside angular expressions.
If you need Object.keys, make it accessible in the scope.


- due to [bdfc9c02](https://github.com/angular/angular.js/commit/bdfc9c02d021e08babfbc966a007c71b4946d69d),
  values 'f', '0', 'false', 'no', 'n', '[]' are no longer
treated as falsy. Only JavaScript falsy values are now treated as falsy by the
expression parser; there are six of them: false, null, undefined, NaN, 0 and "".


- due to [fa6e411d](https://github.com/angular/angular.js/commit/fa6e411da26824a5bae55f37ce7dbb859653276d),
  promise unwrapping has been removed. It has been deprecated since 1.2.0-rc.3.
  It can no longer be turned on.
  Two methods have been removed:
  * `$parseProvider.unwrapPromises`
  * `$parseProvider.logPromiseWarnings`


- **$interpolate:** due to [88c2193c](https://github.com/angular/angular.js/commit/88c2193c71954b9e7e7e4bdf636a2b168d36300d),
  the function returned by `$interpolate`
  no longer has a `.parts` array set on it.

  Instead it has two arrays:
  * `.expressions`, an array of the expressions in the
    interpolated text. The expressions are parsed with
    `$parse`, with an extra layer converting them to strings
    when computed
  * `.separators`, an array of strings representing the
    separations between interpolations in the text.
    This array is **always** 1 item longer than the
    `.expressions` array for easy merging with it





<!-- @section -->

## Miscellaneous Angular helpers

- **Angular.copy:** due to [b59b04f9](https://github.com/angular/angular.js/commit/b59b04f98a0b59eead53f6a53391ce1bbcbe9b57),

This changes `angular.copy` so that it applies the prototype of the original
object to the copied object.  Previously, `angular.copy` would copy properties
of the original object's prototype chain directly onto the copied object.

This means that if you iterate over only the copied object's `hasOwnProperty`
properties, it will no longer contain the properties from the prototype.
This is actually much more reasonable behaviour and it is unlikely that
applications are actually relying on this.

If this behaviour is relied upon, in an app, then one should simply iterate
over all the properties on the object (and its inherited properties) and
not filter them with `hasOwnProperty`.

**Be aware that this change also uses a feature that is not compatible with
IE8.**  If you need this to work on IE8 then you would need to provide a polyfill
for `Object.create` and `Object.getPrototypeOf`.


- **forEach:** due to [55991e33](https://github.com/angular/angular.js/commit/55991e33af6fece07ea347a059da061b76fc95f5),
  forEach will iterate only over the initial number of items in
the array. So if items are added to the array during the iteration, these won't
be iterated over during the initial forEach call.

This change also makes our forEach behave more like Array#forEach.


- **angular.toJson:** due to [c054288c](https://github.com/angular/angular.js/commit/c054288c9722875e3595e6e6162193e0fb67a251),

If you expected `toJson` to strip these types of properties before, you will have to
manually do this yourself now.






<!-- @section -->

## jqLite / JQuery

- **jqLite:** due to [a196c8bc](https://github.com/angular/angular.js/commit/a196c8bca82a28c08896d31f1863cf4ecd11401c),
  previously it was possible to set jqLite data on Text/Comment
nodes, but now that is allowed only on Element and Document nodes just like in
jQuery. We don't expect that app code actually depends on this accidental feature.


- **jqLite:** due to [d71dbb1a](https://github.com/angular/angular.js/commit/d71dbb1ae50f174680533492ce4c7db3ff74df00),
  the jQuery `detach()` method does not trigger the `$destroy` event.
  If you want to destroy Angular data attached to the element, use `remove()`.






<!-- @section -->

## Angular HTML Compiler (`$compile`)


- due to [2ee29c5d](https://github.com/angular/angular.js/commit/2ee29c5da81ffacdc1cabb438f5d125d5e116cb9),

The isolated scope of a component directive no longer leaks into the template
that contains the instance of the directive.  This means that you can no longer
access the isolated scope from attributes on the element where the isolated
directive is defined.

See https://github.com/angular/angular.js/issues/10236 for an example.

- due to [2cde927e](https://github.com/angular/angular.js/commit/2cde927e58c8d1588569d94a797e43cdfbcedaf9),


Requesting isolate scope and any other scope on a single element is an error.
Before this change, the compiler let two directives request a child scope
and an isolate scope if the compiler applied them in the order of non-isolate
scope directive followed by isolate scope directive.

Now the compiler will error regardless of the order.

If you find that your code is now throwing a `$compile:multidir` error,
check that you do not have directives on the same element that are trying
to request both an isolate and a non-isolate scope and fix your code.


- due to [eec6394a](https://github.com/angular/angular.js/commit/eec6394a342fb92fba5270eee11c83f1d895e9fb), The `replace` flag for defining directives that
  replace the element that they are on will be removed in the next major angular version.
  This feature has difficult semantics (e.g. how attributes are merged) and leads to more
  problems compared to what it solves. Also, with Web Components it is normal to have
  custom elements in the DOM.


- due to [299b220f](https://github.com/angular/angular.js/commit/299b220f5e05e1d4e26bfd58d0b2fd7329ca76b1),
  calling `attr.$observe` no longer returns the observer function, but a
    deregistration function instead. To migrate the code follow the example below:

Before:

    directive('directiveName', function() {
      return {
        link: function(scope, elm, attr) {
          var observer = attr.$observe('someAttr', function(value) {
            console.log(value);
          });
        }
      };
    });

After:

    directive('directiveName', function() {
      return {
        link: function(scope, elm, attr) {
          var observer = function(value) {
            console.log(value);
          };

          attr.$observe('someAttr', observer);
        }
      };
    });





<!-- @section -->

## Forms, Inputs and ngModel

- due to [1be9bb9d](https://github.com/angular/angular.js/commit/1be9bb9d3527e0758350c4f7417a4228d8571440),


If an expression is used on ng-pattern (such as `ng-pattern="exp"`) or on the
pattern attribute (something like on `pattern="{{ exp }}"`) and the expression
itself evaluates to a string then the validator will not parse the string as a
literal regular expression object (a value like `/abc/i`).  Instead, the entire
string will be created as the regular expression to test against. This means
that any expression flags will not be placed on the RegExp object. To get around
this limitation, use a regular expression object as the value for the expression.

    //before
    $scope.exp = '/abc/i';

    //after
    $scope.exp = /abc/i;


- **ngModelOptions:** due to [adfc322b](https://github.com/angular/angular.js/commit/adfc322b04a58158fb9697e5b99aab9ca63c80bb),


This commit changes the API on `NgModelController`, both semantically and
in terms of adding and renaming methods.

* `$setViewValue(value)` -
This method still changes the `$viewValue` but does not immediately commit this
change through to the `$modelValue` as it did previously.
Now the value is committed only when a trigger specified in an associated
`ngModelOptions` directive occurs. If `ngModelOptions` also has a `debounce` delay
specified for the trigger then the change will also be debounced before being
committed.
In most cases this should not have a significant impact on how `NgModelController`
is used: If `updateOn` includes `default` then `$setViewValue` will trigger
a (potentially debounced) commit immediately.
* `$cancelUpdate()` - is renamed to `$rollbackViewValue()` and has the same meaning,
which is to revert the current `$viewValue` back to the `$lastCommittedViewValue`,
to cancel any pending debounced updates and to re-render the input.

To migrate code that used `$cancelUpdate()` follow the example below:

Before:


```js
$scope.resetWithCancel = function (e) {
  if (e.keyCode == 27) {
    $scope.myForm.myInput1.$cancelUpdate();
    $scope.myValue = '';
  }
};
```

After:


```js
$scope.resetWithCancel = function (e) {
  if (e.keyCode == 27) {
    $scope.myForm.myInput1.$rollbackViewValue();
    $scope.myValue = '';
  }
}
```

- types date, time, datetime-local, month, week now always
  require a `Date` object as model ([46bd6dc8](https://github.com/angular/angular.js/commit/46bd6dc88de252886d75426efc2ce8107a5134e9),
   [#5864](https://github.com/angular/angular.js/issues/5864))







<!-- @section -->

## Scopes and Digests (`$scope`)

- due to [8c6a8171](https://github.com/angular/angular.js/commit/8c6a8171f9bdaa5cdabc0cc3f7d3ce10af7b434d),
  Scope#$id is now of type number rather than string. Since the
id is primarily being used for debugging purposes this change should not affect
anyone.


- due to [82f45aee](https://github.com/angular/angular.js/commit/82f45aee5bd84d1cc53fb2e8f645d2263cdaacbc),
  [#7445](https://github.com/angular/angular.js/issues/7445),
  [#7523](https://github.com/angular/angular.js/issues/7523)
  `$broadcast` and `$emit` will now reset the `currentScope` property of the event to
  null once the event finished propagating. If any code depends on asynchronously accessing their
  `currentScope` property, it should be migrated to use `targetScope` instead. All of these cases
  should be considered programming bugs.






<!-- @section -->

## Server Requests (`$http`, `$resource`)
- **$http:** due to [ad4336f9](https://github.com/angular/angular.js/commit/ad4336f9359a073e272930f8f9bcd36587a8648f),


Previously, it was possible to register a response interceptor like so:


```js
// register the interceptor as a service
$provide.factory('myHttpInterceptor', function($q, dependency1, dependency2) {
  return function(promise) {
    return promise.then(function(response) {
      // do something on success
      return response;
    }, function(response) {
      // do something on error
      if (canRecover(response)) {
        return responseOrNewPromise
      }
      return $q.reject(response);
    });
  }
});

$httpProvider.responseInterceptors.push('myHttpInterceptor');
```

Now, one must use the newer API introduced in v1.1.4 (4ae46814), like so:


```js
$provide.factory('myHttpInterceptor', function($q) {
  return {
    response: function(response) {
      // do something on success
      return response;
    },
    responseError: function(response) {
      // do something on error
      if (canRecover(response)) {
        return responseOrNewPromise
      }
      return $q.reject(response);
    }
  };
});

$httpProvider.interceptors.push('myHttpInterceptor');
```

More details on the new interceptors API (which has been around as of v1.1.4) can be found at
interceptors



- **$httpBackend:** due to [6680b7b9](https://github.com/angular/angular.js/commit/6680b7b97c0326a80bdccaf0a35031e4af641e0e), the JSONP behavior for erroneous and empty responses changed:
    Previously, a JSONP response was regarded as erroneous if it was empty. Now Angular is listening to the
    correct events to detect errors, i.e. even empty responses can be successful.


- **$resource:** due to [d3c50c84](https://github.com/angular/angular.js/commit/d3c50c845671f0f8bcc3f7842df9e2fb1d1b1c40),

  If you expected `$resource` to strip these types of properties before,
  you will have to manually do this yourself now.






<!-- @section -->

## Modules and Injector (`$inject`)

- due to [c0b4e2db](https://github.com/angular/angular.js/commit/c0b4e2db9cbc8bc3164cedc4646145d3ab72536e),

Previously, config blocks would be able to control behaviour of provider registration, due to being
invoked prior to provider registration. Now, provider registration always occurs prior to configuration
for a given module, and therefore config blocks are not able to have any control over a providers
registration.

**Example**:

Previously, the following:


```js
angular.module('foo', [])
  .provider('$rootProvider', function() {
    this.$get = function() { ... }
  })
  .config(function($rootProvider) {
    $rootProvider.dependentMode = "B";
  })
  .provider('$dependentProvider', function($rootProvider) {
     if ($rootProvider.dependentMode === "A") {
       this.$get = function() {
        // Special mode!
       }
     } else {
       this.$get = function() {
         // something else
       }
    }
  });
```

would have "worked", meaning behaviour of the config block between the registration of "$rootProvider"
and "$dependentProvider" would have actually accomplished something and changed the behaviour of the
app. This is no longer possible within a single module.






<!-- @section -->

## Animation (`ngAnimate`)


- due to [1cb8584e](https://github.com/angular/angular.js/commit/1cb8584e8490ecdb1b410a8846c4478c6c2c0e53),
`$animate` will no longer default the after parameter to the last element of the parent
container. Instead, when after is not specified, the new element will be inserted as the
first child of the parent container.

To update existing code, change all instances of `$animate.enter()` or `$animate.move()` from:

`$animate.enter(element, parent);`

to:

`$animate.enter(element, parent, angular.element(parent[0].lastChild));`



- due to [1bebe36a](https://github.com/angular/angular.js/commit/1bebe36aa938890d61188762ed618b1b5e193634),

  Any class-based animation code that makes use of transitions
and uses the setup CSS classes (such as class-add and class-remove) must now
provide a empty transition value to ensure that its styling is applied right
away. In other words if your animation code is expecting any styling to be
applied that is defined in the setup class then it will not be applied
"instantly" unless a `transition:0s none` value is present in the styling
for that CSS class. This situation is only the case if a transition is already
present on the base CSS class once the animation kicks off.

Before:

    .animated.my-class-add {
      opacity:0;
      transition:0.5s linear all;
    }
    .animated.my-class-add.my-class-add-active {
      opacity:1;
    }

After:

    .animated.my-class-add {
      transition:0s linear all;
      opacity:0;
    }
    .animated.my-class-add.my-class-add-active {
      transition:0.5s linear all;
      opacity:1;
    }

Please view the documentation for ngAnimate for more info.



<!-- @section -->

## Testing

- due to [85880a64](https://github.com/angular/angular.js/commit/85880a64900fa22a61feb926bf52de0965332ca5), some deprecated features of
Protractor tests no longer work.

`by.binding(descriptor)` no longer allows using the surrounding interpolation
markers in the descriptor (the default interpolation markers are `{{}}`).
Previously, these were optional.

Before:

    var el = element(by.binding('{{foo}}'));

After:

    var el = element(by.binding('foo'));

Prefixes `ng_` and `x-ng-` are no longer allowed for models. Use `ng-model`.

`by.repeater` cannot find elements by row and column which are not children of
the row. For example, if your template is

    <div ng-repeat="foo in foos">{{foo.name}}</div>

Before:

    var el = element(by.repeater('foo in foos').row(2).column('foo.name'))

After:

You may either enclose `{{foo.name}}` in a child element

    <div ng-repeat="foo in foos"><span>{{foo.name}}</span></div>

or simply use:

    var el = element(by.repeater('foo in foos').row(2))



<!-- @section -->

## Internet Explorer 8

- due to [eaa1d00b](https://github.com/angular/angular.js/commit/eaa1d00b24008f590b95ad099241b4003688cdda),
  As communicated before, IE8 is no longer supported.







<!-- @section -->

# Migrating from 1.0 to 1.2


> <p>**Note:** AngularJS versions 1.1.x are considered "experimental" with breaking changes between minor releases.
>Version 1.2 is the result of several versions on the 1.1 branch, and has a stable API.</p>
>
><p>If you have an application on 1.1 and want to migrate it to 1.2, everything in the guide
>below should still apply, but you may want to consult the
>[changelog](https://github.com/angular/angular.js/blob/master/CHANGELOG.md) as well.</p>

<ul class="nav nav-list">
  <li class="nav-header">Summary of Breaking Changes</li>
  <li>ngRoute has been moved into its own module</li>
  <li>Templates no longer automatically unwrap promises</li>
  <li>Syntax for named wildcard parameters changed in <code>$route</code></li>
  <li>You can only bind one expression to <code>*[src]</code>, <code>*[ng-src]</code> or <code>action</code></li>
  <li>Interpolations inside DOM event handlers are now disallowed</li>
  <li>Directives cannot end with -start or -end</li>
  <li>In $q, promise.always has been renamed promise.finally</li>
  <li>ngMobile is now ngTouch</li>
  <li>resource.$then has been removed</li>
  <li>Resource methods return the promise</li>
  <li>Resource promises are resolved with the resource instance</li>
  <li>$location.search supports multiple keys</li>
  <li>ngBindHtmlUnsafe has been removed and replaced by ngBindHtml</li>
  <li>Form names that are expressions are evaluated</li>
  <li>hasOwnProperty disallowed as an input name</li>
  <li>Directives: Order of postLink functions reversed</li>
  <li>Directive priority</li>
  <li>ngScenario</li>
  <li>ngInclude and ngView replace its entire element on update</li>
  <li>URLs are now sanitized against a whitelist</li>
  <li>Isolate scope only exposed to directives with <code>scope</code> property</li>
  <li>Change to interpolation priority</li>
  <li>Underscore-prefixed/suffixed properties are non-bindable</li>
  <li>You cannot bind to select[multiple]</li>
  <li>Uncommon region-specific local files were removed from i18n</li>
  <li>Services can now return functions</li>
</ul>



<!-- @section -->

## ngRoute has been moved into its own module

Just like `ngResource`, `ngRoute` is now its own module.

Applications that use `$route`, `ngView`, and/or `$routeParams` will now need to load an
`angular-route.js` file and have their application's module dependency on the `ngRoute` module.

Before:


```html
<script src="angular.js"></script>
```

```javascript
var myApp = angular.module('myApp', ['someOtherModule']);
```

After:


```html
<script src="angular.js"></script>
<script src="angular-route.js"></script>
```

```javascript
var myApp = angular.module('myApp', ['ngRoute', 'someOtherModule']);
```

See [5599b55b](https://github.com/angular/angular.js/commit/5599b55b04788c2e327d7551a4a699d75516dd21).



<!-- @section -->

## Templates no longer automatically unwrap promises

`$parse` and templates in general will no longer automatically unwrap promises.

Before:

```javascript
$scope.foo = $http({method: 'GET', url: '/someUrl'});
```


```html
<p>{{foo}}</p>
```

After:

```javascript
$http({method: 'GET', url: '/someUrl'})
  .success(function(data) {
    $scope.foo = data;
  });
```


```html
<p>{{foo}}</p>
```

This feature has been deprecated. If absolutely needed, it can be reenabled for now via the
`$parseProvider.unwrapPromises(true)` API.

See [5dc35b52](https://github.com/angular/angular.js/commit/5dc35b527b3c99f6544b8cb52e93c6510d3ac577),
[b6a37d11](https://github.com/angular/angular.js/commit/b6a37d112b3e1478f4d14a5f82faabf700443748).



<!-- @section -->

## Syntax for named wildcard parameters changed in `$route`

To migrate the code, follow the example below. Here, `*highlight` becomes `:highlight*`

Before:

```javascript
$routeProvider.when('/Book1/:book/Chapter/:chapter/*highlight/edit',
          {controller: noop, templateUrl: 'Chapter.html'});
```

After:

```javascript
$routeProvider.when('/Book1/:book/Chapter/:chapter/:highlight*/edit',
        {controller: noop, templateUrl: 'Chapter.html'});
```

See [04cebcc1](https://github.com/angular/angular.js/commit/04cebcc133c8b433a3ac5f72ed19f3631778142b).



<!-- @section -->

## You can only bind one expression to `*[src]`, `*[ng-src]` or `action`

With the exception of `<a>` and `<img>` elements, you cannot bind more than one expression to the
`src` or `action` attribute of elements.

This is one of several improvements to security introduces by Angular 1.2.

Concatenating expressions makes it hard to understand whether some combination of concatenated
values are unsafe to use and potentially subject to XSS vulnerabilities. To simplify the task of
auditing for XSS issues, we now require that a single expression be used for `*[src/ng-src]`
bindings such as bindings for `iframe[src]`, `object[src]`, etc. In addition, this requirement is
enforced for `form` tags with `action` attributes.

<table class="table table-bordered code-table">
<thead>
<tr>
  <th>Examples</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>&lt;img src="{{a}}/{{b}}"&gt;</code></td>
  <td class="success">ok</td>
</tr>
<tr>
  <td><code>&lt;iframe src="{{a}}/{{b}}"&gt;&lt;/iframe&gt;</code></td>
  <td class="error">bad</td>
</tr>
<tr>
  <td><code>&lt;iframe src="{{a}}"&gt;&lt;/iframe&gt;</code></td>
  <td class="success">ok</td>
</tr>
</tbody>
</table>


To migrate your code, you can combine multiple expressions using a method attached to your scope.

Before:

```javascript
scope.baseUrl = 'page';
scope.a = 1;
scope.b = 2;
```


```html
<!-- Are a and b properly escaped here? Is baseUrl controlled by user? -->
<iframe src="{{baseUrl}}?a={{a}&b={{b}}">
```

After:

```javascript
var baseUrl = "page";
scope.getIframeSrc = function() {

  // One should think about their particular case and sanitize accordingly
  var qs = ["a", "b"].map(function(value, name) {
      return encodeURIComponent(name) + "=" +
             encodeURIComponent(value);
    }).join("&");

  // `baseUrl` isn't exposed to a user's control, so we don't have to worry about escaping it.
  return baseUrl + "?" + qs;
};
```


```html
<iframe src="{{getIframeSrc()}}">
```

See [38deedd6](https://github.com/angular/angular.js/commit/38deedd6e3d806eb8262bb43f26d47245f6c2739).



<!-- @section -->

## Interpolations inside DOM event handlers are now disallowed

DOM event handlers execute arbitrary Javascript code. Using an interpolation for such handlers
means that the interpolated value is a JS string that is evaluated. Storing or generating such
strings is error prone and leads to XSS vulnerabilities. On the other hand, `ngClick` and other
Angular specific event handlers evaluate Angular expressions in non-window (Scope) context which
makes them much safer.

To migrate the code follow the example below:

Before:

```
JS:   scope.foo = 'alert(1)';
HTML: <div onclick="{{foo}}">
```

After:

```
JS:   scope.foo = function() { alert(1); }
HTML: <div ng-click="foo()">
```

See [39841f2e](https://github.com/angular/angular.js/commit/39841f2ec9b17b3b2920fd1eb548d444251f4f56).



<!-- @section -->

## Directives cannot end with -start or -end

This change was necessary to enable multi-element directives. The best fix is to rename existing
directives so that they don't end with these suffixes.

See [e46100f7](https://github.com/angular/angular.js/commit/e46100f7097d9a8f174bdb9e15d4c6098395c3f2).



<!-- @section -->

## In $q, promise.always has been renamed promise.finally

The reason for this change is to align `$q` with the [Q promise
library](https://github.com/kriskowal/q), despite the fact that this makes it a bit more difficult
to use with non-ES5 browsers, like IE8.

`finally` also goes well together with the `catch` API that was added to `$q` recently and is part
of the [DOM promises standard](http://dom.spec.whatwg.org/).

To migrate the code follow the example below.

Before:

```javascript
$http.get('/foo').always(doSomething);
```

After:

```javascript
$http.get('/foo').finally(doSomething);
```

Or for IE8-compatible code:

```javascript
$http.get('/foo')['finally'](doSomething);
```

See [f078762d](https://github.com/angular/angular.js/commit/f078762d48d0d5d9796dcdf2cb0241198677582c).



<!-- @section -->

## ngMobile is now ngTouch

Many touch-enabled devices are not mobile devices, so we decided to rename this module to better
reflect its concerns.

To migrate, replace all references to `ngMobile` with `ngTouch` and `angular-mobile.js` with
`angular-touch.js`.

See [94ec84e7](https://github.com/angular/angular.js/commit/94ec84e7b9c89358dc00e4039009af9e287bbd05).



<!-- @section -->

## resource.$then has been removed

Resource instances do not have a `$then` function anymore. Use the `$promise.then` instead.

Before:

```javascript
Resource.query().$then(callback);
```

After:

```javascript
Resource.query().$promise.then(callback);
```

See [05772e15](https://github.com/angular/angular.js/commit/05772e15fbecfdc63d4977e2e8839d8b95d6a92d).



<!-- @section -->

## Resource methods return the promise

Methods of a resource instance return the promise rather than the instance itself.

Before:

```javascript
resource.$save().chaining = true;
```

After:

```javascript
resource.$save();
resource.chaining = true;
```

See [05772e15](https://github.com/angular/angular.js/commit/05772e15fbecfdc63d4977e2e8839d8b95d6a92d).



<!-- @section -->

## Resource promises are resolved with the resource instance

On success, the resource promise is resolved with the resource instance rather than HTTP response object.

Use interceptor API to access the HTTP response object.

Before:

```javascript
Resource.query().$then(function(response) {...});
```

After:

```javascript
var Resource = $resource('/url', {}, {
  get: {
    method: 'get',
    interceptor: {
      response: function(response) {
        // expose response
        return response;
      }
    }
  }
});
```

See [05772e15](https://github.com/angular/angular.js/commit/05772e15fbecfdc63d4977e2e8839d8b95d6a92d).



<!-- @section -->

## $location.search supports multiple keys

`$location.search` now supports multiple keys with the
same value provided that the values are stored in an array.

Before this change:

* `parseKeyValue` only took the last key overwriting all the previous keys.
* `toKeyValue` joined the keys together in a comma delimited string.

This was deemed buggy behavior. If your server relied on this behavior then either the server
should be fixed, or a simple serialization of the array should be done on the client before
passing it to `$location`.

See [80739409](https://github.com/angular/angular.js/commit/807394095b991357225a03d5fed81fea5c9a1abe).



<!-- @section -->

## ngBindHtmlUnsafe has been removed and replaced by ngBindHtml

`ngBindHtml` provides `ngBindHtmlUnsafe` like
behavior (evaluate an expression and innerHTML the result into the DOM) when bound to the result
of `$sce.trustAsHtml(string)`. When bound to a plain string, the string is sanitized via
`$sanitize` before being innerHTML'd. If the `$sanitize` service isn't available (`ngSanitize`
module is not loaded) and the bound expression evaluates to a value that is not trusted an
exception is thrown.

When using this directive you can either include `ngSanitize` in your module's dependencies (See the
example at the ngBindHtml reference) or use the $sce service to set the value as
trusted.

See [dae69473](https://github.com/angular/angular.js/commit/dae694739b9581bea5dbc53522ec00d87b26ae55).



<!-- @section -->

## Form names that are expressions are evaluated

If you have form names that will evaluate as an expression:

```
<form name="ctrl.form">
```

And if you are accessing the form from your controller:

Before:

```javascript
function($scope) {
  $scope['ctrl.form'] // form controller instance
}
```

After:

```javascript
function($scope) {
  $scope.ctrl.form // form controller instance
}
```

This makes it possible to access a form from a controller using the new "controller as" syntax.
Supporting the previous behavior offers no benefit.

See [8ea802a1](https://github.com/angular/angular.js/commit/8ea802a1d23ad8ecacab892a3a451a308d9c39d7).



<!-- @section -->

## hasOwnProperty disallowed as an input name

Inputs with name equal to `hasOwnProperty` are not allowed inside form or ngForm directives.

Before, inputs whose name was "hasOwnProperty" were quietly ignored and not added to the scope.
Now a badname exception is thrown. Using "hasOwnProperty" for an input name would be very unusual
and bad practice. To migrate, change your input name.

See [7a586e5c](https://github.com/angular/angular.js/commit/7a586e5c19f3d1ecc3fefef084ce992072ee7f60).



<!-- @section -->

## Directives: Order of postLink functions reversed

The order of postLink fn is now mirror opposite of the order in which corresponding preLinking and compile functions execute.

Previously the compile/link fns executed in order, sorted by priority:

<table class="table table-bordered table-striped code-table">
<thead>
<tr>
  <th>#</th>
  <th>Step</th>
  <th align="center">Old Sort Order</th>
  <th align="center">New Sort Order</th>
</tr>
</thead>
<tbody>
<tr>
  <td>1</td>
  <td>Compile Fns</td>
  <td align="center" colspan="2">High → Low</td>
</tr>
<tr>
  <td>2</td>
  <td colspan="3">Compile child nodes</td>
</tr>
<tr>
  <td>3</td>
  <td>PreLink Fns</td>
  <td align="center" colspan="2">High → Low</td>
</tr>
<tr>
  <td>4</td>
  <td colspan="3">Link child nodes</td>
</tr>
<tr>
  <td>5</td>
  <td>PostLink Fns</td>
  <td align="center">High → Low</td>
  <td align="center">**Low → High**</td>
</tr>
</tbody>
</table>

<small>"High → Low" here refers to the `priority` option of a directive.</small>

Very few directives in practice rely on the order of postLinking functions (unlike on the order
of compile functions), so in the rare case of this change affecting an existing directive, it might
be necessary to convert it to a preLinking function or give it negative priority.

You can look at [the diff of this
commit](https://github.com/angular/angular.js/commit/31f190d4d53921d32253ba80d9ebe57d6c1de82b) to see how an internal
attribute interpolation directive was adjusted.

See [31f190d4](https://github.com/angular/angular.js/commit/31f190d4d53921d32253ba80d9ebe57d6c1de82b).



<!-- @section -->

## Directive priority

the priority of ngRepeat, ngSwitchWhen, ngIf, ngInclude and ngView has changed. This could affect directives that explicitly specify their priority.

In order to make ngRepeat, ngSwitchWhen, ngIf, ngInclude and ngView work together in all common scenarios their directives are being adjusted to achieve the following precedence:


Directive        | Old Priority | New Priority
-----------------|--------------|-------------
ngRepeat         | 1000         | 1000
ngSwitchWhen     | 500          | 800
ngIf             | 1000         | 600
ngInclude        | 1000         | 400
ngView           | 1000         | 400

See [b7af76b4](https://github.com/angular/angular.js/commit/b7af76b4c5aa77648cc1bfd49935b48583419023).



<!-- @section -->

## ngScenario

browserTrigger now uses an eventData object instead of direct parameters for mouse events.
To migrate, place the `keys`,`x` and `y` parameters inside of an object and place that as the
third parameter for the browserTrigger function.

See [28f56a38](https://github.com/angular/angular.js/commit/28f56a383e9d1ff378e3568a3039e941c7ffb1d8).



<!-- @section -->

## ngInclude and ngView replace its entire element on update

Previously `ngInclude` and `ngView` only updated its element's content. Now these directives will
recreate the element every time a new content is included.

This ensures that a single rootElement for all the included contents always exists, which makes
definition of css styles for animations much easier.

See [7d69d52a](https://github.com/angular/angular.js/commit/7d69d52acff8578e0f7d6fe57a6c45561a05b182),
[aa2133ad](https://github.com/angular/angular.js/commit/aa2133ad818d2e5c27cbd3933061797096356c8a).



<!-- @section -->

## URLs are now sanitized against a whitelist

A whitelist configured via `$compileProvider` can be used to configure what URLs are considered safe.
By default all common protocol prefixes are whitelisted including `data:` URIs with mime types `image/*`.
This change shouldn't impact apps that don't contain malicious image links.

See [1adf29af](https://github.com/angular/angular.js/commit/1adf29af13890d61286840177607edd552a9df97),
[3e39ac7e](https://github.com/angular/angular.js/commit/3e39ac7e1b10d4812a44dad2f959a93361cd823b).



<!-- @section -->

## Isolate scope only exposed to directives with `scope` property

If you declare a scope option on a directive, that directive will have an
[isolate scope](https://github.com/angular/angular.js/wiki/Understanding-Scopes). In Angular 1.0, if a
directive with an isolate scope is used on an element, all directives on that same element have access
to the same isolate scope. For example, say we have the following directives:

```
// This directive declares an isolate scope.
.directive('isolateScope', function() {
  return {
    scope: {},
    link: function($scope) {
      console.log('one = ' + $scope.$id);
    }
  };
})

// This directive does not.
.directive('nonIsolateScope', function() {
  return {
    link: function($scope) {
      console.log('two = ' + $scope.$id);
    }
  };
});
```

Now what happens if we use both directives on the same element?

```
<div isolate-scope non-isolate-scope></div>
```

In Angular 1.0, the nonIsolateScope directive will have access to the isolateScope directive’s scope. The
log statements will print the same id, because the scope is the same. But in Angular 1.2, the nonIsolateScope
will not use the same scope as isolateScope. Instead, it will inherit the parent scope. The log statements
will print different id’s.

If your code depends on the Angular 1.0 behavior (non-isolate directive needs to access state
from within the isolate scope), change the isolate directive to use scope locals to pass these explicitly:

**Before**

```
<input ng-model="$parent.value" ng-isolate>

.directive('ngIsolate', function() {
  return {
    scope: {},
    template: '{{value}}'
  };
});
```

**After**

```
<input ng-model="value" ng-isolate>

.directive('ngIsolate', function() {
  return {
    scope: {value: '=ngModel'},
    template: '{{value}}
  };
});
```

See [909cabd3](https://github.com/angular/angular.js/commit/909cabd36d779598763cc358979ecd85bb40d4d7),
[#1924](https://github.com/angular/angular.js/issues/1924) and
[#2500](https://github.com/angular/angular.js/issues/2500).



<!-- @section -->

## Change to interpolation priority

Previously, the interpolation priority was `-100` in 1.2.0-rc.2, and `100` before 1.2.0-rc.2.
Before this change the binding was setup in the post-linking phase.

Now the attribute interpolation (binding) executes as a directive with priority 100 and the
binding is set up in the pre-linking phase.

See [79223eae](https://github.com/angular/angular.js/commit/79223eae5022838893342c42dacad5eca83fabe8),
[#4525](https://github.com/angular/angular.js/issues/4525),
[#4528](https://github.com/angular/angular.js/issues/4528), and
[#4649](https://github.com/angular/angular.js/issues/4649)


<!-- @section -->

## Underscore-prefixed/suffixed properties are non-bindable

> <p>**Reverted**: This breaking change has been reverted in 1.2.1, and so can be ignored if you're using **version 1.2.1 or higher**</p>

This change introduces the notion of "private" properties (properties
whose names begin and/or end with an underscore) on the scope chain.
These properties will not be available to Angular expressions (i.e. {{
}} interpolation in templates and strings passed to `$parse`)  They are
freely available to JavaScript code (as before).

**Motivation**

Angular expressions execute in a limited context. They do not have
direct access to the global scope, `window`, `document` or the Function
constructor. However, they have direct access to names/properties on
the scope chain. It has been a long standing best practice to keep
sensitive APIs outside of the scope chain (in a closure or your
controller.) That's easier said than done for two reasons:

1. JavaScript does not have a notion of private properties so if you need
someone on the scope chain for JavaScript use, you also expose it to
Angular expressions
2. The new `controller as` syntax that's now in increased usage exposes the
entire controller on the scope chain greatly increasing the exposed surface.

Though Angular expressions are written and controlled by the developer, they:

1. Typically deal with user input
2. Don't get the kind of test coverage that JavaScript code would

This commit provides a way, via a naming convention, to allow restricting properties from
controllers/scopes. This means Angular expressions can access only those properties that
are actually needed by the expressions.

See [3d6a89e8](https://github.com/angular/angular.js/commit/3d6a89e8888b14ae5cb5640464e12b7811853c7e).



<!-- @section -->

## You cannot bind to select[multiple]

Switching between `select[single]` and `select[multiple]` has always been odd due to browser quirks.
This feature never worked with two-way data-binding so it's not expected that anyone is using it.

If you are interested in properly adding this feature, please submit a pull request on Github.

See [d87fa004](https://github.com/angular/angular.js/commit/d87fa0042375b025b98c40bff05e5f42c00af114).



<!-- @section -->

## Uncommon region-specific local files were removed from i18n

AngularJS uses the Google Closure library's locale files. The following locales were removed from
Closure, so Angular is not able to continue to support them:

`chr`, `cy`, `el-polyton`, `en-zz`, `fr-rw`, `fr-sn`, `fr-td`, `fr-tg`, `haw`, `it-ch`, `ln-cg`,
`mo`, `ms-bn`, `nl-aw`, `nl-be`, `pt-ao`, `pt-gw`, `pt-mz`, `pt-st`, `ro-md`, `ru-md`, `ru-ua`,
`sr-cyrl-ba`, `sr-cyrl-me`, `sr-cyrl`, `sr-latn-ba`, `sr-latn-me`, `sr-latn`, `sr-rs`, `sv-fi`,
`sw-ke`, `ta-lk`, `tl-ph`, `ur-in`, `zh-hans-hk`, `zh-hans-mo`, `zh-hans-sg`, `zh-hans`,
`zh-hant-hk`, `zh-hant-mo`, `zh-hant-tw`, `zh-hant`

Although these locales were removed from the official AngularJS repository, you can continue to
load and use your copy of the locale file provided that you maintain it yourself.

See [6382e21f](https://github.com/angular/angular.js/commit/6382e21fb28541a2484ac1a241d41cf9fbbe9d2c).


<!-- @section -->

## Services can now return functions

Previously, the service constructor only returned objects regardless of whether a function was returned.

Now, `$injector.instantiate` (and thus `$provide.service`) behaves the same as the native
`new` operator and allows functions to be returned as a service.

If using a JavaScript preprocessor it's quite possible when upgrading that services could start behaving incorrectly.
Make sure your services return the correct type wanted.

**Coffeescript example**

```
myApp.service 'applicationSrvc', ->
  @something = "value"
  @someFunct = ->
    "something else"
```

pre 1.2 this service would return the whole object as the service.

post 1.2 this service returns `someFunct` as the value of the service

you would need to change this services to

```
myApp.service 'applicationSrvc', ->
  @something = "value"
  @someFunct = ->
    "something else"
  return
```

to continue to return the complete instance.

See [c22adbf1](https://github.com/angular/angular.js/commit/c22adbf160f32c1839fbb35382b7a8c6bcec2927).
