
## Introduction to Template-driven forms

Developing forms requires design skills (which are out of scope for this page), as well as framework support for  _two-way data binding, change tracking, validation, and error handling_, which you'll learn about on this page.

This page shows you how to build a simple form from scratch. Along the way you'll learn how to:

-   Build an Angular form with a component and template.
-   Use  [ngModel](https://angular.io/api/forms/NgModel)  to create two-way data bindings for reading and writing input-control values.
-   Track state changes and the validity of form controls.
-   Provide visual feedback using special CSS classes that track the state of the controls.
-   Display validation errors to users and enable/disable form controls.
-   Share information across HTML elements using template reference variables.

## Create a form component

An Angular form has two parts: an HTML-based  _template_  and a component  _class_  to handle data and user interactions programmatically.

Because template-driven forms are in their own module, you need to add the [FormsModule](https://angular.io/api/forms/FormsModule) to the array of `imports` for the application module before you can use forms.

You add the [FormsModule](https://angular.io/api/forms/FormsModule) to the list of `imports` defined in the `@[NgModule](https://angular.io/api/core/NgModule)` decorator. This gives the application access to all of the template-driven forms features, including [ngModel](https://angular.io/api/forms/NgModel).


## Create an initial HTML form template

Update the template file with the following contents:

    <div class="container">
        <h1>Hero Form</h1>
        <form>
          <div class="form-group">
            <label for="name">Name</label>
            <input type="text" class="form-control" id="name" required>
          </div>
    
          <div class="form-group">
            <label for="alterEgo">Alter Ego</label>
            <input type="text" class="form-control" id="alterEgo">
          </div>
    
          <button type="submit" class="btn btn-success">Submit</button>
    
        </form>
    </div>

In template driven forms, if you've imported `[FormsModule](https://angular.io/api/forms/FormsModule)`, you don't have to do anything to the `<form>` tag in order to make use of `[FormsModule](https://angular.io/api/forms/FormsModule)`. Continue on to see how this works.


    <input type="text" class="form-control" id="name"
           required
           [(ngModel)]="model.name" name="name">
    TODO: remove this: {{model.name}}

Focus on the binding syntax:  [([ngModel](https://angular.io/api/forms/NgModel))]="...".

You need one more addition to display the data. Declare a template variable for the form. Update the  `<form>`  tag with  #heroForm="[ngForm](https://angular.io/api/forms/NgForm)"  as follows:

    <form #heroForm="ngForm">

The variable `heroForm` is now a reference to the `[NgForm](https://angular.io/api/forms/NgForm)` directive that governs the form as a whole.

### The  _NgForm_  directive

What  [NgForm](https://angular.io/api/forms/NgForm)  directive? You didn't add an  [NgForm](https://angular.io/api/forms/NgForm)  directive.

Angular did. Angular automatically creates and attaches an  [NgForm](https://angular.io/api/forms/NgForm)  directive to the  `<form>`  tag.

The  [NgForm](https://angular.io/api/forms/NgForm)  directive supplements the  `form`  element with additional features. It holds the controls you created for the elements with an  [ngModel](https://angular.io/api/forms/NgModel)  directive and  `name`  attribute, and monitors their properties, including their validity. It also has its own  `valid`  property which is true only  _if every contained control_  is valid.

Internally, Angular creates [FormControl](https://angular.io/api/forms/FormControl) instances and registers them with an [NgForm](https://angular.io/api/forms/NgForm) directive that Angular attached to the `<form>` tag. Each [FormControl](https://angular.io/api/forms/FormControl) is registered under the name you assigned to the `name` attribute.

-   Each input element has an  `id`  property that is used by the  `label`  element's  `for`  attribute to match the label to its input control.
-   Each input element has a  `name`  property that is required by Angular forms to register the control with the form.

## Track control state and validity with  _ngModel_

Using [ngModel](https://angular.io/api/forms/NgModel)  in a form gives you more than just two-way data binding. It also tells you if the user touched the control, if the value changed, or if the value became invalid.

The  _NgModel_  directive doesn't just track state; it updates the control with special Angular CSS classes that reflect the state. You can leverage those class names to change the appearance of the control.

The `ng-valid`/`ng-invalid` pair is the most interesting, because you want to send a strong visual signal when the values are invalid. You also want to mark required fields. To create such visual feedback, add definitions for the `ng-*` CSS classes.


    .ng-valid[required], .ng-valid.required  {
      border-left: 5px solid #42A948; /* green */
    }
    
    .ng-invalid:not(form)  {
      border-left: 5px solid #a94442; /* red */
    }

    <label for="name">Name</label>
    <input type="text" class="form-control" id="name"
           required
           [(ngModel)]="model.name" name="name"
           #name="ngModel">
    <div [hidden]="name.valid || name.pristine"
         class="alert alert-danger">
      Name is required
    </div>

You need a template reference variable to access the input box's Angular control from within the template. Here you created a variable called `name` and gave it the value "ngModel".

Why "ngModel"? A directive's [exportAs](https://angular.io/api/core/Directive) property tells Angular how to link the reference variable to the directive. You set `name` to `[ngModel](https://angular.io/api/forms/NgModel)` because the `[ngModel](https://angular.io/api/forms/NgModel)` directive's `exportAs` property happens to be "ngModel".

You control visibility of the name error message by binding properties of the `name` control to the message `<div>` element's `hidden` property.

    <div [hidden]="name.valid || name.pristine"
         class="alert alert-danger">

If you ignore the `pristine` state, you would hide the message only when the value is valid. If you arrive in this component with a new (blank) hero or an invalid hero, you'll see the error message immediately, before you've done anything.

Some developers want the message to display only when the user makes an invalid change. Hiding the message while the control is "pristine" achieves that goal. You'll see the significance of this choice when you add a new hero to the form.


## Submit the form with  _ngSubmit_

The user should be able to submit this form after filling it in. The  _Submit_  button at the bottom of the form does nothing on its own, but it will trigger a form submit because of its type (`type="submit"`).

A "form submit" is useless at the moment. To make it useful, bind the form's  `ngSubmit`  event property to the hero form component's  `onSubmit()`  method:

    <form (ngSubmit)="onSubmit()" #heroForm="ngForm">


You'd already defined a template reference variable, `#heroForm`, and initialized it with the value "ngForm". Now, use that variable to access the form with the Submit button.
You'll bind the form's overall validity via the `heroForm` variable to the button's `disabled` property using an event binding. Here's the code:


    <button type="submit" class="btn btn-success" [disabled]="!heroForm.form.valid">Submit</button>



