<!--
{
"name" : "databinding",
"version" : "0.1",
"title" : "Data Binding",
"description" : "How to synchronize data between model and view components.",
"homepage" : "https://docs.angularjs.org/guide",
"freshnessDate" : 2015-06-02,
"license" : "CC BY 3.0"
}
-->

# Data Binding

Data-binding in Angular apps is the automatic synchronization of data between the model and view
components.  The way that Angular implements data-binding lets you treat the model as the
single-source-of-truth in your application.  The view is a projection of the model at all times.
When the model changes, the view reflects the change, and vice versa.

## Data Binding in Classical Template Systems

<img class="right" src="img/One_Way_Data_Binding.png"/><br />
Most templating systems bind data in only one direction: they merge template and model components
together into a view. After the merge occurs, changes to the model
or related sections of the view are NOT automatically reflected in the view. Worse, any changes
that the user makes to the view are not reflected in the model. This means that the developer has
to write code that constantly syncs the view with the model and the model with the view.

## Data Binding in Angular Templates

<img class="right" src="img/Two_Way_Data_Binding.png"/><br />
Angular templates work differently. First the template (which is the uncompiled HTML along with
any additional markup or directives) is compiled on the browser. The compilation step produces a
live view. Any changes to the view are immediately reflected in the model, and any changes in
the model are propagated to the view. The model is the single-source-of-truth for the application
state, greatly simplifying the programming model for the developer. You can think of
the view as simply an instant projection of your model.

Because the view is just a projection of the model, the controller is completely separated from the
view and unaware of it. This makes testing a snap because it is easy to test your controller in
isolation without the view and the related DOM/browser dependency.


## Related Topics

* Angular Scopes
* Angular Templates