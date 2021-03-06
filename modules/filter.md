<!--
{
"name" : "filter",
"version" : "0.1",
"title" : "Filters",
"description" : "A filter formats the value of an expression for display to the user.",
"canonicalSource" : "https://docs.angularjs.org/guide/filter",
"homepage" : "https://docs.angularjs.org/guide",
"freshnessDate" : 2015-06-02,
"license" : "CC BY 3.0"
}
-->


<!-- @section -->

## Overview

A filter formats the value of an expression for display to the user. They can be used in view templates,
controllers or services and it is easy to define your own filter.

The underlying API is the `filterProvider`.


<!-- @section -->

## Using filters in view templates

Filters can be applied to expressions in view templates using the following syntax:

        {{ expression | filter }}

E.g. the markup `{{ 12 | currency }}` formats the number 12 as a currency using the `currency`
filter. The resulting value is `$12.00`.

Filters can be applied to the result of another filter. This is called "chaining" and uses
the following syntax:

        {{ expression | filter1 | filter2 | ... }}

Filters may have arguments. The syntax for this is

        {{ expression | filter:argument1:argument2:... }}

E.g. the markup `{{ 1234 | number:2 }}` formats the number 1234 with 2 decimal points using the
`number` filter. The resulting value is `1,234.00`.



<!-- @section -->

## Using filters in controllers, services, and directives

You can also use filters in controllers, services, and directives. For this, inject a dependency
with the name `<filterName>Filter` to your controller/service/directive. E.g. using the dependency
`numberFilter` will inject the number filter. The injected argument is a function that takes the
value to format as first argument and filter parameters starting with the second argument.

The example below uses the filter called `filter`.
This filter reduces arrays into sub arrays based on
conditions. The filter can be applied in the view template with markup like
`{{ctrl.array | filter:'a'}}`, which would do a fulltext search for "a".
However, using a filter in a view template will reevaluate the filter on
every digest, which can be costly if the array is big.

The example below therefore calls the filter directly in the controller.
By this, the controller is able to call the filter only when needed (e.g. when the data is loaded from the backend
or the filter expression is changed).

  
_Example file_: `index.html`

```javascript
<div ng-controller="FilterController as ctrl">
  <div>
    All entries:
    <span ng-repeat="entry in ctrl.array">{{entry.name}} </span>
  </div>
  <div>
    Entries that contain an "a":
    <span ng-repeat="entry in ctrl.filteredArray">{{entry.name}} </span>
  </div>
</div>
```


  
_Example file_: `script.js`

```javascript
angular.module('FilterInControllerModule', []).
  controller('FilterController', ['filterFilter', function(filterFilter) {
    this.array = [
      {name: 'Tobias'},
      {name: 'Jeff'},
      {name: 'Brian'},
      {name: 'Igor'},
      {name: 'James'},
      {name: 'Brad'}
    ];
    this.filteredArray = filterFilter(this.array, 'a');
  }]);
```




<!-- @section -->

## Creating custom filters

Writing your own filter is very easy: just register a new filter factory function with
your module. Internally, this uses the `filterProvider`.
This factory function should return a new filter function which takes the input value
as the first argument. Any filter arguments are passed in as additional arguments to the filter
function.

The filter function should be a [pure function](http://en.wikipedia.org/wiki/Pure_function), which
means that it should be stateless and idempotent. Angular relies on these properties and executes
the filter only when the inputs to the function change.

> **Note:** Filter names must be valid angular expression identifiers, such as `uppercase` or `orderBy`.
>Names with special characters, such as hyphens and dots, are not allowed.  If you wish to namespace
>your filters, then you can use capitalization (`myappSubsectionFilterx`) or underscores
>(`myapp_subsection_filterx`).

The following sample filter reverses a text string. In addition, it conditionally makes the
text upper-case.

  
_Example file_: `index.html`

```javascript
<div ng-controller="MyController">
  <input ng-model="greeting" type="text"><br>
  No filter: {{greeting}}<br>
  Reverse: {{greeting|reverse}}<br>
  Reverse + uppercase: {{greeting|reverse:true}}<br>
</div>
```


  
_Example file_: `script.js`

```javascript
angular.module('myReverseFilterApp', [])
  .filter('reverse', function() {
    return function(input, uppercase) {
      input = input || '';
      var out = "";
      for (var i = 0; i < input.length; i++) {
        out = input.charAt(i) + out;
      }
      // conditional based on optional argument
      if (uppercase) {
        out = out.toUpperCase();
      }
      return out;
    };
  })
  .controller('MyController', ['$scope', function($scope) {
    $scope.greeting = 'hello';
  }]);
```




<!-- @section -->

## Stateful filters

It is strongly discouraged to write filters that are stateful, because the execution of those can't
be optimized by Angular, which often leads to performance issues. Many stateful filters can be
converted into stateless filters just by exposing the hidden state as a model and turning it into an
argument for the filter.

If you however do need to write a stateful filter, you have to mark the filter as `$stateful`, which
means that it will be executed one or more times during the each `$digest` cycle.

  
_Example file_: `index.html`

```javascript
<div ng-controller="MyController">
  Input: <input ng-model="greeting" type="text"><br>
  Decoration: <input ng-model="decoration.symbol" type="text"><br>
  No filter: {{greeting}}<br>
  Decorated: {{greeting | decorate}}<br>
</div>
```


  
_Example file_: `script.js`

```javascript
angular.module('myStatefulFilterApp', [])
  .filter('decorate', ['decoration', function(decoration) {

    function decorateFilter(input) {
      return decoration.symbol + input + decoration.symbol;
    }
    decorateFilter.$stateful = true;

    return decorateFilter;
  }])
  .controller('MyController', ['$scope', 'decoration', function($scope, decoration) {
    $scope.greeting = 'hello';
    $scope.decoration = decoration;
  }])
  .value('decoration', {symbol: '*'});
```




<!-- @section -->

## Testing custom filters

See the [phonecat tutorial](http://docs.angularjs.org/tutorial/step_09#test) for an example.
