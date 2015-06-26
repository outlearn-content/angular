<!--
{
"name" : "css-styling",
"version" : "0.1",
"title" : "Working With CSS",
"description" : "List of CSS classes set by Angular.",
"homepage" : "https://docs.angularjs.org/guide",
"freshnessDate" : 2015-06-02,
"license" : "CC BY 3.0"
}
-->


<!-- @section -->

# Overview

Angular sets these CSS classes. It is up to your application to provide useful styling.


<!-- @section -->

# CSS classes used by angular

* `ng-scope`
  - **Usage:** angular applies this class to any element for which a new scope
    is defined. (see [scope](https://pilot.outlearn.com/learn/ShieldSensei/angular/8) guide for more information about scopes)

* `ng-isolate-scope`
  - **Usage:** angular applies this class to any element for which a new
    isolate scope is defined.

* `ng-binding`
  - **Usage:** angular applies this class to any element that is attached to a data binding, via `ng-bind` or
    `{{}}` curly braces, for example. (see [databinding](https://pilot.outlearn.com/learn/ShieldSensei/angular/5) guide)

* `ng-invalid`, `ng-valid`
  - **Usage:** angular applies this class to a form control widget element if that element's input does
    not pass validation. (see [input](https://docs.angularjs.org/api/ng/directive/input) directive)

* `ng-pristine`, `ng-dirty`
  - **Usage:** angular ngModel directive applies `ng-pristine` class
    to a new form control widget which did not have user interaction. Once the user interacts with
    the form control, the class is changed to `ng-dirty`.

* `ng-touched`, `ng-untouched`
  - **Usage:** angular ngModel directive applies `ng-untouched` class
    to a new form control widget which has not been blurred. Once the user blurs the form control,
    the class is changed to `ng-touched`.



<!-- @section -->

## Related Topics

* [Angular Templates](https://pilot.outlearn.com/learn/ShieldSensei/angular/10)
* [Angular Forms](https://pilot.outlearn.com/learn/ShieldSensei/angular/13)
