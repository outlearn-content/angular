<!--
{
"name" : "templates",
"version" : "0.1",
"title" : "Templates",
"description" : "Angular combines the template with information from the model and controller to render the dynamic view that a user sees in the browser.",
"homepage" : "https://docs.angularjs.org/guide",
"freshnessDate" : 2015-06-02,
"license" : "CC BY 3.0"
}
-->

In Angular, templates are written with HTML that contains Angular-specific elements and attributes.
Angular combines the template with information from the model and controller to render the dynamic
view that a user sees in the browser.

These are the types of Angular elements and attributes you can use:

* Directive — An attribute or element that
  augments an existing DOM element or represents a reusable DOM component.
* Markup} — The double curly brace notation `{{ }` to bind expressions
  to elements is built-in Angular markup.
* Filter — Formats data for display.
* Form controls — Validates user input.

The following code snippet shows a template with directives and
curly-brace expression bindings:

```html
<html ng-app>
 <!-- Body tag augmented with ngController directive  -->
 <body ng-controller="MyController">
   <input ng-model="foo" value="bar">
   <!-- Button tag with ng-click directive, and
          string expression 'buttonText'
          wrapped in "{{ }}" markup -->
   <button ng-click="changeFoo()">{{buttonText}}</button>
   <script src="angular.js">
 </body>
</html>
```

In a simple app, the template consists of HTML, CSS, and Angular directives contained
in just one HTML file (usually `index.html`).

In a more complex app, you can display multiple views within one main page using "partials" –
segments of template located in separate HTML files. You can use the
ngView directive to load partials based on configuration passed
to the $route} service. The {@link tutorial/ angular tutorial shows this
technique in steps seven and eight.


## Related Topics

* Filters
* Forms

## Related API

* API Reference
