<!--
{
"name" : "bootstrap",
"version" : "0.1",
"title" : "Bootstrap",
"description" : "Explains the Angular initialization process.",
"canonicalSource" : "https://docs.angularjs.org/guide/bootstrap",
"homepage" : "https://docs.angularjs.org/guide",
"freshnessDate" : 2015-06-02,
"license" : "CC BY 3.0"
}
-->


<!-- @section -->

# Bootstrap

This page explains the Angular initialization process and how you can manually initialize Angular
if necessary.



<!-- @section -->

## Angular `<script>` Tag

This example shows the recommended path for integrating Angular with what we call automatic
initialization.



```html
<!doctype html>
<html xmlns:ng="http://angularjs.org" ng-app>
  <body>
    ...
    <script src="angular.js"></script>
  </body>
</html>
```

  1. Place the `script` tag at the bottom of the page. Placing script tags at the end of the page
    improves app load time because the HTML loading is not blocked by loading of the `angular.js`
    script. You can get the latest bits from http://code.angularjs.org. Please don't link
    your production code to this URL, as it will expose a security hole on your site. For
    experimental development linking to our site is fine.
    * Choose: `angular-[version].js` for a human-readable file, suitable for development and
      debugging.
    * Choose: `angular-[version].min.js` for a compressed and obfuscated file, suitable for use in
      production.
  2. Place `ng-app` to the root of your application, typically on the `<html>` tag if you want
    angular to auto-bootstrap your application.

        <html ng-app>

  3. If you choose to use the old style directive syntax `ng:` then include xml-namespace in `html`
    to make IE happy. (This is here for historical reasons, and we no longer recommend use of
    `ng:`.)

        <html xmlns:ng="http://angularjs.org">




<!-- @section -->

## Automatic Initialization

<img src="https://raw.githubusercontent.com/outlearn-content/angular/master/img/guide/concepts-startup.png">

Angular initializes automatically upon `DOMContentLoaded` event or when the `angular.js` script is
evaluated if at that time `document.readyState` is set to `'complete'`. At this point Angular looks
for the `ng-app` directive which designates your application root.
If the `ng-app` directive is found then Angular will:

  * load the module associated with the directive.
  * create the application injector
  * compile the DOM treating the `ng-app` directive as the root of the compilation. This allows you to tell it to treat only a
    portion of the DOM as an Angular application.



```html
<!doctype html>
<html ng-app="optionalModuleName">
  <body>
    I can add: {{ 1+2 }}.
    <script src="angular.js"></script>
  </body>
</html>
```

As a best practice, consider adding an `ng-strict-di` directive on the same element as
`ng-app`:



```html
<!doctype html>
<html ng-app="optionalModuleName" ng-strict-di>
  <body>
    I can add: {{ 1+2 }}.
    <script src="angular.js"></script>
  </body>
</html>
```

This will ensure that all services in your application are properly annotated.
See the [dependency injection strict mode](https://docs.angularjs.org/guide/di) docs
for more.



<!-- @section -->

## Manual Initialization

If you need to have more control over the initialization process, you can use a manual
bootstrapping method instead. Examples of when you'd need to do this include using script loaders
or the need to perform an operation before Angular compiles a page.

Here is an example of manually initializing Angular:


```html
<!doctype html>
<html>
<body>
  <div ng-controller="MyController">
    Hello {{greetMe}}!
  </div>
  <script src="http://code.angularjs.org/snapshot/angular.js"></script>

  <script>
    angular.module('myApp', [])
      .controller('MyController', ['$scope', function ($scope) {
        $scope.greetMe = 'World';
      }]);

    angular.element(document).ready(function() {
      angular.bootstrap(document, ['myApp']);
    });
  </script>
</body>
</html>
```

Note that we provided the name of our application module to be loaded into the injector as the second
parameter of the angular.bootstrap function. Notice that `angular.bootstrap` will not create modules
on the fly. You must create any custom modules before you pass them as a parameter.

You should call `angular.bootstrap()` *after* you've loaded or defined your modules.
You cannot add controllers, services, directives, etc after an application bootstraps.

> **Note:** You should not use the ng-app directive when manually bootstrapping your app.

This is the sequence that your code should follow:

  1. After the page and all of the code is loaded, find the root element of your AngularJS
  application, which is typically the root of the document.

  2. Call angular.bootstrap to compile the element into an
  executable, bi-directionally bound application.



<!-- @section -->

## Deferred Bootstrap

This feature enables tools like [Batarang](https://github.com/angular/angularjs-batarang) and test runners
to hook into angular's bootstrap process and sneak in more modules
into the DI registry which can replace or augment DI services for
the purpose of instrumentation or mocking out heavy dependencies.

If `window.name` contains prefix `NG_DEFER_BOOTSTRAP!` when
angular.bootstrap is called, the bootstrap process will be paused
until `angular.resumeBootstrap()` is called.

`angular.resumeBootstrap()` takes an optional array of modules that
should be added to the original list of modules that the app was
about to be bootstrapped with.
