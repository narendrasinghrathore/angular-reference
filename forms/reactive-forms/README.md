# Reactive Forms
Reactive forms use an explicit and immutable approach to managing the state of a form at a given point in time. Each change to the form state returns a new state, which maintains the integrity of the model between changes. Reactive forms are built around observable streams, where form inputs and values are provided as streams of input values, which can be accessed synchronously.

Reactive forms also provide a straightforward path to testing because you are assured that your data is consistent and predictable when requested. Any consumers of the streams have access to manipulate that data safely.

Reactive forms differ from template-driven forms in distinct ways. Reactive forms provide more predictability with synchronous access to the data model, immutability with observable operators, and change tracking through observable streams. If you prefer direct access to modify data in your template, template-driven forms are less explicit because they rely on directives embedded in the template, along with mutable data to track changes asynchronously. See the  [Forms Overview](https://angular.io/guide/forms-overview)  for detailed comparisons between the two paradigms.



The `[FormControl](https://angular.io/api/forms/FormControl)` class is the basic building block when using reactive forms. To register a single form control, import the `[FormControl](https://angular.io/api/forms/FormControl)` class into your component and create a new instance of the form control to save as a class property.

    import { Component } from '@angular/core';
    import { FormControl } from '@angular/forms';
    
    @Component({
      selector: 'app-name-editor',
      templateUrl: './name-editor.component.html',
      styleUrls: ['./name-editor.component.css']
    })
    export class NameEditorComponent {
      name = new FormControl('');
    }

Use the constructor of `[FormControl](https://angular.io/api/forms/FormControl)` to set its initial value, which in this case is an empty string. By creating these controls in your component class, you get immediate access to listen for, update, and validate the state of the form input.


### Registering the control in the template

After you create the control in the component class, you must associate it with a form control element in the template. Update the template with the form control using the  `formControl`  binding provided by  `[FormControlDirective](https://angular.io/api/forms/FormControlDirective)`  included in  `[ReactiveFormsModule](https://angular.io/api/forms/ReactiveFormsModule)`.

    <label>
      Name:
      <input type="text" [formControl]="name">
    </label>
Using the template binding syntax, the form control is now registered to the `name` input element in the template. The form control and DOM element communicate with each other: the view reflects changes in the model, and the model reflects changes in the view.


### Replacing a form control value

Reactive forms have methods to change a control's value programmatically, which gives you the flexibility to update the value without user interaction. A form control instance provides a  `setValue()`  method that updates the value of the form control and validates the structure of the value provided against the control's structure. For example, when retrieving form data from a backend API or service, use the  `setValue()`  method to update the control to its new value, replacing the old value entirely.

The following example adds a method to the component class to update the value of the control to  _Nancy_  using the  `setValue()`  method.

    updateName() {
      this.name.setValue('Nancy');
    }

**Note:** In this example, you're using a single control. When using the `setValue()` method with a form group or form array instance, the value needs to match the structure of the group or array.

## Grouping form controls
Just as a form control instance gives you control over a single input field, a form group instance tracks the form state of a group of form control instances (for example, a form). Each control in a form group instance is tracked by name when creating the form group. The following example shows how to manage multiple form control instances in a single group.

To initialize the form group, provide the constructor with an object of named keys mapped to their control.

    import { Component } from '@angular/core';
    import { FormGroup, FormControl } from '@angular/forms';
    
    @Component({
      selector: 'app-profile-editor',
      templateUrl: './profile-editor.component.html',
      styleUrls: ['./profile-editor.component.css']
    })
    export class ProfileEditorComponent {
      profileForm = new FormGroup({
        firstName: new FormControl(''),
        lastName: new FormControl(''),
      });
    }

The individual form controls are now collected within a group. A `[FormGroup](https://angular.io/api/forms/FormGroup)` instance provides its model value as an object reduced from the values of each control in the group. A form group instance has the same properties (such as `value` and `untouched`) and methods (such as `setValue()`) as a form control instance.

### Step 2: Associating the FormGroup model and view

A form group tracks the status and changes for each of its controls, so if one of the controls changes, the parent control also emits a new status or value change. The model for the group is maintained from its members. After you define the model, you must update the template to reflect the model in the view.

    <form [formGroup]="profileForm">
      
      <label>
        First Name:
        <input type="text" formControlName="firstName">
      </label>
    
      <label>
        Last Name:
        <input type="text" formControlName="lastName">
      </label>
    
    </form>

Note that just as a form group contains a group of controls, the _profile form_  `[FormGroup](https://angular.io/api/forms/FormGroup)` is bound to the `form` element with the `[FormGroup](https://angular.io/api/forms/FormGroup)` directive, creating a communication layer between the model and the form containing the inputs. The `[formControlName](https://angular.io/api/forms/FormControlName)` input provided by the `[FormControlName](https://angular.io/api/forms/FormControlName)` directive binds each individual input to the form control defined in `[FormGroup](https://angular.io/api/forms/FormGroup)`. The form controls communicate with their respective elements. They also communicate changes to the form group instance, which provides the source of truth for the model value.


### Saving form data

The  `ProfileEditor`  component accepts input from the user, but in a real scenario you want to capture the form value and make available for further processing outside the component. The  `[FormGroup](https://angular.io/api/forms/FormGroup)`  directive listens for the  `submit`  event emitted by the  `form`  element and emits an  `ngSubmit`  event that you can bind to a callback function.

Add an  `ngSubmit`  event listener to the  `form`  tag with the  `onSubmit()`  callback method.



    <form [formGroup]="profileForm" (ngSubmit)="onSubmit()">


The  `onSubmit()`  method in the  `ProfileEditor`  component captures the current value of  `profileForm`. Use  `[EventEmitter](https://angular.io/api/core/EventEmitter)`  to keep the form encapsulated and to provide the form value outside the component. The following example uses  `console.warn`  to log a message to the browser console.

    onSubmit() {
      // TODO: Use EventEmitter with form value
      console.warn(this.profileForm.value);
    }

The  `submit`  event is emitted by the  `form`  tag using the native DOM event. You trigger the event by clicking a button with  `submit`  type. This allows the user to press the  **Enter**  key to submit the completed form.

Use a  `button`  element to add a button to the bottom of the form to trigger the form submission.

     <button type="submit" [disabled]="!profileForm.valid">Submit</button>

**Note:**  The button in the snippet above also has a  `disabled`  binding attached to it to disable the button when  `profileForm`  is invalid. You aren't performing any validation yet, so the button is always enabled. Simple form validation is covered in the  [Simple form validation](https://angular.io/guide/reactive-forms#simple-form-validation)  section.


## Creating nested form groups

When building complex forms, managing the different areas of information is easier in smaller sections, and some groups of information naturally fall into the same group. Using a nested form group instance allows you to break large forms groups into smaller, more manageable ones.

### Step 1: Creating a nested group

An address is a good example of information that can be grouped together. Form groups can accept both form control and form group instances as children. This makes composing complex form models easier to maintain and logically group together.

    import { Component } from '@angular/core';
    import { FormGroup, FormControl } from '@angular/forms';
    
    @Component({
      selector: 'app-profile-editor',
      templateUrl: './profile-editor.component.html',
      styleUrls: ['./profile-editor.component.css']
    })
    export class ProfileEditorComponent {
      profileForm = new FormGroup({
        firstName: new FormControl(''),
        lastName: new FormControl(''),
        address: new FormGroup({
          street: new FormControl(''),
          city: new FormControl(''),
          state: new FormControl(''),
          zip: new FormControl('')
        })
      });
    }

### Step 2: Grouping the nested form in the template

After you update the model in the component class, update the template to connect the form group instance and its input elements.

    <div formGroupName="address">
      <h3>Address</h3>
    
      <label>
        Street:
        <input type="text" formControlName="street">
      </label>
    
      <label>
        City:
        <input type="text" formControlName="city">
      </label>
      
      <label>
        State:
        <input type="text" formControlName="state">
      </label>
    
      <label>
        Zip Code:
        <input type="text" formControlName="zip">
      </label>
    </div>

**Note:** Display the value for the form group instance in the component template using the `value` property and `[JsonPipe](https://angular.io/api/common/JsonPipe)`.

## Partial model updates

When updating the value for a form group instance that contains multiple controls, you may only want to update parts of the model. This section covers how to update specific parts of a form control data model.

### Patching the model value
There are two ways to update the model value:

-   Use the  `setValue()`  method to set a new value for an individual control. The  `setValue()`  method strictly adheres to the structure of the form group and replaces the entire value for the control.
    
-   Use the  `patchValue()`  method to replace any properties defined in the object that have changed in the form model.

The strict checks of the `setValue()` method help catch nesting errors in complex forms, while `patchValue()` fails silently on those errors.
This is necessary because the `patchValue()` method applies the update against the model structure. `PatchValue()` only updates properties that the form model defines.

## Generating form controls with FormBuilder

Creating form control instances manually can become repetitive when dealing with multiple forms. The  `[FormBuilder](https://angular.io/api/forms/FormBuilder)`  service provides convenient methods for generating controls.

The following section refactors the  `ProfileEditor`  component to use the form builder service to create form control and form group instances.

### Step 1: Importing the FormBuilder class
Import the  `[FormBuilder](https://angular.io/api/forms/FormBuilder)`  class from the  `@angular/forms`  package.

    import { FormBuilder } from '@angular/forms';

### Step 2: Injecting the FormBuilder service

The  `[FormBuilder](https://angular.io/api/forms/FormBuilder)`  service is an injectable provider that is provided with the reactive forms module. Inject this dependency by adding it to the component constructor.

    constructor(private fb: FormBuilder) { }

### Step 3: Generating form controls

The  `[FormBuilder](https://angular.io/api/forms/FormBuilder)`  service has three methods:  `[control()](https://angular.io/api/forms/FormBuilder#control)`,  `[group()](https://angular.io/api/forms/FormBuilder#group)`, and  `[array()](https://angular.io/api/forms/FormBuilder#array)`. These are factory methods for generating instances in your component classes including form controls, form groups, and form arrays.

Use the  `[group](https://angular.io/api/animations/group)`  method to create the  `profileForm`  controls.

    import { Component } from '@angular/core';
    import { FormBuilder } from '@angular/forms';
    
    @Component({
      selector: 'app-profile-editor',
      templateUrl: './profile-editor.component.html',
      styleUrls: ['./profile-editor.component.css']
    })
    export class ProfileEditorComponent {
      profileForm = this.fb.group({
        firstName: [''],
        lastName: [''],
        address: this.fb.group({
          street: [''],
          city: [''],
          state: [''],
          zip: ['']
        }),
      });
    
      constructor(private fb: FormBuilder) { }
    }

n the example above, you use the `[group()](https://angular.io/api/forms/FormBuilder#group)` method with the same object to define the properties in the model. The value for each control name is an array containing the initial value as the first item in the array.

**Note:** You can define the control with just the initial value, but if your controls need sync or async validation, add sync and async validators as the second and third items in the array.

HTML5 has a set of built-in attributes that you can use for native validation, including `required`, `[minlength](https://angular.io/api/forms/MinLengthValidator)`, and `[maxlength](https://angular.io/api/forms/MaxLengthValidator)`. You can take advantage of these optional attributes on your form input elements. Add the `required` attribute to the `firstName` input element.

**Caution:** Use these HTML5 validation attributes _in combination with_ the built-in validators provided by Angular's reactive forms. Using these in combination prevents errors when the expression is changed after the template has been checked.

### Displaying form status

When you add a required field to the form control, its initial status is invalid. This invalid status propagates to the parent form group element, making its status invalid. Access the current status of the form group instance through its  `status`  property.

    <p>
      Form Status: {{ profileForm.status }}
    </p>

## Dynamic controls using form arrays

`[FormArray](https://angular.io/api/forms/FormArray)` is an alternative to `[FormGroup](https://angular.io/api/forms/FormGroup)` for managing any number of unnamed controls. As with form group instances, you can dynamically insert and remove controls from form array instances, and the form array instance value and validation status is calculated from its child controls. However, you don't need to define a key for each control by name, so this is a great option if you don't know the number of child values in advance.

    import { FormArray } from '@angular/forms';

You can initialize a form array with any number of controls, from zero to many, by defining them in an array. Add an  `aliases`  property to the form group instance for  `profileForm`  to define the form array.

Use the  `[FormBuilder.array()](https://angular.io/api/forms/FormBuilder#array)`  method to define the array, and the  `[FormBuilder.control()](https://angular.io/api/forms/FormBuilder#control)`  method to populate the array with an initial control.

    profileForm = this.fb.group({
      firstName: ['', Validators.required],
      lastName: [''],
      address: this.fb.group({
        street: [''],
        city: [''],
        state: [''],
        zip: ['']
      }),
      aliases: this.fb.array([
        this.fb.control('')
      ])
    });
The aliases control in the form group instance is now populated with a single control until more controls are added dynamically.

A getter provides easy access to the aliases in the form array instance compared to repeating the  `profileForm.get()`  method to get each instance. The form array instance represents an undefined number of controls in an array. It's convenient to access a control through a getter, and this approach is easy to repeat for additional controls.

Use the getter syntax to create an  `aliases`  class property to retrieve the alias's form array control from the parent form group.

    get aliases() {
      return this.profileForm.get('aliases') as FormArray;
    }

**Note:** Because the returned control is of the type `[AbstractControl](https://angular.io/api/forms/AbstractControl)`, you need to provide an explicit type to access the method syntax for the form array instance.

Define a method to dynamically insert an alias control into the alias's form array. The `[FormArray.push()](https://angular.io/api/forms/FormArray#push)` method inserts the control as a new item in the array.

    addAlias() {
      this.aliases.push(this.fb.control(''));
    }

To attach the aliases from your form model, you must add it to the template. Similar to the `[formGroupName](https://angular.io/api/forms/FormGroupName)` input provided by `FormGroupNameDirective`, `[formArrayName](https://angular.io/api/forms/FormArrayName)` binds communication from the form array instance to the template with `FormArrayNameDirective`.

Add the template HTML below after the `<div>` closing the `[formGroupName](https://angular.io/api/forms/FormGroupName)` element.


    <div formArrayName="aliases">
      <h3>Aliases</h3> <button (click)="addAlias()">Add Alias</button>
    
      <div *ngFor="let address of aliases.controls; let i=index">
        <!-- The repeated alias template -->
        <label>
          Alias:
          <input type="text" [formControlName]="i">
        </label>
      </div>
    </div>


The `*[ngFor](https://angular.io/api/common/NgForOf)` directive iterates over each form control instance provided by the aliases form array instance. Because form array elements are unnamed, you assign the index to the `i` variable and pass it to each control to bind it to the `[formControlName](https://angular.io/api/forms/FormControlName)` input.

Each time a new alias instance is added, the new form array instance is provided its control based on the index. This allows you to track each individual control when calculating the status and value of the root control.


### Reactive forms API
Listed below are the base classes and services used to create and manage form controls.

Class

Description

[AbstractControl](https://angular.io/api/forms/AbstractControl)

The abstract base class for the concrete form control classes  [FormControl](https://angular.io/api/forms/FormControl),  [FormGroup](https://angular.io/api/forms/FormGroup), and  [FormArray](https://angular.io/api/forms/FormArray). It provides their common behaviors and properties.

[FormControl](https://angular.io/api/forms/FormControl)

Manages the value and validity status of an individual form control. It corresponds to an HTML form control such as  `<input>`  or  `<select>`.

[FormGroup](https://angular.io/api/forms/FormGroup)

Manages the value and validity state of a group of  [AbstractControl](https://angular.io/api/forms/AbstractControl)  instances. The group's properties include its child controls. The top-level form in your component is  [FormGroup](https://angular.io/api/forms/FormGroup).

[FormArray](https://angular.io/api/forms/FormArray)

Manages the value and validity state of a numerically indexed array of  [AbstractControl](https://angular.io/api/forms/AbstractControl)  instances.

[FormBuilder](https://angular.io/api/forms/FormBuilder)

An injectable service that provides factory methods for creating control instances.

#### Directives[](https://angular.io/guide/reactive-forms#directives "Link to this heading")

Directive

Description

[FormControlDirective](https://angular.io/api/forms/FormControlDirective)

Syncs a standalone  [FormControl](https://angular.io/api/forms/FormControl)  instance to a form control element.

[FormControlName](https://angular.io/api/forms/FormControlName)

Syncs  [FormControl](https://angular.io/api/forms/FormControl)  in an existing `[FormGroup](https://angular.io/api/forms/FormGroup)  instance to a form control element by name.

[FormGroupDirective](https://angular.io/api/forms/FormGroupDirective)

Syncs an existing  [FormGroup](https://angular.io/api/forms/FormGroup)  instance to a DOM element.

[FormGroupName](https://angular.io/api/forms/FormGroupName)

Syncs a nested  [FormGroup](https://angular.io/api/forms/FormGroup)  instance to a DOM element.

[FormArrayName](https://angular.io/api/forms/FormArrayName)

Syncs a nested [FormArray](https://angular.io/api/forms/FormArray)  instance to a DOM element.
